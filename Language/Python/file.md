[TOC]

## ReadMe

## 目录遍历
os.walk
```python
import os
for (dir, dirs, files) in os.walk(""):
	#dir 根路径，是一字符串
	#dirs  路径下的所有目录名，是一列表
	#files 路径下的所有非目录文件名，是一列表
```

os.listdir()
os.path.isdir()
os.path.isfile()
```python
import os
os.listdir()
	#可以列出路径下所有文件和目录名，但是不包括当前目录., 上级目录.. 以及子目录下的文件.
os.path.isfile()
os.path.isdir()
	#判断当前路径是否为文件或目录
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

