[TOC]

## ReadMe

glibc下socket编程接口；

**socket vs pipe**
socket是一个fd对应两个buffer、有对应的read/write buffer.
管道则是一个buffer对应两个fd，有对应的读、写fd；


**tcp vs udp**

面向连接，可靠数据包传输（对IP层进行优化）；
​    传输慢、开销大；
​    适用场景：追求数据的完整性要求高于效率，如文件传输、大数据传输；

无连接，不可靠的数据报传递；
​    传输快、开销小；
​    适用场景：追求时效性比数据完整性高，如游戏、视频会议电话；

UDP+自己封装应用层数据校验协议，也可弥补udp的不足；


**b/s vs c/s**

B/S
跨平台（部署成本低）、工作量少；

C/S
客户端可以缓存大量数据；速度快；自定义协议；（协议选择灵活）



## API

### socket

创建一个网络套接字；

```cpp
#include <sys/types.h>
#include <sys/socket.h>
int socket(int domain, int type, int protocol);
```



### Build tcp connection

建立tcp连接，需要以下代码协同完成。

```cpp
/* ---- server side code.  */
int bind(int sockfd , const struct sockaddr * my_addr, socklen_t addrlen);
int listen(int sockfd, int backlog); 
while (true) {
    int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
}


/* ---------client side code  */
int connect(int s, const struct sockaddr * name, int namelen);


/* ----------api des. */
int listen(int sockfd, int backlog);
//args
	//backlog, 指定AcceptQueue的长度。
//Syns Queue.
	//表示处于syn_recv状态的队列。
	//max(64, /proc/sys/net/ipv4/tcp_max_syn_backlog=2048)

int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
//Accept Queue.
	//表示已完成连接的队列，等待被accept函数取走。
	//min(backlog, /proc/sys/net/core/somaxconn), ----backlog是listen(lfd, backlog)中的backlog。
//Switch.
	///proc/sys/net/ipv4/tcp_abort_on_overflow = 0
		//队列满的时：如果为1，则在listen队列满的时候返回reset，如果为0，则还是正常三次握手。
	///proc/sys/net/ipv4/tcp_synack_retries = 5
//TCP_DEFER_ACCEPT option.
	//1. 收到第三次握手ack的时候，内核将这个连接标记为acked，然后把这个包丢掉。
	//2. 不往上传递accept，也就是应用层不会accept，并且连接保持在syn_recv的状态。
	//3. 重传超时计时器继续，也就是如果之后没有带数据的包过来，就会重传syn+ack，并且在超过syn次数之后，reset这个连接
	//4. 如果在重传之前，有数据包过来，才会带着数据包，将accept请求传递上去。

```





<font color=red>内核会为每个listen状态的socket维护两个队列！（syn/accept queue）</font>具体情况如下图：
http://jm.taobao.org/2017/05/25/525-1/

![如图，创建连接的过程](img/socket-tcpSyncAcceptQueue.jpg)

第一步，Client会发送一个SYN包，简单情况是SYN发送成功了，然后Client会把这个连接的Socket放入一个Socket等待队列，是Client这边维护的一个队列，<font color=blue>但是如果这里发送失败了，Server如果不给回复，它会按这个间隔去重新发送，3、6、12、24…重试十几次，会返回一个Connect Time out</font>.

第二步，Server收到SYN包，然后把这个Socket放入Server这边维护的<font color=red>SYN Queue</font>，然后返回SYN+ACK报文，syn queue size = <font color=red>max(64，/proc/sys/net/ipv4/tcp_max_syn_backlog)</font>.

第三步，Client收到Server发过来的ACK+SYN报文，相当于Client这边来看的话其和Server端的连接完成了，然后会返回一个ACK给Server。

第四步，正常情况是ACK收到，然后Server端看来建立也连接成功，然后把Socket从维护的SYN Queue放入Accept Queue，这样整个连接建立。但是Accept Queue也是有长度限制的。
<font color=blue>如果Accept队列满了，则需要按照内核参数/proc/sys/net/ipv4/tcp_abort_on_overflow来进行处理</font>：
如果==1，首先它会丢弃ACK，然后返回一个RST，这样的话就需要整个重新建立连接，Client会返回来一个Connection reset by peer。
如果==0，那么Server这边会不处理这个ACK，直接丢弃，~~那么Client是怎么知道的呢，其实是在靠Read函数来确定的，Client在确定建立连接之后会紧接着发数据给Server，但是Server还没有建立连接，所以它会不理会，然后会一直重发，直到超时，这时候返回一个Read timeout~~（这应该是正确的描述，不应该打删除线）, server过一段时间再次发送syn+ack给client（也就是重新走握手的第二步），<font color=red>形成重传的效果</font>，如果client超时等待比较短，就很容易异常了。（总体来说client觉得自己连上了，server认为这个连接没建立成功，于是会有大量的重传，不管是client的数据传输，还是server的连接建立包重传。----Rabin）
Accept Queue满了会drop掉握手的第1个包（syn）。  ----refer tcp_v4_conn_request()







### IO operation

#### tcp recv send

send
```cpp
if (recv() == -1) {
    if (errno == eagain/ewouldblock) {
        //设置了非阻塞方式读，但没有数据到达；
    }
    if (errno == eintr) {
        //eintr 意味着read这个慢系统调用被中断了；
    }
}
```


#### udp recvfrom sendto

```cpp
ssize_t recvfrom(fd, *buffer, bufferLen, recvFlag, 
                 *outputPeerAddr, *outputPeerAddrLen);
	//返回成功接收的数据；-1失败，0对端关闭

ssize_t sendto(fd, *buffer, bufferLen, sendFlag, *inputDstAddr, inputDstAddrLen);
	//返回发送成功的字节数，-1失败；
```



### close

关闭套接字，并释放资源；

```cpp
#include <unistd.h>
int close(int fd);
    //会把sock_fd的内部计数器减1；
    //并且当sock_fd == 0，则调用shutodwn() && 释放文件描述符；

#include <sys/socket.h>
int shutdown(int sockfd, int how);
    //只进行了TCP连接的断开, 并没有释放文件描述符
```

**close() VS shutdown()**
shutdown可以灵活指定关闭哪端（读端、写端、读写端）；close只能是写端？
shutown会重置fd的引用数为0（如之前dup出来的fd），然后关闭对应的端；而close只是fd的引用数-1，当为0时才会进行相应的操作；


**半关闭状态**
调用close()成功返回、那么不能再往对端写数据了；
fin包已经收到对端的ack了，但对端的fin未到达的这段时间；（只能收、不能发）

-----
**struct linger**

https://www.cnblogs.com/pengyusong/p/6434253.html




### sockaddr transform

网络环境用大端；
PC环境用小端；


**字节序**与系统有关还是cpu架构有关？？

字节序转换（双字节、四字节）
```cpp
#include <arpa/inet.h>

uint32_t htonl(uint32_t hostlong);
uint16_t htons(uint16_t hostshort);

uint32_t ntohl(uint32_t netlong);
uint16_t ntohs(uint16_t netshort);
```



[各种地址的定义查看：](./type.md#网络地址)
```cpp
char *inet_ntoa(struct in_addr in);
	//将ip地址转成字符串；
in_addr_t inet_addr(const char *cp);
	//转换cp为in_addr_t，即32位一整数；
int inet_aton(const char *cp, struct in_addr *inp);
	//将cp转换成inp；
	//返回非0如果cp为合法Ip；0如果cp无效；


inet_ntop();
inet_pton();
```


### setsockopt

设置、获取套接字的选项；

```cpp
int getsockopt(int sock, int level, int optname, void *optval, socklen_t *optlen);
int setsockopt(int sock, int level, int optname, const void *optval, socklen_t optlen);
	//成功，返回0；
	//失败，返回-1，errno有被设置。
```



#### SOL_SOCKET options

该级别的选项

|   选项名称    | 说明                   | 类型           |
| :-----------: | :--------------------- | :------------- |
| SO_BROADCAST  | 允许发送广播数据       | int            |
|   SO_DEBUG    | 允许调试               | int            |
| SO_DONTROUTE  | 不查找路由             | int            |
|   SO_ERROR    | 获得套接字错误         | int            |
| SO_KEEPALIVE  | 保持连接               | int            |
|   SO_LINGER   | 延迟关闭连接           | struct linger  |
| SO_OOBINLINE  | 带外数据放入正常数据流 | int            |
|   SO_RCVBUF   | 接收缓冲区大小         | int            |
|   SO_SNDBUF   | 发送缓冲区大小         | int            |
|  SO_RCVLOWAT  | 接收缓冲区下限         | int            |
|  SO_SNDLOWAT  | 发送缓冲区下限         | int            |
|  SO_RCVTIMEO  | 接收超时               | struct timeval |
|  SO_SNDTIMEO  | 发送超时               | struct timeval |
| SO_REUSERADDR | 允许重用本地地址和端口 | int            |
| SO_REUSEPORT  | linux3.9+, 重用端口    |                |
|    SO_TYPE    | 获得套接字类型         | int            |
| SO_BSDCOMPAT  | 与BSD系统兼容          | int            |



##### reuseAddr & reusePort

**SO_REUSEADDR**

SO_REUSEADDR只包括了adder而非port？？？ <font color=red>两者都包含</font>！

reuseaddr只能复用<font color=red>non-active listening</font>的socket。如下：

> socket1.bind(172.20.20.1, 8080); socket2.bind(172.21.20.1, 8080);
> ​	---这个不需要reuseaddr都可以。
>
> socket1.bind(172.20.20.1, 8080); socket2.bind(172.20.20.1, 8080);  
> ​	---这个是不可以的，但当socket1处于**non-active listening**的情况下是可以reuseaddr的（即未在socket1上调用listen()并成功的accept()连接之前），比如主动关闭时的time_wait.
> ​	---这些绑定同一ip,port的套接字并不是所有都能读取信息，**只有最后一个套接字会正常接收数据**。



**SO_REUSEPORT**

linux 3.9引入了SO_REUSEPORT，可以完美解决同一个ip, port的复用问题，同时也解决了惊群效应；

> socket1.bind(172.20.20.1, 8080); socket2.bind(172.20.20.1, 8080); 
> ​	----这种操作只要对socket设置上reuseport即可；

对应的编程模式也变了：

```cpp
// --------------------------------------------old Mode v1 as follow.
int main() {
    lfd = socket();
    bind(lfd, ..);
    listen(lfd, ..);
    while (true) {
        thread(lfd);
    }
    waitClose(); //block until the stop operation.
}
void* WorkerThreadPool.thread1(void*) {
    cfd = accept(lfd, ..); //这样当一个连接来时会有惊群效应；
    read(cfd, ..);
    write(cfd, ..);
    close(cfd);
}


// --------------------------------------------old Mode v2 as follow.
void* listenThread(void*) {
    lfd = socket();
    bind(lfd, ..);
    listen(lfd, ..);
    while (true) {
        cfd = accept(lfd, ..);
        //newTask and insert task into WorkerThreadPool.
        WorkerThreadPool.addTask(new Task(cfd));
    }
}
void* WorkerThreadPool.thread1(void*) {
    task = taskQeue.getTask(); //wait taskQueue has task and got one.
    read(cfd, ..);
    write(cfd, ..);
    close(cfd);
}


// --------------------------------------------New Mode as follow.
void* WorkerThreadPool.thread1(void*) {
    lfd = socket();
    setsockopt(lfd, SOL_SOCKET, SO_REUSEPORT, &val, sizeof(val));
    bind(lfd, "127.0.0.1", 8080);
    listen(lfd, 5);
    while (true) {
        cfd = accept(lfd, ..);
        read(cfd, ..);
        write(cfd, ..);
        close(cfd);
    }
}

void* WorkerThreadPool.thread2(void*) {
	//工作线程2，实现同工作线程1.
}
```

Notice. <font color=red>如上由哪个工作线程中的lfd来接收这个2元组(lip, lport)上的连接请求，由内核进行负载</font>；





#### IPPROTO_TCP options

该级别选项

|     选项名称     | 说明                                                         | 类型 |
| :--------------: | :----------------------------------------------------------- | :--- |
|    TCP_MAXSEG    | TCP最大数据段的大小                                          | int  |
|   TCP_NODELAY    | 不使用Nagle算法                                              | int  |
| TCP_DEFER_ACCEPT | 延迟accept()，保证只要accept()返回就会有数据到达，即recv()不会再阻塞住了。 | int  |





#### IPPROTO_IP options

该级别选项

|  选项名称  | 说明                 | 类型 |
| :--------: | :------------------- | :--- |
| IP_HDRINCL | 在数据包中包含IP首部 | int  |
| IP_OPTINOS | IP首部选项           | int  |
|   IP_TOS   | 服务类型             |      |
|   IP_TTL   | 生存时间             | int  |

Refer: http://www.cnblogs.com/eeexu123/p/5275783.html



## Socket & Error

各种错误码；

### EINTR

网络读写被信号所打断。

### sigpipe

socket向一个关闭的连接发数据，那么首次会得到一个rst包，再发会得到内核发送的sigpipe信号。
sigpipe默认动作：结束进程。

```cpp
#include <signal.h>
signal(SIGPIPIE, SIG_IGN);    
```

## IO Multiplexing 

一种机制，可以监视多个描述符，一旦某个描述符就绪（一般是读就绪或者写就绪），能够通知程序进行相应的读写操作。

select, poll, epoll
本质上它们都是同步io的手段（数据需要自己动手拷贝）即读写是阻塞的；
而异步I/O则无需自己负责进行读写，异步I/O的实现会负责把数据从内核拷贝到用户空间。


**select**
缺点：监听上限受文件描述符限制(1024，要修改只能去编译内核)；轮询满足条件的fd（3，4， 1023）这种场景需要额外编码来提高效率；
优点：跨平台，如win, linux, mac, 类unix, mips；


**poll**
优点：自带数组结构；监听事件集合与返回事件集合分离；突破1024监听限制；
缺点：不能跨平台（linux）; 无法直接定位到满足监听事件的fd，还是得靠轮询增加编码难度；




## UDP socket
udp通信
> sendto, recvfrom的次数要对上号。
> sendto, recvfrom中的参数需要指定对端的地址。

server peer as follow.

```cpp
lfd = socket(sock_dgram);
bind(lfd, ..);
while (true) {
    recvfrom(lfd, remoteAddr);
    sendto(lfd, dstAddr);
}
close(lfd);
```

client peer as follow.

```cpp
fd = socket(sock_dgram);
while (true) {
    sendto(fd, dstAddr);
    recvfrom(fd, remoteAddr);
}
close(fd);
```

## Unix Socket
本地socket是ipc的一种；
> pipe, fifo, mmap, signal, unix socket.

本地socket**编程模式同于tcp socket流程**（bind, listen, accept, connect），但区别如下：

```cpp
fd = socket(AF_UNIX/AF_LOCAL, anyType, 0); //diff. 协议可以任意选择；

#include <stddef.h>
#define offsetof(type, member) ((size_t)&((type*)0)->member)
	//((int)&((type*)0)->member) 只能用于32bit platform.
	//((long)&((type*)0)->member) 只能用于64bit platform.
struct sockaddr_un
{
    sa_family_t sun_family; //AF_UNIX.
    char sun_path[108];  //unix path.
}; //diff. 地址结构；
unlink(addr.sun_path);  //不同点：bind之前删除sun_path，保证bind成功；
bind(fd, addr, offsetof(struct sockaddr_un, sun_path)+strlen(addr.sun_path));
    //diff. 地址长度设置；
    //bind会创建一个伪文件sun_path，不占用磁盘空间（文件大小为0）。

socket(); bind(); listen(); accept(); recv(); send();
socket(); bind(); connect(); send(); recv();
    //diff. client在connect()之前需要bind()，不能依赖隐式绑定了！
```



-------
**size_t**

```cpp
//size_t defined 'unsigned int' on 32bit in libc.
//size_t defined 'unsigned long int' on 64bit in libc.
#if __WORDSIZE == 32   //on 32bit.
typedef unsigned int       size_t;
#else
typedef unsigned long int  size_t;
#endif
printf("size_t.%d int.%d long.%d ptr.%d\n", 
       sizeof(size_t), sizeof(int), sizeof(long), sizeof(int*));
	//size_t.4 int.4 long.4 ptr.4  ----on 32bit platform.
	//size_t.8 int.4 long.8 ptr.8  ----on 64bit platform.
	//指针的长度intptr_t 应该是unsigned long int型的；
```


| **位数** | **char**   | **short**   | **int**     | **long**        | **指针**        |
| -------- | ---------- | ----------- | ----------- | --------------- | --------------- |
| 16       | 1个字节8位 | 2个字节16位 | 2个字节16位 | **4个字节32位** | **2个字节16位** |
| 32       | 1个字节8位 | 2个字节16位 | 4个字节32位 | **4个字节32位** | **4个字节32位** |
| 64       | 1个字节8位 | 2个字节16位 | 4个字节32位 | **8个字节64位** | **8个字节64位** |



## Other









