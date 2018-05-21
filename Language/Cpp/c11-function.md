[toc]

## ReadMe
std::function
- 类模板：一种通用、多态的函数封装
- 可对c++中现有的可调用实体的安全包裹；（如函数指针这种是类型不安全的）
	- 普通函数
	- lambda表达式
	- 函数指针
	- 其它函数对象


## 普通函数

```cpp
#include <functional>  
//声明一个模板  
typedef std::function<int(int)> Functional;

//normal function  
int TestFunc(int a)
{ 
	return a;
}  

int main(int argc, char* argv[])  
{  
	//封装普通函数  
	Functional obj = TestFunc;  
	int res = obj(0);  
	cout << "normal function : " << res << endl;  
}
```

## lambda表达式



## 函数指针

## 其它函数对象

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
