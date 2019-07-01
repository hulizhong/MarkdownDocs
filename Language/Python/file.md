[TOC]

## ReadMe

python下file的处理；
python下文件系统的处理；（路径、。。）



问题点：

- [x] readlines(1000)文件不够1000行，但怎么一次性读取不完呢？
- [ ] 。。。





## 文件处理

`read()、readline()、readlines()`均可接受一个变量用以限制每次读取的数据量，但通常不使用。



### read

read()读取整个文件，将文件内容放到一个字符串变量中。
但是：<font color=red>如果文件非常大，尤其是大于内存时，无法使用read()方法</font>。

```python
file = open('readme.txt', 'r')
try:
    text = file.read()
finally:
    file.close()
```



### readline, readlines

readline()方法每次读取一行；返回的是一个字符串对象，保持当前行的内存。
但是：<font color=red>比readlines慢得多</font>。

```python
file = open("sample.txt", 'r')

# Version------------------基础版本
try:
    while True:
    	line = file.readline() #得到的line本身包含一个换行符号, line=line.strip('\n')
    	if not line:
        	break
finally:
    file.close()

#f.readlines(1000)  #有内容不够1000行，但没一次读完，怎么回事？？？---rabin.
#f.readlines()      #这样就能读取完
    
# Version--------------python2.2以后
for line in file:
    pass # do something
```

readlines()一次性读取整个文件，自动将文件内容分析成一个行的列表。

```python
file = open("sample.txt", 'r')
try:
	lines = file.readlines()
finally:
    file.close()
```



### write, writelines

```python
file.write(str)  #参数：str
file.writelines([str])  #参数为：str、str序列
```



## 路径、文件名处理

### API

```python
import os

os.path.basename("/home/rabin/1.log") #1.log
os.path.dirname("/home/rabin/1.log")  #/home/rabin
os.path.split()   #把文件名和路径拆分到元组中
os.path.exists()
os.path.isdir()
os.path.isfile()

os.listdir()
	#可以列出路径下所有文件和目录名，但是不包括当前目录., 上级目录.. 以及子目录下的文件.
os.rename(oldfile, newfile);
```



### 目录遍历

os.walk
```python
import os
for (dir, dirs, files) in os.walk(""):
	#dir 根路径，是一字符串
	#dirs  路径下的所有目录名，是一列表
	#files 路径下的所有非目录文件名，是一列表
```



## csv处理
使用csv标准库进行处理。
```python
import csv

csv_reader = csv.reader(open("fileName.csv"))
for row in csv_reader:
	print row

writer = csv.writer(fw, delimiter='|', quoting=csv.QUOTE_NONNUMERIC)
	#delimiter 字段之间的分隔符号
	#quoting csv的存储模式
		#QUOTE_ALL	为所有的栏增加双引号包围
		#QUOTE_MINIMAL	仅为包含特殊符号的栏增加双引号包围
		#QUOTE_NONNUMERIC	为所有非数字的栏增加双引号包围
		#QUOTE_NONE	在 reader() 函数中，表示不要去掉数据中的双引号包围

reader = csv.reader(fp, skipinitialspace=True)
	#skipinitialspace会删除读取出来的双引号、空格之类的修饰符号；
```
更多细节：https://blog.csdn.net/signjing/article/details/38363599
方言指定了解析或写一个数据文件时使用的所有记号（token），如下表：

|属性 |默认值 |含义 |
|-----|-------|-----|
|delimiter |, |字段分隔符（一个字符）|
|doublequote |True |这个标志控制quotechar实例是否成对|
|escapechar |None |这个字符用来指示一个转义序列|
|lineterminator |\r\n |书写器使用这个字符结束一行|
|quotechar |“ |这个字符串用来包围包含特殊值的字段（一个字符）|
|quoting |QUOTE_MINIMAL |控制前面介绍的引号行为|
|skipinitialspace |False |忽略字段定界符后面的空白符|
|strict |False |当设置为True时，错误的csv输入将弹出异常Error。|

------------
问题集：

问题：处理csv文件错误
```python
_csv.error: line contains NULL byte
```
看看文件的编码，我的是utf-16le，转成utf-8 without bom就行了；

