[TOC]

## ReadMe
c语法；

## 指针
const与指针
```cpp
//const和*的优先级相同；且是从右向左读的，即“右左法则”。
const char *ptr;  //*ptr指向const char，指针的指向的内容不能变；
char const *ptr;  //同上；
char* const ptr;  //const ptr指向char，指针指向不能变；
const char* const ptr;  //指向不能变、指向的内容也不能变；
```

## 头文件、实现文件

### 全局变量的定义、引用

一般的模式：在实现文件中定义，在头文件中引用extern；

> 尽量使用static把变量定义限制于该源文件作用域，除非变量被设计成全局的； 



extern用在变量或函数的声明前，用来说明“此变量/函数是在别处定义的，要在此处引用”。

引入一个问题
```bash
duplicate symbol __ZN8xx7yy14zz in:
    CMakeFiles/xx.dir/xx.cpp.o
    CMakeFiles/xx.dir/yy.cpp.o
ld: 1 duplicate symbol for architecture x86_64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
```
定义了2个xxyyzz全局变量；
在头文件中定义了全局变量，导致引入头文件就有一个全局变量；



#### extern C

extern "C"的主要作用就是为了能够正确实现C++代码调用其他C语言代码，这个功能十分有用处，原因如下：

> 在C++出现以前，很多代码都是C语言写的，而且很底层的库也是C语言写的，
> 为了更好的支持原来的C代码和已经写好的C语言库，需要在C++中尽可能的支持C，而extern "C"就是其中的一个策略。

加上extern "C"后，会指示编译器这部分代码按C语言的进行编译，而不是C++的。
> 由于C++支持函数重载，因此编译器编译函数的过程中会将函数的参数类型也加到函数名中；
> 而C语言并不支持函数重载，因此编译C语言代码的函数时不会带上函数的参数类型，一般只包括函数名。

引入一个c库的头文件；或者这样为c提供c++的库；
```cpp
#ifdef __cplusplus  //告诉编译器，这部分代码按C语言的格式进行编译，而不是C++的
extern "C"{
#endif
	//fun list...
#ifdef __cplusplus
}
#endif

#ifdef __cplusplus
extern "C"{
#endif
	#include "xx.h"
#ifdef __cplusplus
}
#endif
```

### 引入头文件
尽量在实现文件中引入所需的头文件；而非在头文件中引入所需的头文件；
```bash
common.h 公共头文件；
funA.h 
funA.c 在此处引入common.h
```



## 预编译

预编译，时在编译时期发生的，不满足条件的代码段不会被编译成汇编代码。

### 语法

如下，

```cpp
#define XX 5

/* grammar 1. -------------是否定义了这个宏。  */
#if defined (XX)
//是否定义了xx这个宏；
	//#if !defined (XX)
	//#ifdef XX
	//#ifndef XX
#endif


/* grammar 2.-----------------------宏条件是否满足。*/
#if (XX == 1)
//条件xx成立，时编译这一段
#elif (XX == 5)
//条件yy成立，时编译这一段
#else
//条件xx, yy不成立，时编译这一段
#endif
```

### OS特定宏

gcc/g++内置了如下宏，用以判断OS的类型，如下：

```cpp
#if defined(__linux__)
//linux platform code.
#endif

#if defined(__APPLE__) && defined(__MACH__)
//mac platform code.
#endif

#if defined(_WIN32) || defined(WIN32)
//win platform include 32bit and 64bit.
#endif

#ifdef _WIN64
//win 64 bit platform.
#endif
```

