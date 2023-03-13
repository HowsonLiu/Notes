# C/C++
---
## C与C++的区别
- C面向过程， C++面向对象
- struct在C中是结构体，只能存放变量。在C++中是类，可以有成员函数
- C++中有引用
- C的作用域只有局部与全局，C++有局部，类作用域以及命名空间
## 基本数据类型大小
|| 32位编译器 | 64位编译器 |
|:--:|:--:|:--:|
| **void\*** | **4字节** | **8字节** |
| char | 1字节 | 1字节 |
| short | 2字节 | 2字节 |
| int | 4字节 | 4字节 |
| uint | 4字节 | 4字节 |
| float | 4字节 | 4字节 |
| double | 8字节 | 8字节 |
| long | 4字节 | 4字节 |
| ulong | 4字节 | 4字节 |
| long long | 8字节 | 8字节 |
- 指针大小 = 编译器位数/8
- 关于long变量在64位下的大小，某些博客上说是64bits，但在MSVC2017上试了一下结果是32bits。这点可能根据不同编译器大小不一致，需要注意
## 内存对齐
### 一、定义
为了让内存存取更有效率而采用的一种编译阶段优化内存存取的手段
### 二、使用内存对齐的原因
- 不是所有的硬件平台都支持访问任意地址上的数据
- CPU读取未对齐的数据可能需要访问多次内存
### 三、编译器命令
```c++
#pragma pack(n) //指令在预处理阶段执行
```
手动设置内存对齐系数，n只能为1，2，4，8，16，默认为8
实际数据成员x对齐的大小：
> align(x) = min(sizeof(x), n)
### 四、对齐规则
1. 第一个成员变量放到offset为0的地方
2. 成员变量是基本数据类型x，则存储的起始位置为align(x)的整数倍
3. 成员变量是结构体y（其内部最大的基本数据类型成员变量m），则存储的起始位置为align(m)的整数倍
4. 成员变量是数组（类型为x），则存储的起始位置为align(x)的整数倍
5. 收尾，总大小为内部最大基本类型成员x的align(x)的整数倍
### 五、试试看
```c++
class A {
	char c;
	double d;
	char h;
	int i[4];
};
//sizeof(A) =  1 + (7) + 8 + 1 + (3) + 4*4 + (4) = 40
//#pragma pack(2) sizeof(A) = 1 + (1) + 8 + 1 + (1) + 4*4 = 28

class B{
	char c;
	int i;
	A a;
	char h;
};
// sizeof(B) = 1 + (3) + 4 + 40 + 1 + (7) = 56
// #pragma pack(2) sizeof(B) = 1 + (1) + 4 + 28 + 1 + (1) = 36
```
## C语言位域
将一个字节划分成不同的区域，并说明区域位数，从而储存不需要占一个字节的信息
```c++
struct bitsfield{
	char c : 2;	
	int i : 2;	
	int j : 2;	
	int  : 0;	
	int m : 2;	
}
```
### 要求
- 位域成员类型必须为整数类型，如int, char, long, short等
```c++
char c : 2; // ok
float f : 2; // error
A a : 2; // error
```
- 位域长度不能大于类型本身长度
```c++
int i : 33; // error int 32 bits
```
- 无名位域用于填充，不可访问
- 赋值不能超出位域大小，否则按位截取取值
```c++
bitsfield b;
b.i = 2;
cout << b.i << endl;	// 打印出-2，因为0x00000010截取为0x10，二进制补码为-2
```
### 大小
- 相邻位域类型相同，捆绑存储
```c++
struct bitsfield{
	char c : 2;	// 一个字节
	int i : 2;	
	int j : 2;	// i与j捆绑储存，一共占一个字节中的4个bits
	int  : 0;	// 空域分割，剩下的4个bits不能放m
	int m : 2;	// m放到下一个字节的前2个bits
}
// sizeof(bitsfield) = sizeof(char) + sizeof(int)
```
- 一个字节剩余空间不够存放下一个位域时，从下一字节开始
```c++
struct bitsfieldex{
	int c : 5;
	int d : 2;	// c d 捆绑为1个字节
	int e : 2;	// e储存在下一字节
}
// sizeof(bitsfieldex) = sizeof(int)
```
## C++关键字
### const
表明被修饰的变量或者对象值不能改变，改变会在编译时报错
- 修饰变量，则变量必须初始化
- 修饰对象，则对象不可变，且只能调用const的成员函数
- 修饰指针，根据*在const的位置：左定值，右定向
	```c++
	const int* p = &a; // (*p)不可变
	int* const p = &a; // p不可变
	const int* const p = &a; //p和(*p)都不可变
	```
- 修饰函数参数，一般仅用于指针和引用，值传递无意义
- 修饰函数返回值，在调用者看来使用返回值与使用一个const变量差不多
- 修饰类成员变量，则成员变量必须使用列表初始化
- 修饰类成员函数，则
	1. 该函数不能修改实例的值，即不能修改this指针
	2. 可以修改声明为mutable的成员变量
	3. 不能与static同时使用，因为static的成员函数没有this指针
	4. 有无const属于重载，默认调用无const版本的成员函数
### static
被修饰的变量或对象储存在静态存储区，生存周期为程序的整个运行期，未初始化时默认初始化为0
- 修饰全局变量，则静态全局变量只能在当前文件使用（全局变量可通过extern在其他文件使用）
- 修饰函数，则函数只能在当前文件使用
- 修饰局部变量，则
	- 局部作用域，只在当前函数可见
	- 函数结束不销毁，下次访问值不变
- 修饰成员变量，则
	- 被所有对象共享
	- 没有实例也能访问
	- 不能在类声明中定义，要在全局定义
	```c++
	class A{
		static int i = 1; //error
	}
	int A::i = 1; // ok
	```
	- 遵循类作用域访问规则（public、protected、private）
- 修饰成员函数，则
	- 函数内没有this指针
	- 不能访问非静态成员函数与成员变量
### extern
此变量（函数）已经在别处定义，要在此处使用
- 常用于文件间的全局变量共享
- extern "C" 常用于C++代码中使用C函数。它告诉编译器，不要改动此函数的名称，按C语言进行编译。由于C++函数多态，编译器往往会修改函数符号名，这就导致链接时编译器找不到C语言编译的函数
### volatile
表示变量可能被某些编译器未知的因素改变，比如操作系统，硬件，或者其他线程。在读取此值的时候应该从内存读取，而不是读取寄存器的备份。它有三个特点：
- 易变性
- 不可优化
```c++
int a = 1;
printf("%d", a);	// push 1 编译器优化之后会直接用常量替换 
volatile int b = 1;	// mov eax, dword ptr [esp]
printf("%d", b);	// push eax
```
- 顺序性   
指的是两个volatile变量的语句一定是顺序执行
### override（C++11）
表明此函数重写了基类的虚函数，但是只起提示作用。若编译器找不到对应虚函数则会在编译时报错，但实际上不加上此关键字也能实现重写。
### final（C++11）
- 禁止虚函数被重写
- 禁止基类被继承
### inline
- 将调用inline函数的地方将函数体展开，这样可以减少重复调用时栈内存的重复开辟
- 只适合简单函数使用，不能用于while，switch，或是递归自己的函数
- inline只是对编译器的建议
- 定义在类中的成员函数默认内联
- inline必须与定义使用，不能与声明使用
## KMP算法
### 例子
在“ABCABCDEFGH”（source）中查找“ABCABB”（target），一般暴力的方法是通过i指针遍历source，j指针遍历target，不一致时同时回退i、j指针到起始位置的后一个位置解决。但是KMP算法是通过之前遍历时相同的信息，不回退i指针，适量回退j指针实现，以达到更好的效率
### 核心
- 在遍历中我们不难推出，任何时候source[0, i-1] = target[0, j-1]
- 我们需要尝试找出一个k，使得target[0, k-1] = target[j-k, j-1]
- 这样在source[i] != target[j]时，j可以通过回退到k避免再次比较重复部分
- 我们需要写出一个next数组，记录k在j不同取值时的值
- 显然这个next只跟target有关
### 代码
```c++
void getNext(const char* target, int size, int* next){
	if(!target || size < 0 || !next) return;
	next[0] = -1;
	int j = 0;
	int k = -1;
	while(j < size - 1){
		if(k == -1 || target[j] == target[k])
			next[++j] = ++k;
		else
			k = next[k];
	}
}
```
接下来，我们在普通的比较失败时
```c++
if(source[i] != target[j]) j = next[j]; // 不需要回溯i指针了
```
## 坑
- VS中同一解决方案中不同项目互相调用，项目类型不能为{exe, exe}（即使设置一个为启动工程也不行）。
- 在{exe, lib}的解决方案中，如果lib项目调用了其他lib，则其他lib的路径名字设置只能在exe项目中设置。

## 关于链接
- .def文件仅在导出时使用，导入不需要使用
- dll的缺失并不会导致链接的错误
### 链接失败的原因
- 仔细检查lib路径以及lib库名字
- 有可能是缺少`extern "C"`关键字造成的，比如ffmpeg

## 变量声明与命名空间
- A命名空间里面的类a需要使用B命名空间里面的类b，在a类的头文件中：
```c++
namespace B{
	class b;	// right
}

class B::b;	// error
namespace A{
	class B::b;	// error
	class a{
	private:
		b instance;
	}
}
```

## dump调试
dump文件包含的是内存的镜像信息。分为：
- 内核态dump，用于分析内核相关的问题
- 用户态dump，用于分析用户程序的问题
	mini dump，尺寸小，只记录常用信息
	full dump，记录用户态所有的信息

细节
- IDE会把异常截取，所以在IDE中不能正常跑到生成dmp的逻辑中
- dmp文件需要与exe及对应pdb文件放在同一个目录
- 打开dmp的方法：
	- 双击打开，若dmp中崩溃处代码路径找不到，则提示你打开对应文件。这种方法只能看到崩溃栈中的代码。
	- 拖进已经打开的工程内，工程可以与编译成exe的工程稍微不一致。这种方法由于加载了整个工程，因此可以常规地任意跳转

## [动态链接与静态链接](https://blog.csdn.net/alisa_xf/article/details/79496113)
- 对于第三方库来说，动态链接一般指的是`LoadLibary`这种，静态链接指的是`#pragma comment`这种
- 对于C++运行时库来说，VS设置中一般有两种选项：Multi-threaded(MT)与Muti-threaded DLL(MD)。MT指的是将C++运行时库以静态链接的方式链接在一起,它链接的库是LIBCMT.lib，最终生成的文件中只有一个exe。而MD指的是将C++运行时库以动态链接的方式连接在一起，它需要的库是MSVCRT.dll，因此若运行的机器缺少此dll就会出错

## union
union 只会默认初始化第一个成员，因此成员的顺序可能会造成union默认初始化不成功