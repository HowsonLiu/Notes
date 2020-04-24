WeakCallback是网易nbase库的一个重要部分，主要用来检测对象的生命周期。下面结合源码分析他是如何检测生命周期以及其缺陷。
# SupportWeakCallback
```c++
class BASE_EXPORT SupportWeakCallback
{
public:
	typedef std::weak_ptr<WeakFlag> _TyWeakFlag;
public:
	virtual ~SupportWeakCallback(){};

	template<typename CallbackType>
	auto ToWeakCallback(const CallbackType& closure)
		->WeakCallback<CallbackType>
	{
		return WeakCallback<CallbackType>(GetWeakFlag(), closure);
	}

	std::weak_ptr<WeakFlag> GetWeakFlag()
	{
		if (m_weakFlag.use_count() == 0) {
			m_weakFlag.reset((WeakFlag*)NULL);
		}
		return m_weakFlag;
	}

private:
	template<typename ReturnValue, typename... Param, typename WeakFlag>
	static std::function<ReturnValue(Param...)> ConvertToWeakCallback(
		const std::function<ReturnValue(Param...)>& callback, std::weak_ptr<WeakFlag> expiredFlag)
	{
		auto weakCallback = [expiredFlag, callback](Param... p) {
			if (!expiredFlag.expired()) {
				return callback(p...);
			}
			return ReturnValue();
		};

		return weakCallback;
	}

protected:
	std::shared_ptr<WeakFlag> m_weakFlag;
};

class WeakFlag
{

};
```
`SupportWeakCallback`类的主要作用是提供被检测生命周期的功能，如果一个类需要被监控（例如UI窗口），则继承这个类即可。检测方法主要通过智能指针`shared_ptr`实现。成员变量中有个`WeakFlag`类型的shared指针，成员方法将其对应的weak指针暴露出来。这样子，持有weak指针的一方就可以在任何时候通过`weak_ptr::expired`方法判断对应shared指针是否存在，从而检测到对象的生命周期。`WeakFlag`是一个空类，占位使用。
# WeakCallback
``` c++
template<typename T>
class WeakCallback
{
public:
	WeakCallback(const std::weak_ptr<WeakFlag>& weak_flag, const T& t) :
		weak_flag_(weak_flag),
		t_(t)
	{

	}

	WeakCallback(const std::weak_ptr<WeakFlag>& weak_flag, T&& t) :
		weak_flag_(weak_flag),
		t_(std::move(t))
	{

	}

	template<class WeakType>
	WeakCallback(const WeakType& weak_callback) :
		weak_flag_(weak_callback.weak_flag_),
		t_(weak_callback.t_)
	{

	}

	template<class... Args>
	auto operator ()(Args && ... args) const
		->decltype(t_(std::forward<Args>(args)...))
	{
		if (!weak_flag_.expired()) {
			return t_(std::forward<Args>(args)...);
		}
		return decltype(t_(std::forward<Args>(args)...))();
	}

	bool Expired() const
	{
		return weak_flag_.expired();
	}


	std::weak_ptr<WeakFlag> weak_flag_;
	mutable T t_;
};
```
`WeakCallback`可以将一个对象跟一个weak指针绑定在一起，在实际应用中主要配合可执行对象使用。它实现了`operator ()`函数，当weak指针所绑定的对象存在时，才执行函数。
# Bind
```c++
// global function 
template<class F, class... Args, class = typename std::enable_if<!std::is_member_function_pointer<F>::value>::type>
auto Bind(F && f, Args && ... args)
	->decltype(std::bind(f, args...))
{
	return std::bind(f, args...);
}

// const class member function 
template<class R, class C, class... DArgs, class P, class... Args>
auto Bind(R(C::*f)(DArgs...) const, P && p, Args && ... args)
	->WeakCallback<decltype(std::bind(f, p, args...))>
{
	std::weak_ptr<WeakFlag> weak_flag = ((SupportWeakCallback*)p)->GetWeakFlag();
	auto bind_obj = std::bind(f, p, args...);
	static_assert(std::is_base_of<nbase::SupportWeakCallback, C>::value, "nbase::SupportWeakCallback should be base of C");
	WeakCallback<decltype(bind_obj)> weak_callback(weak_flag, std::move(bind_obj));
	return weak_callback;
}

// non-const class member function 
template<class R, class C, class... DArgs, class P, class... Args>
auto Bind(R(C::*f)(DArgs...), P && p, Args && ... args) 
	->WeakCallback<decltype(std::bind(f, p, args...))>
{
	std::weak_ptr<WeakFlag> weak_flag = ((SupportWeakCallback*)p)->GetWeakFlag();
	auto bind_obj = std::bind(f, p, args...);
	static_assert(std::is_base_of<nbase::SupportWeakCallback, C>::value, "nbase::SupportWeakCallback should be base of C");
	WeakCallback<decltype(bind_obj)> weak_callback(weak_flag, std::move(bind_obj));
	return weak_callback;
}
```
继承了`SupportWeakCallback`的类，应该使用`nbase::Bind`而非`std::Bind`。因为`nbase::Bind`能生成`WeakCallback`的可执行对象，比`std::Bind`多了一个生命周期的检测，更加安全。对于全局函数`nbase::Bind`退化成`std::Bind`
# WeakCallbackFlag
```c++
class BASE_EXPORT WeakCallbackFlag final : public SupportWeakCallback
{
public:
	void Cancel()
	{
		m_weakFlag.reset();
	}

	bool HasUsed()
	{
		return m_weakFlag.use_count() != 0;
	}
};

```
`SupportWeakCallback`一般用于类的继承，而`WeakCallbackFlag`一般用在成员变量。`SupportWeakCallback`只有在析构的时候才会销毁对应的shared指针，但`WeakCallbackFlag`可以通过主动调用Cancel销毁，比`SupportWeakCallback`更轻便。在实际应用中主要配合`nbase::ThreadManager::PostRepeatedTask`当定时器使用
# 缺点
`WeakCallback`的优点在于，它提供一种比较优雅的方式去检测对象的生命周期；但是缺点在于，它并不是线程安全的。比较典型的例子是这样的
```
// 继承了SupportWeakCallback类的成员函数
void classSupportWeakCallbackFunc(){
    auto http_cb = ToWeakCallback([=](bool ret, int response_code, const std::string &reply){
        QLOG_APP(L"baidu result: Ret:{0}, Code:{1}, Json:{2}") << ret << response_code << reply;
	StdClosure cb = ToWeakCallback([=](){
		// update ui
	};
	Post2UI(cb);
    }));
    HttpHandler::Get(http_cb, "www.baidu.com");
}
```
当初我在项目里很容易就把`ToWeakCallback`顺手写进去了，但是不对的。问题就在第二个`ToWeakCallback`里。它是在Http线程里被调用，并且没有锁操作。因此有可能在执行第一个`ToWeakCallback`的时候this是存在的，但是在生成第二个`ToWeakCallback`的时候已经销毁。正确的写法是这样的
```
// 继承了SupportWeakCallback类的成员函数
void classSupportWeakCallbackFunc(){
    auto weak_flag = this->GetWeakFlag();
    auto http_cb = ToWeakCallback([this, weak_flag](bool ret, int response_code, const std::string &reply){
        QLOG_APP(L"baidu result: Ret:{0}, Code:{1}, Json:{2}") << ret << response_code << reply;
	StdClosure cb = [=](){
            if(weak_flag.expried()) return;
	    // update ui
	};
	Post2UI(cb);
    });
    HttpHandler::Get(http_cb, "www.baidu.com");
}
```
在http线程不操作this，而是传递weak指针，留到UI线程再去判断。