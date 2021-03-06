[TOC]

## ReadMe
perl的regex；

正则是通用的，目前风格有两种：

> POSIX风格正则表达式：Regular Expression
> Perl风格正则表达式：Perl-Compatible Regular Expression，即pcre.
>
> c, c++正则库有哪些呢？
> gnu regex,  regcomp(), regexec(), regerror(), regfree(), ...
> boost regex, basic_regex, match_results, sub_match, reg_match, reg_search, reg_replace, ...
> pcre, pcre_compile, pcre_exec, ..
> pcre++, PE_Options, PE.pattern/error/FullMatch/PartialMatch(), ...



问题：`通配符wildcard`  VS `正则regular`？`模式表达式` VS `正则表达式regular expression`？

> 它俩不是一个东西。 （wildcard更多应用于一些路径匹配方面！regex则应用于字符串处理方面！）
> wildcard 是一种命令行的路迳扩展(path expansion)功能，一般格式如下：
>
> > *: 匹配 0 或多个字元
> > ?: 匹配任意单一字元
> > [list]: 匹配 list 中的任意单一字元(注一)
> > [!list]: 匹配不在 list 中的任意单一字元
> > {string1,string2,...}: 匹配 sring1 或 string2 (或更多)其一字串
>
>  模式表达式是那些包含一个或多个通配符的字。





## perl正则概要

包含三种用法，分别是：匹配、替换、转化。

### 匹配、替换、转化

**匹配、替换、转换**：Perl的正则表达式的三种形式，分别是匹配，替换和转化:

```perl
my $rgx = qr/string/imosx;  #正则字符串、有可能进行正则预编译加速后续的匹配速度；

while ($txt =~ m/$rgx/g) { #匹配
	# =~表示匹配， 
	# !~表示不匹配
}
while ($txt =~ /$rgx/g) { #匹配
}

if ($txt =~ s/pattern/replace/g) { #替换
	#e 替换字符串为表达式
	#g 替换所有匹配字符串
	#o 替换一次
	#e 替换字符串作为表达式，表达式参数为匹配中的字符串；
}

while ($txt =~ tr/$rgx/xx/g) { #转化
	#c 转化所有未指定字符；
	#d 删除所有指定字符；
	#s 把多个相同的输出字符压缩成一个；
}
```

### 结果处理

**结果处理**：匹配结果处理

```perl
if ($txt =~ /$rgx/g) { #匹配
	print "$`";  #前段
	print "$&";  #匹配段
	print "$'";  #后段
}

$line = "a b c d";
while($line =~ /(\S)/g){
	$pos = pos($line);
		#返回最后一次模式匹配的位置；
		#如'a1bbcbd'中匹配'bb'，会返回4；但必须是m//g中g的模式；
	print "before $pos=$1\n";
	
	my $endPos = pos $line;
	my $startPos = $endPos - length($&);
}
```

**正则两种返回值**：正则表达式在求值过程中产生两种情况：

```perl
结果状态： $a =~ m/pattern/ 表示$a中是否有子串pattern出现；
	m//存在返回1，不存在返回空；
	s//存在替换返回1，不存在返回空；
	tr//存在转换返回1，不存在返回0；
反向引用： $a =~ s/(key1)(key2)/$2$1/ 表示调换key1,key2两个单词；
```



### 修饰符

**匹配修饰选项**：修饰符

```perl
i  #忽略模式中的大小写
	# my $re = qr/regex_statement/i;  这样才行。 --------注意！
	# if ($txt =~ m/$re/i) 这样不行！-------------rwhy？？
o  #仅赋值一次，仅替换一次；
m  #多行模式，主要受影响是^$符号
s  #单行模式，"."匹配"\n"（默认不匹配）
x  #忽略模式中的空白，除非它已经被转义；
g  #全局匹配。while循环中匹配一个正则，下次匹配从上一次匹配后的位置开始
cg  #全局匹配失败后，允许再次查找匹配串

l  #告诉Perl用本地语言设置；
a  #告诉Perl用ascii方式；
u  #告诉Perl用unicode方式；
```

## 正则相关知识

正则表达式相关知识。

###元字符
特殊字符
```perl
\   #转义、向后引用、八进制转义
()  #标记子表达式
[]  #中括号表达式
{}  #限定符表达式
```

定位符
```perl
#由于在紧靠换行或者字边界的前面或后面不能有一个以上位置，因此不允许诸如 ^* 之类的表达式。
#所以不能将限定符与定位符一起使用。
^
	#匹配字符串的开始位置；
	#多行模式下也匹配'\r' '\n'之后的位置，即行首。
$
	#匹配字符串的结束位置；
	#多行模式匹配'\r' '\n'之前的位置，即行尾。
\b	#匹配一个单词边界，也就是指单词和空格间的位置。
\B	#匹配非单词边界。
\A  #字符串开头(类似^，但不受处理多行选项的影响)
\Z  #字符串结尾（类似$，但不受多行选项影响）
```

限定符
```perl
?  #匹配前面子表达式0到1次；
	#跟在*, +, ?, {n,}, {n,m}这些限定符之后，匹配模式为非贪婪（尽可能少匹配）。
*  #匹配前面子表达式0到多次；贪婪
+  #匹配前面子表达式1到多次；贪婪
{n}    #匹配n次；
{n,}   #至少匹配n次；贪婪
{n,m}  #匹配n到m次；贪婪
	#n与m之间除逗号之外，不能有空格；
```

非打印字符

```perl
\cx #ASCII控制字符。比如\cC代表Ctrl+C，即ctrl系列的。
	#x 的值必须为 A-Z 或 a-z 之一。否则，将 c 视为一个原义的 'c' 字符。
\f  #匹配一个换页符。等价于 \x0c 和 \cL。
\n  #匹配一个换行符。等价于 \x0a 和 \cJ。
\r  #匹配一个回车符。等价于 \x0d 和 \cM。
\s  #匹配任何空白字符，包括空格、制表符、换页符等等。等价于 [ \f\n\r\t\v]。
\S  #匹配任何非空白字符。等价于 [^ \f\n\r\t\v]。
\t  #匹配一个制表符。等价于 \x09 和 \cI。
\v  #匹配一个垂直制表符。等价于 \x0b 和 \cK。
```

其它
```perl
\a  #报警字符(打印它的效果是电脑嘀一声)
\d  #匹配一个数字字符。等价于 [0-9]。
\D  #匹配一个非数字字符。等价于 [^0-9]。
\e  #Escape
\w  #匹配字母、数字、下划线。等价于'[A-Za-z0-9_]'。
\W  #匹配非字母、数字、下划线。等价于 '[^A-Za-z0-9_]'。
.   #匹配除换行符\n \r之外的任何单个字符；

x|y	#匹配 x 或 y。
[xyz]   #字符集合。匹配所包含的任意一个字符。
[a-z]	#字符范围。匹配指定范围内的任意字符。
[^xyz]  #负值（取反）字符集合。匹配未包含的任意字符。
[^a-z]	#负值（取反）字符范围。匹配任何不在指定范围内的任意字符。
[-\^\\] #匹配-^\这些字符。

\xn  #匹配 n，其中 n 为十六进制转义值。
\num #匹配 num，其中 num 是一个正整数。
	#对所获取的匹配的引用。例如，'(.)\1' 匹配两个连续的相同字符。
\n  #标识一个八进制转义值或一个向后引用。
	#如果 \n 之前至少 n 个获取的子表达式，则 n 为向后引用。否则，如果 n 为八进制数字 (0-7)，则 n 为一个八进制转义值。
\nm #标识一个八进制转义值或一个向后引用。
	#如果 \nm 之前至少有 nm 个获得子表达式，则 nm 为向后引用。
	#如果 \nm 之前至少有 n 个获取，则 n 为一个后跟文字 m 的向后引用。
	#如果前面的条件都不满足，若 n 和 m 均为八进制数字 (0-7)，则 \nm 将匹配八进制转义值 nm。
\nml	
	#如果 n 为八进制数字 (0-3)，且 m 和 l 均为八进制数字 (0-7)，则匹配八进制转义值 nml。
\0nn	
	#ASCII代码中八进制代码为nn的字符
\xnn	
	#ASCII代码中十六进制代码为nn的字符
\unnnn
	#Unicode代码中4个十六进制代码为nnnn的字符
		# Notice.------rwhy. 验证不通过。
		# \unnnn, \u{nnnn}, \xnnnn均失败！！！
		# 只能用\x{nnnn}来表示16进行的unicode码！！！
	#\u4e00-\u9fa5 汉字编码范围；
```

### 运算符优先级
优先级如下
```perl
转义符：\
圆括号、方括号：() []
限定符：*, +, ?, {n}, {n,}, {n,m}
或：|
```

### 多行、单行模式

默认是单行模式，此时

```bash
$str =~ /pattern/s;
# \. 能匹配所有字符。（默认除回车\r，换行\n之外的所有字符）
```

开启多行模式时，如下：

```bash
$str =~ /pattern/m;
# ^$ 在行头、行尾的基础上，可匹配字符串的开头、结尾。
```

<font color=red>是不是只能同时使用多行模式和单行模式中的一种</font>？
答案是：不是。这两个选项之间没有任何关系，除了它们的名字比较相似（以至于让人感到疑惑）以外。

### 预查

正向（<font color=red>修补匹配项的结尾</font>）、反向（<font color=red>修补匹配项的开始</font>）肯定、否定预查：----editpolus的正则不支持反向预查？

- <font color=red>反向预查，在有些语言、或库中只支持固定长度的</font>，如在libpcre中，`(?<![0-9]+)[0-9]{15}(?![0-9]+)`，需要变更成`(?<![0-9]{1})[0-9]{15}(?![0-9]+)`才能work；

```bash
(?=pattern)
	#正向肯定预查（look ahead positive assert）。首先，要匹配的文本必须满足此子模式前面的表达式；其次，此子模式不参与匹配。例如，"Windows(?=xp|7)"能匹配"Windows7"中的"Windows"，但不能匹配"Windows10"中的"Windows"。
	#这是一个非获取匹配。
	#预查不消耗字符，即在一个匹配发生后，在最后一次匹配之后立即开始下一次匹配的搜索，而不是从包含预查的字符之后开始。
(?!pattern)	
	#正向否定预查(negative assert)。
(?<=pattern)
	#反向(look behind)肯定预查，与正向肯定预查类似，只是方向相反。
	#例如，"(?<=xp|7)Windows"能匹配"7"中的"Windows"，但不能匹配"10Windows"中的"Windows"。
	#perl 5.30之前，只适用于固定宽度的向后查找；
(?<!pattern)   #---------其中的反斜杠为转义用，不在表达式范围内；
	#反向否定预查，与正向否定预查类似，只是方向相反。
	#perl 5.30之前，只适用于固定宽度的向后查找；
```

### 分组

选择（分组）

```perl
(pattern)
	#捕获匹配，匹配 pattern 并获取这一匹配。
	#所捕获的匹配从左到右存储于一个顺序缓存区，编号从1-99，可用\n来访问各缓冲区；
	#法一：\b(\w+)\b\s+\1\b可以用来匹配重复的单词，像go go, 或者kitty kitty。
		#这个也叫反向引用；
	#法二：\b(?<CaptureName>\w+)\b\s+\k<CaptureName>\b
		#在正则中可以用?<name>进行命名、用\k<name>来引用，其中的的<>是必须的！
		#在代码中用$+{name}来获取结果！
	#法三：\b(?'CaptureName'\w+)\b\s+\k'CaptureName'\b
		#在正则中可以用?'name'进行命名、用\k'name'或\g{name}来引用，其中的''{}亦是必须的！
		#在代码中用$+{name}来获取结果！
(?:pattern)
	#非捕获匹配，匹配 pattern 但不获取匹配结果，不进行存储供以后使用。
(?#comment)
	#注释
```




## 匹配效率
具有更加确定性的正则会相对快点
```perl
^\s*[\w]{1-64}\(\)\s*{
^\s*function\s?[\w]{1-64}\(\)\s*{
	第2个正则快；（相对第1个更具有确定性）
```

不保存结果的获取
```perl
(?:)
	通常括号中匹配到的正则，是需要分配空间进行保存的；但?:就不会保存结果了；
```

贪婪（perl正则默认行为）、懒惰
```perl
贪婪模式的量词如下：
	{m,n}
	{m,}
	?  0到1
	*  任意个，0到多
	+  1到多
如果在贪婪量词后面加?，那么就会变成懒惰匹配；

aa.*?cc 匹配字符串aabbccbbcc
	贪婪（尽可能长）会匹配出aabbccbbcc
	懒惰（尽可能短）会匹配出aabbcc
ab.*?cc 匹配字符串aabbcc
	在贪婪量词后加了字符'?'
```

## 问题集
### unicode串匹配

非ascii码匹配，如中文匹配，<font color=red>待匹配字符串必须要用utf8字符串的模式</font>来进行识别匹配！！

```perl
#!/usr/bin/perl -w
use Encode;
my $data = "name on the c–ommand line, the bytec–ode for the script.";
Encode::_utf8_on($data);  #点1，用uft8方式解释字符串。
my $rgx = qr/(c\x{2013}\w*\b)/;
my $rgx2 = qr/(c.*?\b)/;
while ($data =~ /$rgx/ug) { #点2，貌似这个没有效果！！ --rwhy
    print "matched <$&>\n";
    #print "matched <$1>\n";
}
print "------\n";
while ($data =~ /$rgx2/ug) {
    print "matched <$&>\n";
    #print "matched <$1>\n";
}

#------------output as follow.
matched <c–ommand>
matched <c–ode>
------
matched <c>
matched <c>
matched <cript>
```

**注1**：`\x{4e00}-\x{9fa5}` 这是汉字的范围。
​        其中`\x`是16进制的意思，`4e00`是某汉字的unicode码。



### about regex

**pos()**

正则匹配中，`pos()`必须是`//g`匹配，才会有效！！



**//g 全局性** ---<font color=#0000ff>针对同一正则而言，即下次该正则从当前匹配位置开始！</font>

```perl
print "------------------------ 1\n";
my $txt = "Wang hu lizhong hah";
my $rega = qr/hu/;
my $regb = qr/lizhong/;

if ($txt =~ m/$rega/g) {
    print "a1 found =>", $&, "\n";  #找到'hu'，并记录该正则的该位置。
}
if ($txt =~ m/$regb/g) {
    print "b1 found =>", $&, "\n";  #找到'lizhong'。
}
if ($txt =~ m/$regb/g) {
    print "b2 found =>", $&, "\n";  #未输出！！！！
}
```





**忽略大小写**

`my $re = qr/regex_statement/i;`  这样才行。 --------注意！
`if ($txt =~ m/$re/i)` 这样不行！-------------rwhy？？



**T. 向后（正、负）断言/预查**

5.30之前的perl，只支持固定长度的pattern；

```perl
my $txt = "18501960037welcome to runoob site.";
my $reg = qr/(?<=\D|^)(1[34578]\d{9})(?=\D|$)/i;
while ($txt =~ /$reg/g){
   print "matched $& \n";
}

# 5.30
Variable length lookbehind is experimental in regex; marked by <-- HERE in m/(?<=\D|^)(1[34578]\d{9})(?=\D|$) <-- HERE / at prog.pl line 2.

# 5.18
Variable length lookbehind not implemented in regex m/(?<=\D|^)(1[34578]\d{9})(?=\D|$)/ at prog.pl line 2.
```

但是在5.10之后提供了一种特殊的机制`\K`；

```perl
my $reg = qr/(\K\D|^)(1[34578]\d{9})(?=\D|$)/;
```



