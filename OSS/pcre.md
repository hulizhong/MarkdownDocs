[TOC]



## ReadMe

PCRE(Perl Compatible Regular Expressions，即perl语言兼容正则表达式)
是一个用C语言编写的正则表达式函数库，PCRE是一个轻量级的函数库，比Boost之类的正则表达式库小得多。
PCRE十分易用，同时功能也很强大，性能超过了POSIX正则表达式库和一些经典的正则表达式库。
PCRE是用C语言实现的，其C++实现版本是PCRE++。



## 编译、安装



### 特性

多字节特性：那么其对应的api调用为`pcre16_exec, pcre32_exec`.

```bash
./configure --enable-pcre16 --enable-pcre32 --enable-utf8
	#--enable-pcre16: This switch enables 16 bit character support.
	#--enable-pcre32: This switch enables 32 bit character support.
```



jit特性：



## api

将一个正则表达式编译成一个内部表示，在匹配多个字符串时，可以加速匹配。

```cpp
pcre *pcre_compile(const char *pattern, int options, const char **errptr, int *erroffset, const unsigned char *tableptr);
	//pattern   正则表达式
	//options   为0，或者其他参数选项
	//errptr   出错消息
	//erroffset  出错位置
	//tableptr  指向一个字符数组的指针，可以设置为空NULL。 
pcre *pcre_compile2(const char *pattern, int options, int *errorcodeptr, const char **errptr, int *erroffset, const unsigned char *tableptr);
	//errorcodeptr 存放出错码
```



在`pcre_compile`之后会得到pcre的一个内部表示`struct real_pcre`，此时可以进一步调用`pcre_study`对此 结构进行分析和学习，得到一个数据结构`struc pcre_extra`，这个结构可以传送送给`pcre_exec`进行匹配。

```cpp
pcre_extra *pcre_study(const pcre *code, int options, const char **errptr);
	//对编译的模式进行学习，提取可以加速匹配过程的信息。
```



使用编译好的模式进行匹配，采用与Perl相似的算法，返回匹配串的偏移位置。

```cpp
int pcre_exec(const pcre *code, const pcre_extra *extra, const char *subject, int length, int startoffset, int options, int *ovector, int ovecsize);
	//code         编译好的模式
	//extra        指向一个pcre_extra结构体，可以为NULL
	//subject      需要匹配的字符串
	//length       匹配的字符串长度（Byte）
	//startoffset  匹配的开始位置
	//options      选项位
	//ovector      指向一个结果的整型数组。
	//ovecsize     数组大小。-----rabin.注意数组大小只能是3的位数，3n大小最多只能匹配n个。
	//返回值：0未匹配，>0匹配的个数，<0匹配错误。
```



释放

```cpp
//pcre_free;
//pcre_free_study;
```





# Errors

lookbehind assertion is not fixed length

```cpp
(?<![0-9A-Z]+)[0-9A-Z]{12}(?![0-9A-Z]+)
(?<![0-9]+)[0-9]{15}(?![0-9]+)

//PCRE2: lookbehind assertion is not fixed length. ----反向预查（断言）在有些语言、或库里只支持固定长度的；
(?<![0-9A-Z]{1})[0-9A-Z]{12}(?![0-9A-Z]+)
(?<![0-9]{1})[0-9]{15}(?![0-9]+)
```









