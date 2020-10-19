# Curiously Recurring Template Pattern

## CRTP简介

奇异递归模式：派生类继承自以自身为模板参数的模板基类

目的：实现**编译期多态**

例子：
```c++
template<typename Derive>
class Base{
public:
	void fun() {
		static_cast<Derive*>(this)->fun_impl();
	}
	static void static_fun() {
		Derive::static_fun_impl();	
	}
protected:
	void fun_impl();
	static void static_fun_impl();
};

class Derive_1:public Base<Derive_1> {
public:
	void fun_impl() {
		std::cout<<"Derive_1 fun_impl"<<std::endl;
	};
};

class Derive_2:public Base<Derive_2> {
public:
	static void static_fun_impl() {
		std::cout<<"static_fun_impl"<<std::endl;
	}
};
int main()
{
	Derive_1* d1 = new Derive_1();
	d1->fun();
	Derive_2::static_fun();
	return 0;
}
```

基类通过static_cast将this指针转成派生类指针，调用派生类的方法。和运行时多态不同，运行时多态通过虚函数表查询对应函数，会增加额外的查询开销。CRTP通过在编译期确定要调用的函数，增加编译时间的同时，减少运行时开销。

## enable_shared_from_this与CRTP

C++11中新增了smart pointer.类想返回智能指针版的this时，需要该类继承enable_shared_from_this,通过shared_from_this()返回对应智能指针。如下:

```c++
class Foo:public enable_shared_from_this<Foo> {
public:
	Foo(){}
	shared_ptr<Foo> GetSelf() {
		return shared_from_this();
	}
};
```

注：`return shared_ptr<T>(this)`这种将this强转成智能指针的写法是错误的。this实际上是裸指针，而每个shared_ptr有各自的控制块。对同一个资源用多个控制块管理会产生内存问题。每个控制块的引用计数不同，当一个控制块认为该释放资源时，其余控制块会认为该资源仍然存在而访问。

enable_shared_from_this作为基类，模板参数为派生类Foo，正符合CRTP的特点。以下是gcc实现

gcc5.4.0 shared_ptr.h
```c++
  /**
   *  @brief Base class allowing use of member function shared_from_this.
   */
  template<typename _Tp>
    class enable_shared_from_this
    {
    protected:
      constexpr enable_shared_from_this() noexcept { }

      enable_shared_from_this(const enable_shared_from_this&) noexcept { }

      enable_shared_from_this&
      operator=(const enable_shared_from_this&) noexcept
      { return *this; }

      ~enable_shared_from_this() { }

    public:
      shared_ptr<_Tp>
      shared_from_this()
      { return shared_ptr<_Tp>(this->_M_weak_this); }

      shared_ptr<const _Tp>
      shared_from_this() const
      { return shared_ptr<const _Tp>(this->_M_weak_this); }

    private:
      template<typename _Tp1>
	void
	_M_weak_assign(_Tp1* __p, const __shared_count<>& __n) const noexcept
	{ _M_weak_this._M_assign(__p, __n); }

      template<typename _Tp1, typename _Tp2>
	friend void
	__enable_shared_from_this_helper(const __shared_count<>&,
					 const enable_shared_from_this<_Tp1>*,
					 const _Tp2*) noexcept;

      mutable weak_ptr<_Tp>  _M_weak_this;
    };
```

从shared_from_this可以看出，返回的是派生类智能指针对应的weak_ptr。