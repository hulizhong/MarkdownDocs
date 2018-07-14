## ReadMe
perl的语法讲解

## 语法
概要
```perl
语句结尾;

#单行注释

=name
多行注释
=cut
```


## 字符串
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

## 输出
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

## 特殊符号
```perl
\p{xx}
	xx代表一个unicode属性；
```

## 文件
看段demo
```perl
open(FFILE, ">> /tmp/data");  追加模式打开文件
print FFILE $txt;    写入文件
print FFILE "\n-----------\n"; 追加分隔符
close(FFILE);
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
