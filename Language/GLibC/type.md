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
	//unsigned >= 0, 恒成立；

ssize_t sst;
	//真实类型与操作系统有关。（有等号）
	//	typedef  int size_t;   4字节
	//	typedef  long size_t;  8字节
	//一般用于表示被执行读写操作的数据块大小；

off_t ot;
	//long int, 有正负之分，用于偏移量；

//如，ssize_t msgrcv(int msqid, void *msgp, size_t msgsz, long msgtyp, int msgflg);
```

整数相关

```cpp
//inttypes.h
typedef signed char             int8_t; 
typedef short int               int16_t;
typedef int                     int32_t;
# if __WORDSIZE == 64
typedef long int                int64_t;
# else
typedef long long int           int64_t;
# endif

typedef unsigned char           uint8_t;
//...
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

sockaddr\_in, in\_addr

```cpp
//include <arpa/inet.h> or <netinet/in.h>
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
//mysock.sin_addr.s_addr = htonl(INADDR_ANY);
```

sockaddr_in6, in6_addr

```cpp
struct sockaddr_in6 {
 in_port_t sin6_port;        /* Transport layer port # */
 uint32_t sin6_flowinfo;     /* IPv6 flow information */
 struct in6_addr sin6_addr;  /* IPv6 address */
 uint32_t sin6_scope_id;     /* IPv6 scope-id */
};

struct in6_addr{
 union {
     uint8_t __u6_addr8[16];
#if defined __USE_MISC || defined __USE_GNU
     uint16_t __u6_addr16[8];
     uint32_t __u6_addr32[4];
#endif
 } __in6_u;
};
```



ip匹配问题：v4可以换成整数进行，那么v6是不行的？ip段呢？---ip段可以比较ip整数范围！

```cpp
unsigned long inet_addr(const char FAR *cp); //1.1.1.1转成网络字节序。
	//转不了255.255.255.255，需要单独处理。
unsigned long inet_network(const char FAR *cp); //1.1.1.1转成主机字节序。
	//转不了255.255.255.255，需要单独处理。

int inet_aton(const char *cp, struct in_addr *inp);
	//返回的是网络字节序，能处理255.255.255.255.
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



```cpp
int gettimeofday (struct timeval * tv, struct timezone * tz);
struct timeval tv, tv2;
gettimeofday(&tv, NULL);
//...do things
gettimeofday(&tv1, NULL);
int ms = (tv1.tv_sec*1000 + tv1.tv_usec/1000) - (tv.tv_sec*1000 + tv.tv_usec/1000);
```

