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



## 其它

```cpp
#include <assert.h>
void assert(scalar expression);
	//只在NDEBUG宏未开启时有效；否则该宏不会产生任务代码；
	//当expression为false那么向std error输出错误信息，并调用abort()终止程序运行；
```


