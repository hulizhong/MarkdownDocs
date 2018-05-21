
static用法
> 类中使用static，只用于声明；（不能用于定义）
>
> static成员函数只能操作static成员变量；
>> static成员函数即可以被对象调用，又可用类名调用；
>
> static成员变量即可以被static成员函数操作、又可以被非static成员函数操作；
>> static成员变量相当于类的全局变量；
>


- static.h

	```cpp
	#ifndef _STATIC_H
	#define _STATIC_H

	#include <vector>

	class Demo
	{
	public:
		static bool staticInsert(int num);

	private:
		static std::vector<int> mVar;
	};

	#endif
	```

- static.cpp

	```cpp
	#include<iostream>
	#include "static.h"
	using namespace std;

	//如果没有按以下定义：undefined reference to `Demo::mVar'
	std::vector<int> Demo::mVar;
	 
	//‘static’ may not be used when defining (as opposed to declaring) a static data member [-fpermissive]
	//如果定义时加static修饰
	//static std::vector<int> Demo::mVar;

	bool Demo::staticInsert(int num)
	{
		//method 1.
		mVar.push_back(num);
		//method 2.
		Demo::mVar.push_back(num);
		return true;
	}

	int main(int argc, char* argv[])
	{
		Demo::staticInsert(5);
		return 0;
	} 
	```

