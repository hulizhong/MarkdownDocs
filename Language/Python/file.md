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
