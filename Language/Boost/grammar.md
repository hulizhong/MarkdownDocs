[TOC]

## ReadMe
boost

- 是std的补充（TR1, C++11, C++14很多特性都是从boost库来的）。
- 什么时候用boost；
	- std不够用了；
	- gcc太老不支持std=c++11/14之类的；
- 学习方法
	- 它是很多库的集合，只需要做到用哪些库看哪些库。
	- 源码很复杂，会用就行；（到后期发展需要或者好奇心可以看下源码）


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



