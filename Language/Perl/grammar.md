## ReadMe
perl的语法讲解

.pm  模块文件；可以用 use xx来加载。（use语句会自动搜索后缀为.pm的文件）
.pl  库文件；可是现在大家都用来当脚本文件；用require xx.pl来加载。
.pxl  脚本文件。


demo
```perl
#!/usr/bin/perl -w
	#-w表示使用严格的语法控制。
use warnings;  #同于-w，但不是全局性的，可在{}中进行引用；
use strict;
	#是Perl中编译指令，告诉编译器，如果perl代码中有不好的编码风格，那么提示编译失败。
	#代码的编写必须遵循一些规范，否则编译器会报错。
```


## 语法
简义的语法；

### 简明语法
概要
```perl
# 语句以';'结尾。
# 用{}进行作用域分隔。

#单行注释
=name
多行注释
=cut

#-------------------------------------一句话也得打括号
if (xx) {  #前置if必须要大括号
	print "";
}
print "Hello" if (xx);  #后置if不用大括号；

#---------------------------------输入、输出
print "$var\n";
	#默认向标准输出进行输出
print STDERR "$var\n";
	#重定向到错误输出
print "<", $var, ">\n";
	#print用法同于python，但还是有区别的；
	#python print
		#会自动添加换行符号；
		#不同输出项之间的连接符为空格，如上的输出为'< var >'
	#perl print
		#不会自动添加换行符号；
		#不同输出项之间没有连接符；<var>

open FP1 ">>log.txt" or die "cant open the log file";
print FP1 "$var\n";
close FP1;
	#重定义输出到文件
```

### 各种循环
```perl
#--------------------------------------------for
for (init; condition; increment) {
}

#--------------------------------------------foreach
foreach $var (@list) {
	#use strict模式下不能用此，只能用第三种方法；
}

foreach $var (@list) {
	#第二种方法；
} continue {
}

foreach (@arr) {
	#第三种方法；
	print $_, "\n";
}

#--------------------------------------------while
while (condition) {
}

while (condition) {
} continue {
	#在condition之前进行
}

do {
} while (condition)

until (condition) {
	#与while相反，条件为false方可执行；
}
```

循环控制
```perl
while (condition) {
}

while (condition) {
	last; #退出循环
	next label;  #跳过以下各句，直接执行continue部分、并进入下一次循环
	redo;  #语句直接转到循环体的第一行开始重复执行本次循环，（continue + condition不执行）
	...
} continue {
	...
}

goto label;
goto &label; #替换正在运行的子进程；
```

### 函数
声明、调用 ----<font color=yellow size=4>声明先于调用、使用进行！（不管是变量、还是函数）</font>
```perl
sub xx { #不需要形参数列表，但可接收参数；
	@_; #就是参数栈，其实就是数组。
	$_[0], $_[1]; #获取参数，下标索引法。
	shift, pop;   #获取栈开头、结尾。

	return xx;    #可以返回一个标量值或者一个列表
	return (xx, yy);    #可以返回一个标量值或者一个列表
	return (\@xx, \@yy);    #数组列表需要引用，否则会被平铺成一个数组；
}

sub yy() { #不能接收参数；
}

my $a = &xx(params list);  #version 5.0以下调用方式；
my ($a, $b) = xx(params list);
```

传参
```perl
sub xx {
}

my $v1 = 1;
my @ar1;
xx($v1, @ar1);  #传列表，@ar1必须要放在最后；（因为函数用于接收参数为一数组@_）

my %map;
xx($v1, %map);  #传入函数内，会被扩展成key,value的列表。
```

### 引用

为什么需要引用呢？

- **函数传参只接受`标量`**。
- **列表、哈希中的元素只支持`标量`。借助引用可构造复杂的结构**。
  - $hash{"key"} = @array;  <font color=blue>那么只是把array的元素个数赋值给key</font>。

引用 & 解引用（<font color=red>法一：$var代替var。法二：{$var}代替var。法三：->[idx],->{"key"},->(params)</font>）

- 标量

  ```perl
  my $v = 1;
  my $refScalar = \$v;
  
  $$refScalar;
  ${$refScalar};
  ```

- 数组

  ```perl
  my @v = (1, 2);
  my $refArray = \@v;
  my $refArray = [...];
  	#匿名列表，[]。（注意左值的类型为'$'）
  
  @$refArray;   #全部
  @{$refArray};    #全部，可用于返回[]/{}中的[]引用。
  $refArray->[idx];  #单一元素
  $$refArray[idx];   #单一元素
  ```

- 哈希

  ```perl
  my %v = ("key"=>"val");
  my $refHash = \%v;
  my $refHash = {key=>v1, key=>v2};
  	#匿名hash，{}。（注意左值的类型为'$'）
  
  %$refHash;   #全部
  %{$refHash};   #全部，可用于返回[]/{}中的{}引用。
  $refHash->{"keyname"};  #单一元素
  $$refHash{"keyname"};  #单一元素
  ```

- 函数

  ```perl
  sub fun {...}
  my $refFunction = \&fun;
  	#注意：fun后面没有()，否则为调用关系了！！
  my $refFunction = sub {..}
  	#匿名子过程。（注意左值的类型为'$'）
  
  $refFunction->();
  &$refFunction();
  ```

- glob句柄

  ```perl
  $refHandle = \*STDOUT;
  ```

- none



解引用相关

- 判断

  ```perl
  if (ref $var) {
  	#如果var是引用，则会返回真；
  	#perl中只有引用类型，才可进行类型探测；因为普通类型可以看变量名的前缀，如@, %。
  }
  if ($var1 == $var2);
  if ($var1 eq $var2) {
  	#判断两个引用是否指向同一目标；
  }
  ```

- 解引用方法对比（）

  - $$ref VS ${$ref}，后者能从[]/{}解析出[]/{}的引用。
  - `$ref->[idx]/{"key"}/(params)`可存取单一元素，或者进行函数调用。 



### 运算符号

```perl
q{abc}   'abc'
qq{abc}  "abc"
qx{abc}  `abc`
qr/pattern/;
qw{aa bb} #单词列表引号；'aa', 'bb'
~~  #智能匹配
	#匹配某个元素是否在数组中；
	#匹配两个数组中是否有相同元素；
	#正则匹配，替代=~而且比之还强大；

if ($a <=> $b);
	#相等、返回0
	#a<b、返回-1
	#a>b、返回1

$a += 1;
$a -= 1;
$a *= 1;

if ($a and $b);
if ($a or $b);
not if ($a);
if ($a && $b);  #c风格
if ($a || $b);  #c风格
```

| 运算符符（优先级从上到下）   | 结合性   |
| ---------------------------- | -------- |
| ++, --                       | 无       |
| -, ~, !                      | 从右到左 |
| **                           | 从右到左 |
| =~, !~                       | 从左到右 |
| *, /, %, x                   | 从左到右 |
| +, -, .                      | 从左到右 |
| <<, >>                       | 从左到右 |
| -e, -r,                      | 无       |
| <, <=, >, >=, lt, le, gt, ge | 从左到右 |
| ==, !=, <=>, eq, ne, cmp     | 从左到右 |
| &                            | 从左到右 |
| \|, ^                        | 从左到右 |
| &&                           | 从左到右 |
| \|\|                         | 从左到右 |
| ..                           | 从左到右 |
| ? and :                      | 从右到左 |
| =, +=, -=, *=,               | 从右到左 |
| 其他                         |          |
| ,                            | 从左到右 |
| not                          | 从左到右 |
| and                          | 从左到右 |
| or, xor                      | 从左到右 |



### 包、模块

定义包
```perl
package packageName;   #包开始
...
1;   #包结尾，这个结尾是不是可以不用哈？？rwhy
```

引用包
```perl
require packageName;
packageName::fun();

use packageName;
fun();
```

## 变量
预置的各种变量类型；

**bool类型**：perl中没有bool，所以如下：（"!" 或 "not" 否定一个真值）

```perl
--------------------------如下为false
undef  #未定义值
0      #数字0, 即便你写的是000或0.0
''     #空字符串.
	#' '为true
'0'    #包含单个0数字的字符串.
```

**生命周期**：变量生命周期

```perl
use strict;    #指令强制规定必须声明变量

package test;

#-----------------------------------------------------包域全局our，外部可见。
##our操作符用于显式地创建包作用域变量。
##如果全局变量已存在，则our的作用是声明这个全局变量（类似于C中的extern）。
our $x = 5;  #全局变量。
{
	print $x; #包内使用。
    print $test::x;  #包外使用。
    print $main::x;  #包外使用，x定义在没有packagename的pl文件里。
}


#-----------------------------------------------------临时全局local。
{
	local $x = 20;  #全局变量局部的副本；跟全局的x没有关系；
		#作用域，只在{}内。
}


#-----------------------------------------------------私有局部my。
##存活于声明开始的地方，直到闭合作用域的结尾、或者文件结束。
my $var = 2;  #私有变量，作用域为这个文件。
{
	my $var = 1;  #私有变量，作用域为这个{}。
}


#-----------------------------------------------------持久局部state。
##类似my，但也my不同的是遇到'}'后，其值还是会被保留。
##仅能创建闭合作用域为子程序内部的变量。
use feature 'state';
{
	state $v = 0;   #static属性、具有记忆功能；
}

1;
```

### 标量
#### 整数

```perl
#拆分整数
my $num = "12345";
my @numArr = split(//, $num);


```



#### 浮点数
#### 字符串
字符串的常见处理
```perl
my $str = 'Rabin';

my $sz = length($val);  #计算长度
$str eq "strvalue";
	#字符串的对比，不能用==, <之类的（专用于数字类型对比）；
	#lt, eq==, gt, le=<, ge>=, ne!=, cmp返回1、0、-1;
print $a, $b;   #print的连接符号','只能用于print；
$value = 1234.56789;
print sprintf "%.4f\n", $value;
	#sprintf打印格式；

#-------------------------------------------------------------大、小写互转！
$str2 = lc($str);  #一定要赋值给自己哈！！要不没有效果！！
$str2 = uc($str)
$str2 = lcfirst $str;   #第1个字母转小写
$str2 = ucfirst $str;   #第1个字母转大写

#--------------------------------------------------------------查找
index(str，key，beginPos=0)
	#顺序查找，找不到返回-1。
	#find(str, beg=0, end=len(string))  ----没有这种用法吧；
rindex(str, key, beginPos=0)
	#逆序查找。

$count = grep(/on/, @array)
	#匹配数组内字符串；其中//是固定修饰字符；
@result = grep(/on/, @array);
	#匹配数组内字符串；

#--------------------------------------------------------------截取子串
my $newVal = substr($string, offset, length);
	#offset开始截取的位置；如果是负数，那么从右边开始截取；
	#length省略，那么截取到最后一个字符；
substr($src, index($src, $srcb), length($srcb), $srcc);
	#查找并替换；

#--------------------------------------------------------------连接、拆分
join($str, @array)      #字符串合并；其中arr要用qw方式定义；
$line = $line . "456";  #字符串连接；
$line .= "456";
print "ok"x3; #okokok   #重复连接符号x；

@arr = split(/pattern/, $str);   #字符串分隔；
@arr = split(//, $str);    #强制按每个字符进行分隔；
	#可用于遍历字符串；（先分隔成数组，再遍历数组内容）
```

字符处理

```perl
chr(65);  #整数转ASCII字母；
ord(A);   #ascii字母转整数；

#--------------------------------------------------------------判断字符是否是数字
use POSIX qw (isdigit)
if (isdigit('A')) {;}
if ('A' =~ m/\d/) {;}
```





### 数组、列表

使用
```perl
my @arr = ($arg1, $arg2);
my @arr = qw/a b c/;
@array = (1..10);
$size = scalar @array;  #数组长度；

$arr[0];     #下标法，注意最开始的修饰符不是@；
@arr[1..3];  #下标法；
@arr[1,2,4];

pop @arr      #弹出数组最后一个值，并返回它。
shift @arr    #从数组的开头取出元素

push @arr, list    #从数组末尾加入元素
unshift @arr, item #从数组的开头加入元素
```

**Notice**: 定义、使用，注意如下：

- <font color=red>定义使用`@ + ()`，注意不是`[]`。 ---那是匿名引用的`$ + []`</font>。
- 使用时用`$`。

**Notice**: 匿名数组引用，注意如下

- 定义为`$ + []`。

  ```perl
  my $arr_r = [1, 2];
  ```

- 使用时用`-> 或者 $$`。

  ```perl
  $arr_r->[0] = 'c';  #正确
  
  $arr_r[0] = 'C';  #错误！！！修改如下：
  $$arr_r[0] = 'C'; #正确，对上述的修正。
  ```



### 哈希、字典（map）

使用
```perl
my %data = ('google', 'google.com', 'runoob', 'runoob.com', 'taobao', 'taobao.com');
my %data = ('google'=>'google.com', 'runoob'=>'runoob.com', 'taobao'=>'taobao.com');
	#注意key,value对，用','或'=>'进行连接，不能用':'。	
print $data{"google"};
	#使用时用$data，定义时用%data



#--------------------------------------------------------------增、删、改、查
$data{'facebooks'} = 'facebooks.com';  #增加元素
delete $data{'taobao'};  #删除元素
if( exists($data{'facebook'} ) ) {
	#是否存在该key
}
@names = keys %data;   #获取Keys集
@urls = values %data;  #获取values集
$size = @urls;  #获取大小

#--------------------------------------------------------------遍历
while((my $k, my $v) = each(%HASH)) {
}
foreach my $k (keys %HASH) {
    my $v = $HASH{$key};
}
```

**Notice**: 定义时注意以下几点

- <font color=red>定义符号为`% + ()`，注意不是`{}`。---那是匿名引用的`$ + {}`</font>。

- 连接符没有`:`，默认只有`,`及`=>`两种。
- 定义时用`%`，使用时用`$`。

**Notice**: 匿名hash引用，注意如下

- 定义为`$ + {}`。

  ```perl
  my $hash_e = {'a'=>'A'};
  ```

- 使用时用`-> 或者 $$`。

  ```perl
  $hash_e->{'c'} = 'C';  #正确
  
  $hash_e{'c'} = 'C';  #错误！！！修改如下：
  $$hash_e{'c'} = 'C'; #正确，对上述的修正。
  ```




## 特殊符号

```perl
\p{xx}
	xx代表一个unicode属性；
```

## 文件操作
Perl 使用一种叫做文件句柄类型的变量来操作文件。文件句柄(file handle)是一个I/O连接的名称。
默认提供的三种文件句柄: STDIN, STDOUT, STDERR。

看段demo
```perl
open(FFILE, ">> /tmp/data");  #追加模式打开文件
print FFILE $txt;    #写入文件
print FFILE "\n-----------\n";  #追加分隔符
close(FFILE);
```

## 编码
perl只有两种字符串，如下：
> **octets**(Ascii), 8位序列，即字节数组；
> **string**, utf8编码的字符串；

可perl怎么区分这两种的呢？
> perl字符串 = 数据 + uft8 flag;
> utf8 flag 跟字符串是否真为 utf8 编码无关；

所以<font color=red>只要字符串内的utf8 flag被置on，那么就按照string来处理</font>，反之就按octets来处理；
> 当字符串 utf8 flag 为 on 但非 utf8 编码时，print 会提示非法 utf8 字符。

utf8 flag处理，如下
```perl
use Encode;
Encode::is_utf8($str);  #查看utf8 flag是否开启；
Encode::_utf8_on($str);  #开启flag，内部函数不建议使用；
Encode::_utf8_off($str); #关闭flag，内部函数不建议使用；
```



### unicode基本原则
对于任何要处理的unicode字符串，要：
> 将其编码转成utf8;
> 开启它的utf8 flag;



如果你的源代码里含有中文, 那么你最好**遵循这个原则**: 

1. 编写代码时使用utf8编码。  ---->**字符串都utf8编码**。

2. 在文件的开头加上“use utf8;”语句。---->**开启utf8 flag**。

这样, 你源代码里的字符串就都会是utf8编码的, 并且utf8 flag也已经打开。



### Encode module

api as follow.  ---`perldoc encode`

```perl
using Encode;
$octets = Encode::encode(ENCODING, $string [, CHECK])
	#把字符串从utf8编码转成指定的编码, 并关闭utf8 flag。

$string = Encode::decode(ENCODING, $octets [, CHECK])
	#把字符串从其他编码转成utf8编码, 并开启utf8 flag, 
	#不过有个例外就是, 如果字符串是仅仅ascii编码或EBCDIC编码的话, 不开启utf8 flag。
```



转码：如果str是gb2312编码的，那怎么弄呢，如下：

```perl
$str = Encode::decode("gb2312", $str);  #转化为utf8编码并开启utf8 flag。

#以下三种场景：str本身就是utf8编码，只是没有开启utf8 flag.（开启utf8编码字符串的uft8 flag）
$str = Encode::decode_utf8($str);
$str = Encode::decode("utf8", $str);
Encode::_utf8_on($str);  #最简洁，但官方不推荐；
```



### 问题集

**问题**：字符串连接
用.连接字符串：utf8 flag为或关系 + 编码保持各自的原样；

> 只要其中任何一个是utf8 flag = on，那么结果是on的；
> 都没有on，那么结果是off的；

所以<font color=red>如果将两个不同编码的字符串连接在一起，那么之后不管怎么转码，都总会出现一段乱码</font>；



**问题：** `Wide character in print at xx.pl line 14.`
当打印utf8字符串时，默认的io处理会报出`Wide character in print`的警告或者错误，现象如下。

```perl
my $str = "呵呵";
if (Encode::is_utf8($str)) {
    print $str, "withUtf8\n";
}
else {
    print $str, "withAscii\n";
}
Encode::_utf8_on($str);
if (Encode::is_utf8($str)) {
    print $str, "withUtf8\n";
}
else {
    print $str, "withAscii\n";
}
Encode::_utf8_off($str);

#-------------output as follow.
# file regex.pl
regex.pl: Perl script, UTF-8 Unicode text executable
# perl regex.pl
呵呵withAscii
Wide character in print at regex.pl line 18.
呵呵withUtf8
```

解决方法如下：

```perl
binmode(STDIN, ':encoding(utf8)');
binmode(STDOUT, ':encoding(utf8)'); #Encode all standard output as UTF-8:
binmode(STDERR, ':encoding(utf8)');
```





## swig

Simplified wrapper and interface generator
对开发人员的一个常见需求是向脚本语言接口公开 C/C++ 代码，这正是 SWIG 的用武之地。

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



## 问题集

### 错误告警

脚本虽然没有编译这一说，但估且叫它`编译错误`吧！

```perl
use strict;  #严格的语法控制
```

<font color=blue>错误告警，只有第1条是准的！如果你要查看后续的错误提示，那么一定要把第1条告警给注释了！！</font>



