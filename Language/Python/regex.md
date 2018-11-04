[TOC]

## ReadMe
python regex. 1.5版本以后引入，它提供 Perl 风格的正则表达式模式。

```python
import re
print re.search("\\W+\"rabin\"", 'haha? "rabin"').group()
	#""需要转义pattern中的特殊字符串；
print re.search(r"\W+\"rabin\"", 'haha? "rabin"').group()
	#r""不需要转义pattern中的特殊字符串，但"除外；
print re.search(r'\W+"rabin"', 'haha? "rabin"').group()
	#r''不需要转义pattern中的特殊字符串；
```


## 匹配查找
分为以下几种；

- 匹配：只一个
  - re.match 从字符串的起始位置进行匹配，并返回None或匹配对象。
  - re.search 匹配整个字符串，并返回None或匹配对象
- 匹配：全部
  - re.findall 匹配整个字符串，并并返回一个列表，如果没有找到匹配的，则返回空列表。
  - re.finditer 同于findall，但返回迭代器。
- 匹配：分割
  - re.split 按照能够匹配的子串（即正则）将字符串分割后返回列表。
- 匹配：替换
  - re.sub 替换字符串中的匹配项。
- none.





### match
re.match(pattern, string, flags=0)
```bash
pattern  #正则 r'regex_pattern'
string   #待匹配的内容；
flags    #匹配的修饰符，如大小写、单行模式之类；
return
	#成功则返回匹配对象、失败则返回None（用group/groups处理结果）；
	#group(num)
		#group(0) 为匹配中的整个子串  == group()
		#group(1) 为第1个分组匹配成功的子串
	#groups() 全部匹配项，为一元组
```

demo
```python
import re

line = "Cats are smarter than dogs"
matchObj = re.match( r'(.*) are (.*?) .*', line, re.M|re.I)
 
if matchObj:
	print matchObj.groups()  #('Cats', 'smarter')
	print matchObj.group()  #Cats are smarter than dogs
	print matchObj.group(1) #Cats
	print matchObj.group(2) #smarter
else:
    print "No match!!"
```

### search
re.search 扫描整个字符串并返回第一个成功的匹配。
re.search(pattern, string, flags=0)
```bash
return
	#成功则返回一个匹配对象，否则返回None。
```

demo
```python
import re

line = "Cats are smarter than dogs";
searchObj = re.search( r'(.*) are (.*?) .*', line, re.M|re.I)
 
if searchObj:
	print searchObj.group()  #Cats are smarter than dogs
	print searchObj.group(0) #Cats are smarter than dogs
		#group(0) == group()
	print searchObj.group(1) #Cats
	print searchObj.group(2)
else:
	print "Nothing found!!"
```


### findall
match和search是匹配一次，而findall匹配所有。

re.findall(string[, pos[, endpos]])
```bash
string
	#待匹配的字符串。
pos
	#可选参数，指定字符串的起始位置，默认为 0。
endpos
	#可选参数，指定字符串的结束位置，默认为字符串的长度。
```

demo
```python
pattern = re.compile(r'\d+')
result1 = pattern.findall('runoob 123 google 456')
print(result1)  #['123', '456']
```

### finditer
和findall类似，只不过返回迭代器。
re.finditer(pattern, string, flags=0)

demo
```python
it = re.finditer(r"\d+","12a32bc43jf3") 
for match in it: 
    print (match.group())
```

### split
re.split(pattern, string[, maxsplit=0, flags=0])
```bash
maxsplit
	#分隔次数，默认为0不限制；
```

demo
```python
re.split('\W+', ' runoob, runoob, runoob.', 1)
	#['', 'runoob, runoob, runoob.']
 
re.split('a*', 'hello world')
	#['hello world']
	#对于一个找不到匹配的字符串而言，split 不会对其作出分割
```

### sub

re.sub(pattern, repl, string, count=0, flags=0)
```bash
pattern
	#正则
repl
	#用于替换的值，可以是一个函数；
string
	#被匹配替换的字符串；
count
	#替换的次数，0为替换所有；
```



## 关于匹配

### 正则编译

编译正则表达式，生成一个正则表达式（ Pattern ）对象，该对象拥有一系列方法用于正则表达式匹配和替换。

re.compile(pattern[, flags])
```bash
import re

pattern = re.compile(r'\d+')               #用于匹配至少一个数字
m = pattern.match('one12twothree34four')   #查找头部，没有匹配
print m  #None

m = pattern.match('one12twothree34four', 3, 10) #从'1'的位置开始匹配，正好匹配
print m     #<_sre.SRE_Match object at 0x10a42aac0>
m.group(0)   #'12'，打印整个匹配子串
m.span(0)    #(3, 5) 第n个匹配项的头、尾
m.start(0)   #3
m.end(0)     #5
```

### 匹配选项

正则模式模式、正则修饰符

```bash
re.I
	#使匹配对大小写不敏感
re.M
	#多行匹配，影响 ^ 和 $
re.S
	#使 . 匹配包括换行在内的所有字符

re.L
	#做本地化识别（locale-aware）匹配
	#这个标志影响 \w, \W, \b, \B, \d, \D, \s, \S.
re.U
	#根据Unicode字符集解析字符。
	#这个标志影响 \w, \W, \b, \B, \d, \D, \s, \S.

re.X
	#该标志通过给予你更灵活的格式以便你将正则表达式写得更易于理解。
```

### 匹配返回

match(), search()的返回结果有如下特征：

```python
m = pattern.match(..)

m.group(idx=0,)  #匹配的字符串，从1开始（默认为0即全部），可以一次输入多个组号。
m.groups()  #返回元组，包含所有匹配中的字符串。
m.groupdict()  #将命名分组匹配的子串，按照字典返回。

m.start(idx=0) #第idx个分组匹配的子串在整个字符串中的起始位置。
m.end(idx=0)   #第idx个分组匹配的子串在整个字符串中的结束位置。
m.span(idx=0)  #返回 (start(idx), end(idx))。   
```





## 关于正则

### 正则

正则表达式
```bash
^
	#匹配字符串的开头
$
	#匹配字符串的末尾。
.
	#匹配任意字符，除了换行符，当re.DOTALL标记被指定时，则可以匹配包括换行符的任意字符。
[...]
	#用来表示一组字符,单独列出：[amk] 匹配 'a'，'m'或'k'
[^...]
	#不在[]中的字符：[^abc] 匹配除了a,b,c之外的字符。
re*	
	#匹配0个或多个的表达式。
re+
	#匹配1个或多个的表达式。
re?
	#匹配0个或1个由前面的正则表达式定义的片段，非贪婪方式
re{n}
	#精确匹配 n 个前面表达式。例如， o{2} 不能匹配 "Bob" 中的 "o"，但是能匹配 "food" 中的两个 o。
re{n,}
	#匹配n个及以上前面表达式。
	#o{1,} 等价于 "o+"。
	#o{0,} 则等价于 "o*"。
re{ n, m}
	#匹配 n 到 m 次由前面的正则表达式定义的片段，贪婪方式
a| b
	#匹配a或b
(re)
	#匹配括号内的表达式，也表示一个组
(?: re)
	#类似 (...), 但是不表示一个组
(?= re)
	#前向肯定界定符。
	#如果所含正则表达式，以 ... 表示，在当前位置成功匹配时成功，否则失败。但一旦所含表达式已经尝试，匹配引擎根本没有提高；模式的剩余部分还要尝试界定符的右边。
(?! re)
	#前向否定界定符。与肯定界定符相反；当所含表达式不能在字符串当前位置匹配时成功
\w
	#匹配字母数字及下划线
\W
	#匹配非字母数字及下划线
\s
	#匹配任意空白字符，等价于 [\t\n\r\f].
\S
	#匹配任意非空字符
\d
	#匹配任意数字，等价于 [0-9].
\D
	#匹配任意非数字
\G
	#匹配最后匹配完成的位置。
\b
	#匹配一个单词边界，也就是指单词和空格间的位置。
	#例如， 'er\b' 可以匹配"never" 中的 'er'，但不能匹配 "verb" 中的 'er'。
\B
	#匹配非单词边界。'er\B' 能匹配 "verb" 中的 'er'，但不能匹配 "never" 中的 'er'。
\n, \t
	#匹配一个换行符。匹配一个制表符。等
\1...\9
	#匹配第n个分组的内容。
\10
	#匹配第n个分组的内容，如果它经匹配。否则指的是八进制字符码的表达式。


(?imx)
	#正则表达式包含三种可选标志：i, m, 或 x 。只影响括号中的区域。
(?-imx)
	#正则表达式关闭 i, m, 或 x 可选标志。只影响括号中的区域。
(?imx: re)
	#在括号中使用i, m, 或 x 可选标志
(?-imx: re)
	#在括号中不使用i, m, 或 x 可选标志
(?> re)
	#匹配的独立模式，省去回溯。
(?#...)
	#注释.
\A
	#匹配字符串开始
\Z
	#匹配字符串结束，如果是存在换行，只匹配到换行前的结束字符串。
\z
	#匹配字符串结束
```

### 分组匹配
(?P...)
```python
s = '1102231990xxxxxxxx'
res = re.search('(?P<province>\d{3})(?P<city>\d{3})(?P<born_year>\d{3})', s)
print(res.groupdict()) #{'province': '110', 'city': '223', 'born_year': '199'}
```

