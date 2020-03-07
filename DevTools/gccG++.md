[TOC]

## ReadMe

gcc, g++相关。



问题：debug版本、relese版本与`gcc -g`有什么关系？

> 没有关系。
> debug/release版本是用宏来实现的，如gcc -Ddebug_macro，而gcc没有此类相关的内置宏。
> cc -g是在生成的二进制文件里增加调试信息，用于gdb等调试工具。
>
> 那么问题来了？能让断言失败assert()的宏NDEBUG是怎么回事？



## 指令集

### 编译阶段

Gcc编译过程主要的4个阶段：

1. 预处理阶段，完成宏定义和include文件展开等工作；（gcc -E xx.c -o xx.i）
   1. 处理的指令有：`#include, #define, #ifdef, #ifndef, #if, #endif, #undef, #pragma`...
2. 编译阶段，根据编译参数进行不同程度的优化，编译成`汇编代码`(gcc -S xx.c xx.s/xx.s)
3. 汇编阶段，用汇编器把`汇编代码`进一步生成`目标代码`(gcc -c xx.c -o xx.o)
4. 链接阶段，用`连接器`把生成的目标代码和系统、用户提供的库连接起来，生成可执行文件(cc -o xx xx.c)
   1. 静态库：把库文件的代码加载到目标文件中，生成的可执行文件大、运行时不依赖环境。
   2. 动态库：不会把.............................................，生成的可执行文件小、运行时依赖库。
      1. -Wl,-rpath,/path1/libxx.so
      2. LD\_LIBRARY\_PATH
      3. /etc/ld.so.conf, /etc/ld.so.conf.d/
      4. 默认/lib/, /usr/lib/



#### 预编译

预编译，时在编译时期发生的，不满足条件的代码段不会被编译成汇编代码。

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

**OS特定宏**
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

//_DEBUG  表示debug编译，否则为release编译；

```







### 指令

调试、优化：

```bash
#gcc -ggdb3 -O0 xx.c
	#-g, -gLevel, -ggdbLevel
		#-g, 只是编译器，在编译的时候，产生调试信息。
		#-ggdb, 尽可能的生成gdb的可以使用的调试信息。
		#默认是2级别、0是去除调试信息，1是最小化调试信息，3是增加宏信息。
	#-O
```





## 静态、动态库

库相关。

### 生成库

#### on Linux

生成动态库（<font color=red>默认所有函数都对外开放</font>）
```bash
g++ demo.cpp -fPIC -c 
g++ demo.o -shared -o libdemo.so  //g++ demo.cpp -fPIC -shared -o libdemo.so
	#shared生成动态连接库；
	#fPIC生成位置独立的代码；
g++ -fvisibility=default|internal|hidden|protected
	# __attribute ((visibility("public"))) int test2 (int i) linux所有函数下默认声明为此
	#default，默认选项，等效于public.
	#hidden, 隐藏，只有声明了__attribute ((visibility("default")))的函数才会被导出。
	#internal|protected，很少使用

g++ use.cpp -ldemo -L.
```

生成静态库
```bash
g++ -c dynamic_a.cpp dynamic_b.cpp dynamic_c.cpp  
ar cr libdemo.a dynamic_a.o dynamic_b.o dynamic_c.o  #cr标志告诉ar将object文件封装(archive)

g++ use.cpp -ldemo -L. -static #其中static代表libdemo.a为静态库；
g++ use.cpp libdemo.a
```

#### on Mac

#### on Win

静态链接库会生成`xx.lib`，动态库链接会生成`xx.dll, xx.lib`，两者都有lib文件，但实质上是不一样的。

> 前者是静态库（包含了实际执行的代码、符号表等）。
> 后者是<font color=red>导入/引入库</font>（只包含了地址符号表等，用于确保程序能在dll中找到对应的函数，执行代码都在dll文件中）。

<font color=red>动态链接库内的函数分两种：（静态链接库则没有这种说法）</font>

> 一、内部函数，只供库内部使用。（<font color=red>默认为此开关</font>）
> 二、导出函数，可以给外部使用，但需要按如下方式进行。
>
> - \_\_declspec(dllexport)方式进行声明
> - DEF文件方式进行声明

导出函数的声明代码，如下：

```cpp
// ----------------------dllexport方式
// xx.h
__declspec(dllexport) int add(int a, int b);
// xx.cpp
#include <xx.h>
__declspec(dllexport) int add(int a, int b) {...}

// ----------------------def方式
// dll.def
LIBRARY BlogUse
EXPORTS add @ 1
// xx.h
int add(int a, int b);
// xx.cpp
#include <xx.h>
int add(int a, int b) {...}

// ------------------------cmake.generate_export_header
```



静态链接库的使用方式，如下：

```cpp
#pragma comment(lib, "xx.lib")
```

动态链接库使用方法，如下：

```cpp
#pragma comment(lib, "xx.lib")
	//静态链接方式
LoadLibrary(..);
GetProcAddress(..); //需要用'函数指针'来接收动态库中的函数运行地址。
FreddLibrary(..);
	//动态链接方式

//__declspec(dllexport)导出时，调用前需要声明 __declspec(dllimport) add(int a,int b)???
//不需要，只需要#include ”dll.h"即可，.h中int add(int a,int b);
```





### 使用库

使用库，应该是包含编译、链接（静态、动态：生成文件大小、运行时依赖）二个方面；

#### on Linux

- 编译
  - -lxx 编译器查找动态连接库时有隐含的命名规则；
  	- 规则为：给定名字前面加上lib，后面加上.so来确定库的名称；
  - -L 要连接库的路径
  	- 编译期间生效；
  	- 可以相对路径，亦可绝对路径；
  	- LIBRARY_PATH, 也可以在这个环境变量中找吧？？ --rwhy??
  - -static -l所链接的库为静态库；（更改隐含规则去找.a后缀的库，而非.so）
  	- g++ main.cpp libxx.a也可直接这样用，而不是-l模式；
- 链接（链接器ld）即运行，时依赖优先级。
  1. -Wl,-rpath,/path1/libxx.so 要连接库的路径（rpath前也有,号）
     1. 运行期间生效，编译期间无效；
     2. 必须绝对路径；
  2. LD\_LIBRARY\_PATH
     1. 配合在bashrc, profile文件里用LD_LIBRARY_PATH定义；
  3. /etc/ld.so.conf, /etc/ld.so.conf.d/
     1. ldconfig -> /etc/ld.so.cache
     2. ld.so.cache是递增式增长，每次ldconfig；除非是重启机器了；
  4. 默认/lib/, /usr/lib/



#### on Mac

#### on Win



#### dlpopen

```cpp
void * dlopen( const char * pathname, int mode ); //打开动态库。
	//pathname不是绝对路径，则按次序查找库路径。
	//如果x.so依赖y.so，那么先加载y.so后才能加载x.so.
int dlclose (void *handle); //只有计数为0，库才会把系统卸载。

void* dlsym(void* handle,const char* symbol); //获取库的函数的指针。
const char *dlerror(void); //对动态库操作失败的原因。
```



### 查看库
静态库、动态库的查看

```bash
# 动态库有导出符号这一说、静态库则没有。----------nm libxx.a导出符号是T表示。

# ---------------------linux
nm -s libxx.so
nm -s libxx.a
readelf -a libxx.so
//readelf -s ? nm -s 如何看隐藏的符号表。？？？？
# nm -s libxx.so 内部函数
0000000000000980 t _ZN1A4funaEv
0000000000000a14 t _ZN1B4funbEv
# nm -s libxx.so 外部函数
00000000000009e0 T _ZN1A4funaEv
0000000000000a74 T _ZN1B4funbEv
# readelf -s libxx.so 内部函数
50: 0000000000000a14    54 FUNC    LOCAL  DEFAULT   12 _ZN1B4funbEv
51: 0000000000000980    54 FUNC    LOCAL  DEFAULT   12 _ZN1A4funaEv
# readelf -s libxx.so 外部函数
62: 0000000000000a74    54 FUNC    GLOBAL DEFAULT   12 _ZN1B4funbEv
64: 00000000000009e0    54 FUNC    GLOBAL DEFAULT   12 _ZN1A4funaEv

# ---------------------win
dumpbin /exports xx.dll
dumpbin /linkermember xx.lib  #静态库
dumpbin /headers xx.lib
	#... machine (x86)  代表32位
	#... machine (x64)  代表64位

# ---------------------mac
```

静态库的查看ar 

```bash
# ---------------------linux
ar cr libxx.a xx.o  #c 创建静态库; 
ar r libxx.a yy.o   #r 往静态库中增文件；
ar d libxx.a yy.o   #d 从静态库中删文件；
ar t libxx.a        #t 查看静态库中的文件；

# ---------------------win

# ---------------------mac
```



### 查看二进制执行文件的依赖库

目标文件使用了哪些库

```bash
# ---------------------linux
ldd a.out
ldd a.out  #查看目标加载了从哪里加载了哪些库


# ---------------------win

# ---------------------mac
```




## 头文件

## ELF文件

linux下可执行文件的格式，运行起来的进程空间参考：（Language\GLibC\process.md）



### 查看工具

编译失败，用nm查看链接的库里有没有start\_thread\_noexcept这个符号表
```cpp
eventTst.cpp:(.text._ZN5boost6thread12start_threadEv[_ZN5boost6thread12start_threadEv]+0x15): undefined reference to `boost::thread::start_thread_noexcept()'
collect2: error: ld returned 1 exit status
make: *** [ev] Error 1
```



## PE文件

### 查看x86_64, x86

如果使用16进制编辑器（ <font color=red>%!xxd</font>）打开你的exe文件的话，可以看到如图的效果，里面的hex code:  <font color=red>5045 0000 4C</font>就表示是32位的，而hex code: <font color=red>5045 0000 64</font>86就表示是64_86，也就是64位的。

```bash
000000d0: 5045 0000 4c01 0600 3f3f 474f 0000 0000  PE..L...??GO....

00000120: 0000 0000 0000 0000 5045 0000 643f 0800  ........PE..d?..
```

### Windows-on-Windows x-bit

WOW64 (Windows-on-Windows 64-bit)是一个Windows操作系统的子系统, 它为现有的 32 位应用程序提供了 32 位的模拟，可以使大多数 32 位应用程序在无需修改的情况下运行在 Windows 64 位版本上。

WOW64类似于旧的 WOW32 子系统，负责在 Windows 32 位版本下运行 16 位的代码。





​	