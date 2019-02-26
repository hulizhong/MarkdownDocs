[TOC]

## ReadMe
std::function 配合 std::bind 实现了 `可调用对象`的包装器。

- 实质是类模板：一种通用、多态的函数封装
- 可对c++中现有的可调用实体的安全包裹；（如函数指针这种是类型不安全的）
	- 函数指针
		- 普通函数
		- 类非静态成员函数，需要借助std::bind来构造对象
		- 类静态成员函数；
	- lambda表达式
	- 仿函数
- 功能
	- 延迟调用；
	- 特别适合作为回调使用；
- 注意点
	- fcuntion对象不能对比相等与否；（只能用于判空NULL/nullptr）
	- 头文件 functional
	- 编译 -std=c++11


## 函数指针-普通函数
请看如下代码
```cpp
#include <functional>  
//封装普通函数  
int TestFunc(int a)
{ 
	return a;
}  

int main(int argc, char* argv[])  
{  
	typedef std::function<int(int)> Functional;

	Functional obj = TestFunc;  
	int res = obj(0);  
	cout << "normal function : " << res << endl;  
}
```

## 函数指针-类成员函数
非静态成员函数，需要绑定调用对象。注意以下内容
> 借助std::bind将对象与参数绑在一块；
> 成员函数必须加取地址符；

静态成员函数，不需要绑定调用对象；
```cpp
class CTest
{   
public:
	int Func(int a)
    {
        return a;
    }
    static int SFunc(int a)  
    {  
        return a;  
    }  
};

CTest t;
Functional obj = std::bind(&CTest::Func, &t, std::placeholders::_1);  
	//std::placeholders::_1 即第1个参数变量的占位符号。（与传值、引用没关系）
int res = obj(3);  
cout << "member function : " << res << endl;  

obj = CTest::SFunc;  
res = obj(4);                                                                                                                           
cout << "static member function : " << res << endl;  
```

## lambda表达式
请看如下代码
```cpp
auto lambda = [](int a)->int{return a;};

typedef std::function<int(int)> Functional;
Functional obj = lambda;
int res = obj(2);  
```



## 仿函数
请看如下代码
```cpp
class Functor  
{  
public:  
    int operator() (int a)  
    {  
        return a;  
    }  
};  

typedef std::function<int(int)> Functional;
Functor functorObj;
Functional obj = functorObj;
int res = obj(2);  
```


## std::bind
它是一个函数模板，如同一个函数适配器；
把一个原本接收N个参数的函数fn，通过绑定一些参数，返回一个接收M个参数的函数ret；
> 绑定的参数将以`值传递`的方式传递给具体的函数。
> 占位符号将会以`引用`的方式传递。

函数ret可以直接赋值给std::function；


- 它可以绑定
	- 普通函数
	- lambda表达式
	- 类成员函数，成员变量
	- 模板函数


## std::function::target 
返回指向存储的可调用函数目标的指针。

声明如下：
```cpp
//注意：如果指定的类型T与最被std::function绑定的类型不一致，那么会返回空指针。
template< class T > 
T* target() noexcept;

template< class T > 
const T* target() const noexcept;
```

Demo
```cpp
#include <functional>
#include <iostream>
 
int f(int, int) { return 1; }
int g(int, int) { return 2; }
void test(std::function<int(int, int)> const& arg)
{
    std::cout << "test function: ";
    if (arg.target<std::plus<int>>())
        std::cout << "it is plus\n";
    if (arg.target<std::minus<int>>())
        std::cout << "it is minus\n";
 
    int (*const* ptr)(int, int) = arg.target<int(*)(int, int)>();
    if (ptr && *ptr == f)
        std::cout << "it is the function f\n";
    if (ptr && *ptr == g)
        std::cout << "it is the function g\n";
}
 
int main()
{
    test(std::function<int(int, int)>(std::plus<int>()));
    test(std::function<int(int, int)>(std::minus<int>()));
    test(std::function<int(int, int)>(f));
    test(std::function<int(int, int)>(g));
}
```
