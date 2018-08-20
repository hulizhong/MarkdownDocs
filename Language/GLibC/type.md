[TOC]

## ReadMe
各种类型；

## 基本类型
内存数据块大小
```cpp
#include <stddef.h>
size_t st; //代表对象的大小；
	//真实类型与操作系统有关（无符号）；而int是与平台无关的（恒为4字节）；
	//	typedef   unsigned int size_t;  4字节
	//	typedef  unsigned long size_t;  8字节
	//一般用于计数（内存操作，需要执行读写操作的数据块大小），在数组下标、内存管理函数类用的较多；	
ssize_t sst;
	//真实类型与操作系统有关。（有等号）
	//	typedef  int size_t;   4字节
	//	typedef  long size_t;  8字节
	//一般用于表示被执行读写操作的数据块大小；

//如，ssize_t msgrcv(int msqid, void *msgp, size_t msgsz, long msgtyp, int msgflg);
```

## 网络地址
[地址转换函数可参考：](./socket.md#地址转换)
sockaddr 
```cpp
#include <sys/socket.h>

struct sockaddr {
	sa_family_t sa_family;  //协议族，AF_xx
	char        sa_data[14];  //协议地址，包含ip,port；
};
```

sockaddr\_in 
in\_addr
```cpp
include <arpa/inet.h> or <netinet/in.h>

struct sockaddr_in {
    sa_family_t sin_family;
    in_port_t sin_port;  //网络字节序port；
    in_addr sin_addr;   //网络字节序ip；
    unsigned char sin_zero[8];  //为了与sockaddr保持相同大小，特意保留；
}
typedef uint32_t in_addr_t;
struct in_addr {         
	in_addr_t s_addr;
};

sockaddr_in mysock;
bzero(&mysock,sizeof(mysock));
mysock.sin_family = AF_INET;
mysock.sin_port = htons(800);
mysock.sin_addr.s_addr = inet_addr("192.168.1.0");
```



## 时间相关

timeval

```cpp
struct timeval {
    time_t      tv_sec;  //秒s
    suseconds_t tv_usec; //微秒us
};
1s = 1000ms = 1000000us
```