[TOC]



## ReadMe

How to resolve the app crash on Linux, Mac, Win platform.

针对crash问题，应该是用堆栈去定位问题吧。



## Linux Platform







## Mac Platform



### Bus Error

Bus Error: 10





## Win Platform

### Dump File

crash一般会有dump文件产生，然后配置编译时候产生的pdb文件，结合来看carsh stack.



#### With Visual Studio

VS 打开dump文件；（dump与pdb同一目录）
Debug with Native Only





## C++ Crash Code



### Segmentation Fault

Core Dump/Segmentation fault is a specific kind of error caused by accessing memory that <font color=red>“does not belong to you.”</font>, It is an error indicating memory corruption. There are may be the reasons as follow.

- There may be an attempt to write on a **read only memory** location.
-  Attempting to access memory location that doesn’t exist in our system.
  - May be freed.
- There may be an attempt to access **protected memory location** such as **kernel memory**.
- Stack overflow.
- Accessing out of array index bounds.





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



