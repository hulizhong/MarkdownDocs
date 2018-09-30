
## 库
[如何生成动态库、静态库](#生成库)
[如何使用](#运行库)
[查看库时包含哪些内容](#查看库)

### 生成库
生成动态库
```bash
g++ demo.cpp -fPIC -c 
g++ demo.o -shared -o libdemo.so  //g++ demo.cpp -fPIC -shared -o libdemo.so
	shared生成动态连接库；
	fPIC生成位置独立的代码；
nm -s libdemo.so 查看库

g++ use.cpp -ldemo -L.
ldd a.out  查看目标加载了从哪里加载了哪些库
```

生成静态库
```bash
g++ -c dynamic_a.cpp dynamic_b.cpp dynamic_c.cpp  
ar cr libdemo.a dynamic_a.o dynamic_b.o dynamic_c.o  //cr标志告诉ar将object文件封装(archive)
nm -s libdemo.a

g++ use.cpp -ldemo -L. -static 其中static代表libdemo.a为静态库；
g++ use.cpp libdemo.a
```


### 运行库
使用库，应该是包含编译、链接二个方面；

- 编译
	- -lxx 编译器查找动态连接库时有隐含的命名规则；
		- 规则为：给定名字前面加上lib，后面加上.so来确定库的名称；
	- -L 要连接库的路径
		- 编译期间生效；
		- 可以相对路径，亦可绝对路径；
	- -static -l所链接的库为静态库；（更改隐含规则去找.a后缀的库，而非.so）
		- g++ main.cpp libxx.a也可直接这样用，而不是-l模式；

- 链接（链接器ld）
	- 默认/lib/, /usr/lib/
	- LD\_LIBRARY\_PATH
		- 配合在bashrc, profile文件里用LD_LIBRARY_PATH定义；
	- /etc/ld.so.conf, /etc/ld.so.conf.d/
		- ldconfig -> /etc/ld.so.cache
		- ld.so.cache是递增式增长，每次ldconfig；除非是重启机器了；
	- -Wl,-rpath,/path1/libxx.so 要连接库的路径（rpath前也有,号）
		- 运行期间生效，编译期间无效；
		- 必须绝对路径；
	

### 查看库
静态库、动态库的查看

```bash
nm -s libxx.so
nm -s libxx.a
```

静态库的查看ar 

```bash
ar cr libxx.a xx.o  #c 创建静态库; 
ar r libxx.a yy.o   #r 往静态库中增文件；
ar d libxx.a yy.o   #d 从静态库中删文件；
ar t libxx.a        #t 查看静态库中的文件；
```

目标文件使用了哪些库

```bash
ldd a.out
```




## 头文件


## 查看elf工具
编译失败，用nm查看链接的库里有没有start\_thread\_noexcept这个符号表
```cpp
eventTst.cpp:(.text._ZN5boost6thread12start_threadEv[_ZN5boost6thread12start_threadEv]+0x15): undefined reference to `boost::thread::start_thread_noexcept()'
collect2: error: ld returned 1 exit status
make: *** [ev] Error 1
```

## 特殊宏
### 探测OS类型

```bash
#if defined(__linux__)
linux platform code.
#endif

#if defined(__APPLE__) && defined(__MACH__)
mac platform code.
#endif

#if defined(_WIN32) || defined(WIN32)
win platform include 32bit and 64bit.
#endif

#ifdef _WIN64
win 64 bit platform.
#endif
```

