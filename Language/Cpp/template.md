[TOC]



## ReadMe

模板相关；

https://blog.csdn.net/shineHoo/article/details/5723618?utm_source=blogxgwz9

http://www.cnblogs.com/racaljk/p/7822290.html

https://www.cnblogs.com/cthon/p/9203234.html

https://www.cnblogs.com/cthon/p/9203234.html

## 类模板

template class.

```cpp
template <typename T>
class Foo
{
    T mV;
public:
    Foo(T t) : mV(t) {}
};

Foo<int> f(5);
```



## 函数模板

template function.

```cpp
tempalte <typename T>
T max(T a, T b)
{
    return a > b ? a : b;
}
template double max<double, double>;  //显示实例化。--编译期间就产生一个函数定义。（运行期效率更高）
	//template [函数返回类型] [函数模板名]<实际类型列表>（函数参数列表）

max<int>(4, 8);  //运行期间会发生隐式实例化（产生一个(int, int)的函数定义），再调用它。
max<float>(4.0, 8.0);  //同上
```



## 实例化

在程序中的函数模板本身并不会生成函数定义，它只是一个用于生成函数定义的方案。编译器使用模板为特定类型生成函数定义时，得到的是模板实例。这即是函数模板的实例化。


### 显式实例化

explicit instantiation

### 隐式实例化

implicit instantiation



## 特化

specialization，特化（具体化），分全特化、偏特化。
需要对模板中一些特定的类进行特殊的处理。（如max函数需要特殊处理字符串）

```cpp
template <typename T>
void swap<job>(T a, T b)
{
    //...
} 

class job;
template <>
void swap<job>(job a, job b)
{
    //... 注意同显示实例化语法的区别；（template要加<>;  函数需要定义{}）
}  
```



优先级：非模板函数 》特化模板》常规模板



