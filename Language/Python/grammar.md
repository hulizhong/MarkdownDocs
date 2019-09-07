[TOC]

## ReadMe
Python是一种解释型、面向对象、动态数据类型的高级程序设计语言。
- 解释型 = 不需要编译、链接之类的，直接可以在python解释器中执行；
- 面向对象 = 同c++一样是面向对象
- 动态数据类型 = 声明变量时不需要声明变量的类型。（是不是只有定义变量这一说了）



动态语言？

> Python是动态语言，类以及根据类创建的实例可以任意绑定属性以及方法。  
>
> > http://www.cnblogs.com/seirios1993/p/6624157.html



文档查询

```bash
#-----------------法一
$ python 
$ import xx
$ help(xx)
$ help(xx.fun)

#-----------------法二
pydoc xx
```



## 语法

### 简义语法

如下，demo

```python
#单选注释
'''
多行注释！！
'''

var = 100
	#变量定义无需修饰符号。
	#语句没有结尾符号。
print var
	#变量使用无需修饰符号。

def f1():
    print var
    #语句靠`:` + `对齐`进行作用域分隔。

if True:  #条件真为'True'
    pass  #空语句
if False:  #条件假为'False'
    pass
```



### 包、模块
```python
import xx
res = xx.yy()

from xx import yy
res = yy()
```



### 函数

```python
def xx():
    print "1. def."
    print "2. ()"
    print "3. :"
    
xx()   #先定义、后调用（！！不能写在def xx前！！）
```



### 分支测试

**if**

```python
#-----------------------------------------------------单分支
if condition1 and/or condition2:
    pass
else:
    pass

#-----------------------------------------------------多分支
if condition:
    pass
elif condition:
    pass
else:
    pass

#-------单语句
if (var == 100) : print "变量 var 的值为100" 
```

**for**

```python
#-----------------------------------------------------version.1
for itr in sequence:  ##list, string, tuple.
    break #跳出最小封闭for或while循环。
    continue  #跳出本次循环，进入下一次循环。
    pass

#-----------------------------------------------------version.2
for idx in range(len(lst)):
    print lst[idx]
  
#-----------------------------------------------------version.3
for itr in seq:
    pass
else:
    pass    #这里只在for正常结束时执行。（break跳出来的不行）
```

**while**

```python
#------------------no else.
while condition:
    pass

#------------------ with else.
while condition:
    pass
else:
    pass  #这里只在while正常结束时执行。（break跳出来的不行）
```



### 运算符

| 运算符（优先级从上到下） | 描述                                                   |
| ------------------------ | ------------------------------------------------------ |
| **                       | 指数 (最高优先级)                                      |
| ~ + -                    | 按位翻转, 一元加号和减号 (最后两个的方法名为 +@ 和 -@) |
| * / % //                 | 乘，除，取模和取整除                                   |
| + -                      | 加法减法                                               |
| >> <<                    | 右移，左移运算符                                       |
| &                        | 位 'AND'                                               |
| ^ \|                     | 位运算符                                               |
| <= < > >=                | 比较运算符                                             |
| <> == !=                 | 等于运算符                                             |
| = %= /= //= -= += *= **= | 赋值运算符                                             |
| is is not                | 身份运算符                                             |
| in not in                | 成员运算符                                             |
| not and or               | 逻辑运算符                                             |



## 变量

python中的变量与对象  
类型是属于对象的，而不是变量。变量和对象是分离的：对象是内存中储存数据的实体，变量则是指向对象的指针。  
对象分为：可变对象和不可变对象。---id(obj)获取对象的id

不可变对象（如果对象值更改那么就会是一个新的对象，老对象将会迎来垃圾回收机制）
```python
int
float
long
str
tuple  
```

可变对象
```python
list
set
dict
```

--------------
生命周期
在函数体外定义的变量成为全局变量（一般大写），在函数内部定义的变量称为局部变量（一般小写）。
全局变量所有作用域都可读，局部变量只能在本函数可读。
函数在读取变量时，优先读取函数本身自有的局部变量，再去读全局变量。
```python
name = 'Tim' #全局变量
def f1():
	age = 18      #局部变量
	global name   #全局变量默认可读，但要在函数内改变它，则需要在global声明下；
	#全局列表，字典，可修改，但不能重新赋值，如果要重新赋值，则需要在函数内部使用global定义全局变量。
	name = 'Eric'
	print(age, name)   #18 Eric
f1()
print(name)   #Eric
```

-----------
赋值（默认为引用）、拷贝（默认为浅拷贝）
```python
b = a
	#引用，因为此时2个变量的id一致（注意：a是否为可变对象，随着其中任一一个的修改会出现不同的情况）。    
c = copy.copy(a)
	#浅拷贝（指向同一份资源），2个变量的id不一致。       
d = copy.deepcopy(a)
	#深拷贝，2个变量的id不一致。   
```

-----------
函数传参（同于赋值）
隐式传递。  
对于不可变对象作为函数参数，相当于C系语言的值传递；   
对于可变对象作为函数参数，相当于C系语言的引用传递;   
​		

```python
def __lstTest(lstA, lstB):
	''' 
	'''
	la = [1,2,3]
	lb = [3,4,5]
	##----------------method 1: all of these methods cant chane lstA and why.
	lstA = la
	lstA = copy.copy(la)
	lstA = copy.deepcopy(la)
	lstA = list(set(la).intersection(set(lb))) ##交集
	print lstA

	##--------------method 2
	for it in lb: 
		lstB.append(it)
	
def lstTest():
	''' 
	'''
	a = []
	b = []
	__lstTest(a, b)
	print a, b
		
lstTest()
```

### 整数

```python
i = 5
i++ #python不能进行自加加、自减减
i += 1 #这是可以的
i = i + 1

rate = 0.2567
rateStr = '%5.2f' % (rate)  #不够5位则总体占5位，小数点后保留2位。
```



### 字符串
集成的方法如下：  
```python
#-----------------------------------------------------特殊字符
''     #原样字符串，特殊字符无需转义；
""     #特殊字符需转义；
r|R""    #正则字符串
u|U""    #unicode字符串

#-----------------------------------------------------基本操作
length = len(str)

#----------大小写
str.upper()
str.lower()

#-------------查找
str.find(str1)  #return str1's pos, otherwise -1.
str.rfind(str, beg=0 end=len(string))
str.index(str1)   #return str1's pos, otherwise raise exception.

#-----------------------------------------------------截取
str[0:3] #从第1位到第3位。 注意！！！（第1个索引要加1，第2个索引不加1，索引为负不加1）
str[1:]  #从第2位开始到最后
str[-1]  #倒数第1位
str[-3:-1] #倒数第3位到倒数第1位。

str.startswith("sk")`
str = str.lstrip();       #删除左边的空格
str = str.rstrip();
str = str.strip('\t\ ');  #删除两边的\t，空格；

#------------------合并：+、join()、拆分
str = str1 + str2
str = '-';
lst = [a, b, c];
print str.join(lst)

splitLst = str.split("splitStr", 1)  #1为分割的次数，参数名叫啥呢？
	# help(str.split)
	# line.split(sep=",", maxsplit=1) TypeError: split() takes no keyword arguments

#-----------------------------------------------------格式化
# 法一用元组：
strFormat = "I'm format string with %-10s %d %.2f" % (format1, format2, format3)  
# 法二用字典：
print("I'm %(name)s. I'm %(age)d year old" % {'name':'Vamei', 'age':99})

str = u"1234";
str.isdecimal()  #检测unicode字符串是否只包含十进制字符；
str.isdigit()    #检测字符串是否只由数字组成。
```

### 列表list []

列表是有序的。

```python
lst = [1, 3, 2, 4]
	#定义用[].

cmp(lst1, lst2)
len(lst)
max/min(lst)
list(tuple)   #tuple to list
range(from, to, 步阶)   #产生一个递增的list

lst.append(obj)
lst.extend(seq)
lst.count(obj)
lst.index(obj)
	#注意：lst.index()方法会抛出异常，建议用count()。
lst.insert(index, obj)
lst.remove(obj)
lst.pop(obj=lst[-1])
lst.sort([func])
lst.reverse()

#-----------------------------------------------------遍历
for v in t:
    print t
```

**注意**：复制是个坑哦

```python
L1 = L
	#L1为L的别名，用C来说就是指针地址相同，对L1操作即对L操作。函数参数就是这样传递的。   
L1 = L[:]
	#L1为L的克隆，即另一个拷贝。 
```

### 元组tuple ()

同于列表，唯有元素不可改。 

```python
tup = (1, 2, 3, 4, 5)  #定义
```

方法
```python
cmp(tuple1, tuple2)/len()
max/min(t)
tuple(list)	#Seq to tuple

#-----------------------------------------------------遍历
for v in t:
    print t
```

**注意**：只有一个值时，需要注意后缀的逗号

```python
t2 = (3)
print t2  #3
t3 = (3,)
print t3  #(3,)
```

### 字典 {}

<font color=gree>字典默认是无序的</font>（即添加进去的元素，不按你添加的顺序排序）；

```python
dict = {key1 : value1, key2 : value2}
	#默认为无序

import collections  #有序字典
d = collections.OrderedDict()  #有序字典
```

方法

```python
cmp()
len()
str()
type()

keys()/values()/items()
	#Items()返回(key,value)元组的list
copy() 
	#浅拷贝（我的理解是指拷贝了指针的值，但没拷贝指针所指向的资源，亦即两个对象共享某一资源）怎么理解？
	#因为pyton没有指针，所以哪来的浅拷贝？？？
has_key(key)
	#返回True/False
get(key, default=None)
setdefault(key, default=None)
update(dict2)
fromkeys(seq[, val]))
clear()

#-----------------------------------------------------遍历
for k in d.keys():
    print k, d[k]
    pass
for v in d.values():
    print v
    pass
for key,value in a.items():
    print key, value
    pass
```

**注意**：复制是个坑哦

```python
dict1 = dict        #别名  
dict2 = dict.copy()   #克隆，即另一个拷贝。但这也只是个浅拷贝。  
```

### 时间、日期



## 异常

```python
# BaseException 所有异常的基类
# Exception 常规错误的基类

# ----------------format 0.
try:
    pass
except (IndexError,KeyError) as e:
    print(e)

# ---------------------- format 1.
try:
    pass
except Exception as e:
    print(e)
except BaseException as e:
    print(e)
else:
	print("no exception case.")
finnally:
    print("has & no exception case.")
```






## 面向对象
类中要经常用`self` == cpp中的`this`指针
- 类函数的声明；
- 类函数的调用；

没有用public，private等关键词来标志；
> __ 前缀为私有标志
> self.\_\_A() 可以调用 self.\_\_B()吗？    ---可以的

Q.**如何查看一个对象支持哪些属性？？**

> dir(obj)


### 构造函数
子类的构造函数定要手动调用基类的构造函数，否则默认是不会调用基类构造的。
```python
class Base(object):
    def __init__(self):
        print "Base created"
 
class ChildA(Base):
    def __init__(self):
		#老式类的写法；
        Base.__init__(self)
 
class ChildB(Base):
    def __init__(self):
		#新式类的写法，避免了写基类的名字（在多继承下应该很管用）
        super(ChildB, self).__init__()
print ChildA(),ChildB()
```

问题：新式类有什么特点？？
> type（对象）的时候是类名，而非是一个实例。  
> 调用基类的函数可以用super关键字。


### 类变量、类成员变量
如下：  

![这是一张图片](img/grammar-classvalClassmemval.png)

### 继承
语法参照2.1节；



## 多线程
thread 有些缺陷，threading是对thread的补充
```python
t1 = threading.Thread(target=fun1, args=('1111111111',))  #args元组中有个逗号
t1.start()
```


## 注意点
<font color=yellow>**Notice: **</font>for

```python
for .. in ..:   #只有for.. in..， 没有for(;;)这种步阶
```

<font color=yellow>**Notice: **</font>空对象

None和任何其他的数据类型比较永远返回False  
可以将None复制给任何变量，但是你不能创建其他NoneType对象。

```python
type(None)    #<class 'NoneType'>
None == 0     #False
None == ' '   #False
None == False #False
None == None   #True

class Foo(object):
	def __eq__(self, other):
		return True
f = Foo()
if (f is None):  #False，is取决是否是同一对象实例；
if (f == None):  #True，==取决于对象中的__eq__函数；
```

<font color=yellow>**Notice: **</font>if

```python
if x:
	#这是True/False判断  
	#None,  False, 空字符串"", 0, 空列表[], 空字典{}, 空元组()都相当于False

if x is not None:
	#这是单独判断x不是None
if x != None:
	#这是单独判断x不是None
```

<font color=yellow>**Notice: **</font>++， --

没有自增、自减

```python
i += 1   #没有i++
i -= 1   #没有i--
```



## Tips demo

<font color=yellow>**Tips: **</font>sleep

```python
time.sleep(1)  
time.sleep(0.1)
```

<font color=yellow>**Tips: **</font>时间截

```python
Print time.ctime()
```

<font color=yellow>**Tips: **</font>类型转换

```python
int(str)   #str -> int  long(),float()
str(int)   #int -> str
tuple(lst)  #序列lst转元组
list(lst)  #序列lst转列表
json.loads(str)   #str -> list
```

<font color=yellow>**Tips: **</font>随机数

```python
random.randint(1,10)
```



### 神操作

```python
python -m json.tool < /root/replay/thrift-0.9.3/bower.json  #检查json语法哪里有错误！
python -m certifi certifi.where  #检查python用的ca-bundle在哪里！
python -m certifi certifi.oldwhere
```





