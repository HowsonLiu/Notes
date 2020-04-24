---
title: lazyinstance
date: 2020-04-14 22:18:59
tags:
    - duilib
---
`LazyInstance`是谷歌`nbase`库里的模板类，主要作用是延迟创建实例
# 成员
类私有部分如下
```c++
template<typename Type>
class LazyInstance
{	
    // ...
private:
	enum
	{
		kNone = 0,
		kCreating,
		kCreated,
		kDeleting,
	};

	base::subtle::Atomic32 state_;
	base::subtle::AtomicWord instance_;

	DISALLOW_COPY_AND_ASSIGN(LazyInstance);
}
```
将定义都展开一下
```c++
typedef __w64 int32_t Atomic32;
typedef intptr_t AtomicWord;

#define DISALLOW_COPY_AND_ASSIGN(TypeName) \
  TypeName(const TypeName&);               \
  void operator=(const TypeName&)
```
实际上它保存了两个值：
- `state_` 
类型是在win32是int类型，保存此对象的状态，值可以为上面的4中枚举状态
- `instance_` 
在win32下是intptr_t类型，实际上存的是模板Type类型对象的地址

而`DISALLOW_COPY_AND_ASSIGN`说明它禁止拷贝
# 实现
共有部分代码如下
```c++
public:

	LazyInstance() : instance_(NULL)
	{
		base::subtle::NoBarrier_Store(&state_, kNone);
	}

	~LazyInstance()
	{
		// |instance_| should be deleted by |OnExit|
		//DCHECK(instance_ == 0);
	}

	Type& Get()
	{
		return *Pointer();
	}

	Type* Pointer()
	{
		using namespace base::subtle;

		if (Acquire_Load(&state_) != kCreated)
		{
			Atomic32 state =
				NoBarrier_CompareAndSwap(&state_, kNone, kCreating);
			if (state == kNone)
			{
				// we take the chance to create the instance
				instance_ = reinterpret_cast<AtomicWord>(new Type());
				AtExitManager::RegisterCallback(OnExit, this);
				Release_Store(&state_, kCreated);
			}
			else if (state != kCreated)
			{
				// wait, util another thread created the instance
				while (Acquire_Load(&state_) != kCreated)
					Thread::YieldThread();
			}
		}

		return reinterpret_cast<Type *>(instance_);
	}
```
可以看出，他具体延迟创建的代码在Pointer函数中实现。大概流程如下：
1. 通过`Acquire_Load`函数判断当前状态是否为已创建状态`kCreated`，如果是，说明已经有实例，直接返回`instance_`对象
    > `Acquire_Load`函数属于读屏障，不取cache的值，而是取主存的值，在多线程环境下保证了此成员变量为最新
2. 如果当前状态是`kNone`，则更新为`kCreating`，通过`NoBarrier_CompareAndSwap`函数实现
    ```c++
    // Atomically execute:
    //      result = *ptr;
    //      if (*ptr == old_value)
    //        *ptr = new_value;
    //      return result;
    //
    // I.e., replace "*ptr" with "new_value" if "*ptr" used to be "old_value".
    // Always return the old value of "*ptr"
    //
    // This routine implies no memory barriers.
    Atomic32 NoBarrier_CompareAndSwap(volatile Atomic32* ptr,
                                    Atomic32 old_value,
                                    Atomic32 new_value);
    ```
    这个注释写的很明白：CompareAndSwap(CAS)是原子性操作。说明多线程同时跑到这个语句时，此函数保证后入者会得到最新的已经更改过的值，因此只有一个线程能够将`kNone`状态改为`kCreating`状态，另一个函数会在这之后原子操作，得到state为`kCreating`
3. 如果state是`kNone`，则
    > 注意参与比较的是局部变量而非成员变量。经过上一步的顺序执行后，实际上到这里已经线程安全了
	1. 创建实例
	2. 退出注册（后面再说）
	3. 将成员状态`state_`更新为`kCreated`
		> Release_Store函数属于写屏障，让写到cache的数据写进主存，让其他线程可见
4. 如果state不属于`kCreated`（而且2步骤得到的局部变量不是`kNone`），说明有另一个线程重入并且在创建对象，等待创建好了即可
# 比较
```c++
public:
	bool operator ==(Type *object) const
	{
		switch (base::subtle::NoBarrier_Load(&state_))
		{
		case kNone:
			return object == NULL;
		case kCreating:
		case kCreated:
			return instance_ == object;
		default:
			return false;
		}
	}
```
不存在没构造好懒对象就比较的情况，因此结合状态比较即可
# 删除
直接delete会造成内存泄漏，需要用OnExit函数
```c++
private:
	static void OnExit(void *lazy_instance)
	{
		LazyInstance<Type>* me =
			reinterpret_cast<LazyInstance<Type>*>(lazy_instance);
		delete reinterpret_cast<Type*>(me->instance_);
		base::subtle::NoBarrier_Store(&me->instance_, 0);
	}
```
`LazyInstance`对象只能通过此函数进行删除，而此函数是私有的，所以`LazyInstance`对象只能在程序结束时统一删除，并不能手动删除（不过`LazyInstance`对象一般都是全局对象，很少会主动删除）
# 好处
- 全局变量延迟创建
- 相比于单例，`LazyInstance`对象可以有多份
# 关于内存屏障
学习这个之前我还不了解内存屏障，在翻源码时发现了一点小知识，先记录一下
```c++
inline Atomic32 Acquire_Load(volatile const Atomic32* ptr) {
  Atomic32 value = *ptr;
  return value;
}

inline void Release_Store(volatile Atomic32* ptr, Atomic32 value) {
  *ptr = value; // works w/o barrier for current Intel chips as of June 2005
  // See comments in Atomic64 version of Release_Store() below.
}
```
上面是x86架构下msvc的内存屏障
```c++
inline Atomic32 Acquire_Load(volatile const Atomic32* ptr) {
  Atomic32 value = *ptr; // An x86 load acts as a acquire barrier.
  // See comments in Atomic64 version of Release_Store(), below.
  ATOMICOPS_COMPILER_BARRIER();
  return value;
}

inline void Release_Store(volatile Atomic32* ptr, Atomic32 value) {
  ATOMICOPS_COMPILER_BARRIER();
  *ptr = value; // An x86 store acts as a release barrier.
  // See comments in Atomic64 version of Release_Store(), below.
}

#define ATOMICOPS_COMPILER_BARRIER() __asm__ __volatile__("" : : : "memory")
```
这是x86下gcc的内存屏障

`volatile`关键字在msvc跟gcc下都能保证编译期指令不被打乱，但是gcc的运行时内存屏障需要有一段汇编代码`ATOMICOPS_COMPILER_BARRIER()`实现，而msvc通过`volatile`关键字就可以实现了

# 另外版本的实现
https://blog.csdn.net/leehark/article/details/6897858
