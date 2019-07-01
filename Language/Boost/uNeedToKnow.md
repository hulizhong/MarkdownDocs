[TOC]

## ReadMe
**Q. 什么时候用boost ?**

- 是std的补充（TR1, C++11, C++14很多特性都是从boost库来的）。
- 什么时候用boost；
  - std不够用了；
  - gcc太老不支持std=c++11/14之类的；
- 学习方法
  - 它是很多库的集合，只需要做到用哪些库看哪些库。
  - 源码很复杂，会用就行；（到后期发展需要或者好奇心可以看下源码）



**Q. 需要知道boost的哪些特性？**

好，那么请看以下章节。




## RAII
构造函数时资源申请、析构函数时资源释放；

### boost\_scope\_exit
退出时调用{}中的内容；

demo
```cpp
#include <boost/scope_exit.hpp>
int fun()
{
	int fd = open("tst.md", "rb");
	//1. fd此时为按值传递（捕获），亦可按引用传递（捕获）；多个参数之间用逗号分隔；
	//2. 引用捕获时有些编译器不支持指针的引用捕获；
	//3. 类成员函数捕获类对象本身用this_；
	BOOST_SCOPE_EXIT(fd){
		//当程序离开fun()函数时，执行exit,exit_end之间的代码；
		close(fd);
	}BOOST_SCOPE_EXIT_END
}
```



## SmartPointer



## Thread & Lock



## Lambda & Function & Bind

lambda即匿名函数，代表着**就地声明、就地调用**的思想。
function代表着**延后调用**的思想。

### lambda

一个 lambda 表达式通常也称为匿名函数(*unnamed function*)。它在需要的时 候进行声明和定义，即就地进行。

```cpp
#include "boost/lambda/lambda.hpp"

boost::lambda::_1; //参数占位符；为lambda表达式指出了延后的参数。
```



**T. bind**

在lambda操作符不够用时就用绑定吧；

```cpp
#include <boost/lambda/bind.hpp>
```





### function

通常，我们要把函数对象传入算法，把 lambda 表达式存入另一个延后调用的函数，名为 `boost::function`

```cpp
#include <boost/function.hpp>

boost::function<bool (int, int)> f;
if (f) { //f可以判空；
    f(1, 2);
}
```



**T. bind**

需要封装function、包装参数时，就用bind吧；

```cpp
#include <boost/bind.hpp>
```

