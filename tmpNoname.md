[toc] 

## Notes.

nc
- nc -l 127.0.0.1 8888
- telnet 127.0.0.1 8888
	- ctrl + ]


API vs SDK
> Software Development Kit
> Application Programming Interface
>> Technically, if an API is well-documented, you don't need an SDK to build your own software to use the API. But having an SDK generally makes the process much easier.

- Easier integration 简单
- Faster time to market 快速
- Stronger security 安全
- Reliability 可靠
	- 新特性的引入，会发布新的sdk，sdk是经过验证的；


SIGABRT - 6
产生信号6的原因，如下：
- 多次free
- assert(*)
- abort()

gdb 
带参数调试
- gdb --args a.out 5
- set args 5; run|start
- run 5


## OSX

brew
不能安装软件 【osx不能用root来安装软件】
```bash
Lizhong$ sudo brew install gdb
Password:
Error: Running Homebrew as root is extremely dangerous and no longer supported.
As Homebrew does not drop privileges on installation you would be giving all
build scripts full access to your system.

Lizhong$ sudo chown -R $(whoami) /usr/local

Lizhong$ brew install gdb
	#注意前面不用sudo.
	#或许最开始没加suod，就不会有问题；
```

运行缺少库
```bash
$ ./spe_client_util3
dyld: Library not loaded: libboost_thread.dylib
> export DYLD_LIBRARY_PATH=$DYLD_LIBRARY_PATH:/Users/Lizhong/swg/external/lib/:/Users/Lizhong/swg/internal/lib/
>> 比linux多个DY
```


errno 55
```cpp
#define ENOANO 55 /* No anode */

strerror(errno)  //No buffer space available

```


```bash
$ sysctl -a | grep tcp.sendspace
net.inet.tcp.sendspace: 131072
$ sudo sysctl net.inet.tcp.sendspace=131072
```


gcc/g++判断OS类型的宏
```cpp
//OS X
defined(Macintosh)                         //Mac OS 9/X
(defined(__APPLE__) && defined(__MACH__))  //Defined by GNU C and Intel C++

//Linix
defined(__linux)      //老式的、废弃了
defined(linux)        //老式的、废弃了
defined(__linux__)

//Windows
defined(_WIN32)
defined(_WIN64)
defined(_WIN32_WCE)   //define by VS c++.
```

windows git
ssh key位置
> /c/Users/hulizhong/.ssh/id_rsa

git的配置文件（在安装目录下）
> xxx\Git\mingw64\etc\gitconfig



### 未解决的
#### gdb签名
- gdb安装后不能使用，还需要签名
```bash
(gdb) r
Starting program: /Users/Lizhong/swg/WebFilteringEngine/build/src/skyguard/policy_engine/tool/spe_client/a.out 
Unable to find Mach task port for process-id 20749: (os/kern) failure (0x5).
 (please check gdb is codesigned - see taskgated(8))
(gdb) 
```



- 生成证书之后执行以下命令：
```bash
bogon:~ Lizhong$ codesign -s gdb_codesign gdb
gdb: No such file or directory
bogon:~ Lizhong$
bogon:~ Lizhong$ which gdb       
/usr/local/bin/gdb
bogon:~ Lizhong$
bogon:~ Lizhong$ codesign -s gdb_codesign /usr/local/bin/gdb
/usr/local/bin/gdb: unknown error -1=ffffffffffffffff

```

- Need do command `codesign ...` in MacOS GUI mode.

- 但是每次启动gdb怎么都需要输入账号密码；



