[TOC]



## ReadMe

How to resolve the app crash on Linux, Mac, Win platform.

针对crash问题，应该是用堆栈去定位问题吧。



## Linux Platform

以下信号都能默认动作：产生core文件、终止exe运行：
abrt, 异步终止（abort）
bus, 硬件故障。
emt, 硬件故障。
fpe, 算术异常
ill, 非法硬件指令
iot, 硬件故障
quit, 终端退出符
segv, 无效存储访问
sys, 无效调用
trap, 硬件故障
xcpu, 超过cpu限制（setrlimit）
xfsz, 超过文件长度限制（setrlimit）





## Mac Platform



### Bus Error

Bus Error: 10





## Win Platform

### Dump File

crash一般会有dump文件产生，然后配置编译时候产生的<font color=gree>pdb文件</font>，结合来看carsh stack.
注：这个pdb文件就是一些debug信息，**debug版本能生成，release版也能生成**！



#### With Visual Studio

VS 打开dump文件；（dump与pdb同一目录）
Debug with Native Only



### WinDbg

Method 1. 可以分析dump文件；
Method 2. 也可以attach到进程，调试运行程序，直到生成dump；
​	step 2.1: file >> attach
​	step 2.2: g



## C++ Crash Code



### Segmentation Fault

Core Dump/Segmentation fault is a specific kind of error caused by accessing memory that <font color=red>“does not belong to you.”</font>, It is an error indicating memory corruption. There are may be the reasons as follow.

- There may be an attempt to write on a **read only memory** location.
-  Attempting to access memory location that doesn’t exist in our system.
  - May be freed.
- There may be an attempt to access **protected memory location** such as **kernel memory**.
- Stack overflow.
- Accessing out of array index bounds.



T.如何解决？
	M1.直接在生产环境gdb，看堆栈。`gdb -args exe exeArgs; run;`
	M2.生成core文件，拿到开发环境debug。

T.如何针对每次堆栈不一的segmentation？




### Buffer Overflow

It is an anomaly where a program, while writing data to a buffer, overruns the buffer’s boundary and overwrites adjacent memory locations.

```cpp
char buf[2] = "";
strcpy(buf, "over flow...");
```

Notice. 当更多的数据（比最初分配要存储的数据）被放入缓冲区内，额外的数据就会溢出。它会导致一些数据泄露到其他缓冲区中，这些缓冲区可能会破坏或覆盖已有的任何数据。在“缓冲区溢出”攻击中，额外的数据有时会为黑客、恶意用户的行为提供特定的指令（例如，数据可能触发一个响应，破坏文件、更改数据或公布隐私信息）。

缓冲区溢位有两种类型：
基于堆的，很难执行，而且两者中最不常见的是，通过淹没为程序保留的内存空间来攻击应用程序。
基于堆栈的缓冲区溢出，在攻击者中更为常见，利用所谓的堆栈（用于存储用户输入的内存空间）来利用应用程序和程序。



### Heap overflow

If we dynamically allocate large number of variables.

```cpp
int *ptr = (int *)malloc(sizeof(int)*10000000));
```



#### Memory Leak

If we allocate memory and we do not free that memory space after use it may result in memory leakage. 

T.如何定位？--当然是用工具了。
	win. umdh  （单个时间点的bt是没有名称、只有内存地址的）。
	linux. Valgrind
	注意事项：
		注意<font color=red>伪泄漏的情况</font>，一般如果请求是一样的话，泄漏点在每个请求处理的流程中都会触发，所以泄漏次数与client请求次数能对上号的基本上就是有问题的。
		注意<font color=red>异步任务</font>，有些空间是需要等待异步任务完成才能释放的，而非简单的按作用域。

T.如何测试？----长短时间结合。
	step1, 观察短时间的效应；（如用多少线程，每线程发送多少请求到server端，看server端的内存情况。）
	step2, 观察长时间的效应；（如client一直发请求，看server在1h内、4h内的内存情况。)



### Stack overflow

If we declare large number of local variables or declare an array or matrix or any higher dimensional array of large size can result in overflow of stack.

```cpp
int mat[100000][100000];
```



If function recursively call itself infinite times then the stack is unable to store large number of local variables used by every function call and will result in overflow of stack.

```cpp
int fun(int i)
{
    int ii = i;
    fun(i);
}
```



## CPU高占用

是不是只能从堆栈来看cpu卡在哪里？
	win. dump文件。（任务管理器》选中进程》创建转储文件）
	linux. core文件。（）
对比两次call stack？

T.cpu占用高的几个点：
	查看call stack是否有循环、特别是死循环； -----等待信号量是不消耗cpu的；
	有流量的场景处理流量耗cpu的地方；



## 工具篇



### umdh



### valgrind

