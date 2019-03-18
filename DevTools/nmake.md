[TOC]



## ReadMe

win平台上nmake（win下的Makefile）相关知识。



## 语法

### 指令

 nmake [选项] [ /f makefile ] [ /x stderrfile ] [ macrodefs ] [ targets ]

```bash
/f Makefile.nmke #指定Makefile文件，默认名为Makefile
/x stdout.file   #指定namke执行语句的输出文件。
```



### 变量

```bash
varName=xxx   #变量定义
$varName      #变量使用
$(varName)    #变量使用法2.
```



### 参数

```bash
/nologo   #不打印版权申明信息
/I"../include"    #添加头文件查找路径（如果路径中带有空格，一定要用引号括起来）

/Od  #优化选项：带入Debug信息
/O2  #优化选项：最快速度
/O1  #优化选项：最小尺寸

/W3　　#设置3级警告级别
/WX    #将Warining视为error

/DWIN32　　#预编译宏定义(win32程序)
/D_CONSOLE  #预编译宏定义（控制台程序）
/D_DEBUG  #预编译宏定义（Debug版本）

/LD 　　#创建动态链接库 
/LDd 　 #创建调试动态链接库

/ML 　　 #使用 libc.lib 创建单线程可执行文件 
/MLd 　　#使用 libcd.lib 创建调试单线程可执行文件 
/MT 　　 #使用 libcmt.lib 创建多线程可执行文件 
/MTd 　　#使用 libcmtd.lib 创建调试多线程可执行文件 
/MD 　　 #使用 msvcrt.lib/msvcrt.dll 创建多线程可执行文件
/MDd 　　#使用 msvcrtd.lib/msvcrtd.dll创建调试多线程可执行文件

/Z7 　　#生成与 C7.0兼容的调试信息 
/Zd 　　#生成行号 
/Zi 　　#生成完整的调试信息

/machine:X86 #指定目标平AM33|ARM|EBC|IA64|M32R|MIPS|SH3|SH3DSP|SH4|SH5|THUMB|X86|X64
```





### 规则推导

显式规则



隐式规则



## nmake with OSS

### libevent with nmake

更改Makefile.nmake

```bash
OPENSSL_DIR="D:\ProjectSWG\wfe\external            #设置openssl库位置
CFLAGS=$(CFLAGS) /D_DEBUG /Od /W3 /wd4996 /nologo  #增加/D_DEBUG，修改/Ox为/Od
LIBFLAGS=/nologo /machine:x6                       #增加x64
```

nmake /f Makefile.nmake



**fatal error LNK1112: module machine type 'X86' conflicts with target machine type 'x64'**

```bash
lib /nologo /machine:x64 event.obj buffer.obj bufferevent.obj bufferevent_sock.obj  bufferevent_pair.obj listener.obj evmap.obj log.obj evutil.obj  strlcpy.obj signal.obj bufferevent_filter.obj evthread.obj  bufferevent_ratelim.obj evutil_rand.obj evutil_time.obj win32select.obj evthread_win32.obj buffer_iocp.obj  event_iocp.obj bufferevent_async.obj /out:libevent_core.lib
event.obj : fatal error LNK1112: module machine type 'X86' conflicts with target machine type 'x64'
NMAKE : fatal error U1077: '"C:\Program Files (x86)\Microsoft Visual Studio 14.0\VC\BIN\lib.EXE"' : return code '0x458'
Stop.
```

从developer command prompt for vs2015 换成 vs2015 x64 native tools command prompt.



**fatal error U1073: don't know how to make 'print-winsock-errors.obj' **

```bash
cl /I.. /I../WIN32-Code /I../WIN32-Code/nmake /I../include /I../compat /DHAVE_CONFIG_H /DTINYTEST_LOCAL /DEVENT__HAVE_OPENSSL /Ox /W3 /wd4996 /nologo ..\libevent.lib ws2_32.lib shell32.lib advapi32.lib test-changelist.obj
LINK : warning LNK4098: defaultlib 'LIBCMTD' conflicts with use of other libs; use /NODEFAULTLIB:library
NMAKE : fatal error U1073: don't know how to make 'print-winsock-errors.obj'
Stop.
NMAKE : fatal error U1077: '"C:\Program Files (x86)\Microsoft Visual Studio 14.0\VC\BIN\amd64\nmake.exe"' : return code '0x2'
```

下载个libevent.tar.gz，从中的test/print-winsock-errors.c 进行编译。









