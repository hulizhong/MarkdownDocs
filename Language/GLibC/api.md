[toc]

## 目录

- getcwd() 当前工作目录的绝对路径

	```cpp
	#include <unistd.h>

	//废弃不用了，用getcwd()
	char *getwd(char *buf);

	char *getcwd(char *buf, size_t size); 
		//getcwd(NULL, 0);  返回char*的当前目录路径
		//getcwd(buf, size);  当前目录放置于buf中；
	```


## eventfd

创建一个eventfd对象，用于进程、线程间的通知等待；（效率会比进程、线程间的同步机制快吗？） rwhy.
eventfd会在内核维护一个计数器（uint63\_t）；
```cpp
#include <sys/eventfd.h>
//参数initval，初始化内核计数器值；
//参数flags，
	// EFD_CLOEXEC (since Linux 2.6.27)，同于open的o_cloExec
	// EFD_NONBLOCK (since Linux 2.6.27)，设evfd为非阻塞；
	// EFD_SEMAPHORE (since Linux 2.6.30)，读取fd时，提供信号量语义；
int eventfd(unsigned int initval, int flags);
```
对evFd的读写都是8个字节的整数，如果read/write提供的buffer长度不为8，那么报EINVAL错误；
read(evfd)
> 从counter中读取8字节整数；
> 如果读不出来（即counter=0），那么会阻塞；
> nonblock直接返回，errno为EAGAIN

write(evfd)
> 向counter写入8字节整数，直到达到64位最大值，方可阻塞住；
> 如果写不进去，当nonblock直接返回，errno为EAGAIN

close(evfd)
> 只有当内核计数器为0才会释放evfd；


## 其它

```cpp
#include <assert.h>
void assert(scalar expression);
	//只在NDEBUG宏未开启时有效；否则该宏不会产生任务代码；
	//当expression为false那么向std error输出错误信息，并调用abort()终止程序运行；
```

