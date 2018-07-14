[toc]

## ReadMe
各种类型；

## 基本类型
内存数据块大小
```cpp
#include <stddef.h>
size_t st; //代表对象的大小；
	真实类型与操作系统有关（无符号）；而int是与平台无关的（恒为4字节）；
		typedef   unsigned int size_t;  4字节
		typedef  unsigned long size_t;  8字节
	一般用于计数（内存操作，需要执行读写操作的数据块大小），在数组下标、内存管理函数类用的较多；	
ssize_t sst;
	真实类型与操作系统有关。（有等号）
		typedef  int size_t;   4字节
		typedef  long size_t;  8字节
	一般用于表示被执行读写操作的数据块大小；

如 ssize_t msgrcv(int msqid, void *msgp, size_t msgsz, long msgtyp, int msgflg);
```
