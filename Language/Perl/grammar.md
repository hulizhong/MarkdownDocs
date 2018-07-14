## ReadMe
perl的语法讲解

## 语法
语句
```perl
语句结尾;
```

注释
```perl
#单行注释

=name
多行注释
=cut
```

一句话也得打括号
```perl
if (xx) {  #前置if必须要大括号
	print "";  
}

print "Hello" if (xx);  #后置if不用大括号；
```

输入、输出
输出重定向
```perl
print "$var\n";
	默认向标准输出进行输出

print STDERR "$var\n";
	重定向到错误输出

open FP1 ">>log.txt" or die "cant open the log file";
print FP1 "$var\n";
close FP1;
	重定义输出到文件
```


## 变量
### 字符串
字符串的常见处理
```perl
my $val = 'Rabin';
my $sz = length($val);
	计算长度
my $newVal = substr($string,offset,length);
	offset开始截取的位置；如果是负数，那么从右边开始截取；
	length省略，那么截取到最后一个字符；
	截取子串
```

## 特殊符号
```perl
\p{xx}
	xx代表一个unicode属性；
```

## 文件操作
看段demo
```perl
open(FFILE, ">> /tmp/data");  追加模式打开文件
print FFILE $txt;    写入文件
print FFILE "\n-----------\n"; 追加分隔符
close(FFILE);
```

## 编码
perl只有两种字符串，如下：
> octets(Ascii), 8位序列，即字节数组；
> string, utf8编码的字符串；

可perl怎么区分这两种的呢？
> perl字符串 = 数据 + uft8 flag;
> utf8 flag 跟字符串是否真为 utf8 编码无关；

所以只要字符串内的utf8 flag被置on，那么就按照string来处理，反之就按octets来处理；
> 当字符串 utf8 flag 为 on 但非 utf8 编码时，print 会提示非法 utf8 字符。

utf8 flag处理，如下
```perl
use Encode;
Encode::is_utf8($str);  #查看utf8 flag是否开启；
Encode::_utf8_on($str);  #开启flag，内部函数不建议使用；
Encode::_utf8_off($str); #关闭flag，内部函数不建议使用；
```

### 问题点
用.连接字符串：flag是或关系；编码各自的原样；
> 只要其中任何一个是uft8 flag = on，那么结果是on的；
> 都没有on，那么结果是off的；

所以如果将两个不同编码的字符串连接在一起，那么之后不管怎么转码，都总会出现一段乱码；

---------

### unicode基本原则
对于任何要处理的unicode字符串，要：
> 将其编码转成utf8;
> 开启它的utf8 flag;

------
如果你的源代码里含有中文, 那么你最好遵循这个原则: 
1) 编写代码时使用utf8编码, 
2) 在文件的开头加上“use utf8;”语句。
这样, 你源代码里的字符串就都会是utf8编码的, 并且utf8 flag也已经打开。

------
转码：如果str是gb2312编码的，那怎么弄呢，如下：
```perl
$str = Encode::decode("gb2312", $str);  #转化为utf8编码并开启utf8 flag。

#以下三种场景：str本身就是utf8编码，只是没有开启utf8 flag.（开启utf8编码字符串的uft8 flag）
$str = Encode::decode_utf8($str);
$str = Encode::decode("utf8", $str);
Encode::_utf8_on($str);  #最简洁，但官方不推荐；
```

### Encode
perldoc encode
```perl
$octets = encode(ENCODING, $string [, CHECK])
	#把字符串从utf8编码转成指定的编码, 并关闭utf8 flag。

$string = decode(ENCODING, $octets [, CHECK])
	#把字符串从其他编码转成utf8编码, 并开启utf8 flag, 
	#不过有个例外就是, 如果字符串是仅仅ascii编码或EBCDIC编码的话, 不开启utf8 flag。
```

## swig
[python调用c++库](#python调用c++库)
[perl调用c++库](#perl调用c++库)

### python调用c++库
c++ & python
https://blog.csdn.net/u013378306/article/details/70172076

c++ xx.h xx.cpp文件，实现功能；
```cpp
int fun();
```

swig xx.i接口文件
```bash
%module <modulename>
%{
	...原样被输入到生成的_wrap.c/_warp.cxx中；
%}
int fun();
```

生成代码xx_wrap.cxx, xx.py
```bash
swig -c++ -python xx.i
```

编译
python自带一个distutils工具，可以用它来创建python的扩展模块。
```python
"""
setup.py
"""
from distutils.core import setup, Extension
 
example_module = Extension('_xx', 这个必须是_xx
                           sources=['xx_wrap.cxx', 'xx.cpp'],
                           )
setup (name = 'xx',
       version = '0.1',
       author      = "SWIG Docs",
       description = """Simple swig example from docs""",
       ext_modules = [xx_module],
       py_modules = ["xx"],
       )
```
```python
python setup.py build_ext --inplace   生成动态库xx.so
/usr/local/lib/python2.7/dist-packages  把生成的动态库放到python的site目录

import xx
print xx.fun();
```

### perl调用c++库
c++ xx.h xx.cpp文件，实现功能；
```cpp
int fun();
```

swig xx.i接口文件
```bash
%module <modulename>
%{
	...原样被输入到生成的_wrap.c/_warp.cxx中；
%}
extern int fun();
```

生成代码xx_wrap.cxx, modulename.pm （注意这点跟python不一样）
```bash
swig -c++ -perl xx.i
```

编译
```bash
g++ -c -fPIC xx.cpp
g++ -c -fPIC xx_wrap.cxx -I/usr/lib/perl5/5.14.2/CORE #这里面一堆的.h文件
g++ -shared xx.o xx_wrap.o -o moduleName.so  #这个库一定要同于example.
```

使用
```perl
use lib "/home/intranet/perl/swigPerl";   #手动定义库的路径
use GetStr;

my $str1 = GetStr::fun();
```

注意：
```cpp
char* getStr() {
	return "aa\0bb"; //这样返回只能在perl中得到aa，被截断了；
}
std::string getStr() {
	return std::string("aa\0bb"); //这样得到aa
	return std::string("aa\0bb", 6); //这样得到aabb
}

std::string需要在.i接口文件增加头文件
%include <std_string.i>
```
