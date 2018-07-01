
## 库

### 链接库

- 指定法

	```cpp
	-Wl,-rpath,/path1/libxx.so
	//运行期间生效，编译期间无效；
	//必须绝对路径；

	-lxx -L/path1
	//编译期间生效；
	//可以相对路径，亦可绝对路径；
	```

- 环境变量

- 配置文件

## 头文件


## 查看elf工具
用nm查看链接的库里有没有start\_thread\_noexcept这个符号表
```cpp
eventTst.cpp:(.text._ZN5boost6thread12start_threadEv[_ZN5boost6thread12start_threadEv]+0x15): undefined reference to `boost::thread::start_thread_noexcept()'
collect2: error: ld returned 1 exit status
make: *** [ev] Error 1
```

## 宏
探测OS类型
```bash
#if defined(__linux__)
	linux

#if defined(__APPLE__) && defined(__MACH__)
	mac

#ifdef WIN32
#if defined(WIN32)
	win32
```
