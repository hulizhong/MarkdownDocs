[TOC]

| lib name |  lib des |
|----------|----------|
|bisect |二分算法：查找、插入|
|hashlib |hash库|
|struct |二进制字节流的处理|
可以用dir(obj)来查看对像内所有属于及方法。  


## psycopg2
```python
import psycopg2
conn = psycopg2.connect(“dbname=test user=postgres”)
cur = conn.cursor()
cur.execute(“insert into/update/drop/delete from/create table ...”)
##查询
cur.execute(“SELECT * FROM test;”)
cur.fetchone/fetchall()
##事务
conn.commit()
conn.close()

conn.set_session(readonly=True, autocommit=True)
```




## ctypes
Ctypes提供了与c语言兼容的数据类型，使得python可以调用c库；

使用总结
- 根据c的头文件重新定义一套python头文件（只有数据结构）；
	- 其中数据结构重定义时要满足；  
		- python中要实现c语言中的结构，需要用到类。  
		- 重新定义时顺序要一致。
		- 数组类型用：char[6] == c_char * 6
- 在python中导入c库，得到dll对象；
- 利用dll对象来调用dll中的api；


## markdoc   
是一个markdown的wiki系统（类似confluence）   
> 什么是wiki系统？  ---一种多人协作的写作工具。  

问题：markdoc serve 报 cherrypy no module named wsgiserver 错误。
> 换了个低版本的CherryPy 3.8.2，markdoc安装依赖时默认安装了最高版本10.2.2。  


用法
> markdoc init -> build -> serve





## 邮件
收发邮件，涉及到两个库
- smtplib 收发邮件
- email 按格式生成、解析邮件内容

### smtplib
参考： http://www.cnblogs.com/xiaowuyi/archive/2012/03/17/2404015.html

发送邮件的步骤
- 连接发送服务器
- 登录（现在的邮件服务器都需要登录方可进行邮件的发送了）
- 发送

#### 网易邮箱注意点
Error: 无法发送邮件, cause:  (550, 'User has no permission')
> 未开启客户端授权码功能（可用于第三方软件进行邮件的发送）

Error: 无法发送邮件, cause:  (535, 'Error: authentication failed')
> 账号密码错



### email
MIME


## xml
> /usr/lib/python2.7/xml  为python自带的库，分为dom, etree, sax
>> dom 它是W3C DOM API的实现，若需要处理DOM API则该模块很适合。  
>> sax 它是SAX API的实现，这个模块牺牲了便捷性来换取速度和内存占用，SAX是一个基于事件的API，这就意味着它可以“在空中”处理庞大数量的的文档，不用完全加载进内存。  
>> etree（简称 ET），它提供了轻量级的Python式的API，相对于DOM来说ET 快了很多，而且有很多令人愉悦的API可以使用，相对于SAX来说ET的ET.iterparse也提供了 “在空中” 的处理方式，没有必要加载整个文档到内存，ET的性能的平均值和SAX差不多，但是API的效率更高一点而且使用起来很方便。


### dom

```bash
dom = minidom.parse(xml_in_path)
root = dom.documentElement

# 注意返回的是root下所有KeyName为key的node数组。
# 但应该不适用因为分不清了，所以还是应该一级一级的找下去，最后定位到想要的key。
node = root.getElementsByTagName("keyName")[0]

# node.firstChild.nodeValue = newValue
node.childNodes[0].nodeValue = newValue

try:
	with open(xml_out_path, 'w') as ofs:
		dom.writexml(ofs, encoding='utf-8')
except Exception as err:
	print("Error info: {0}".format(err))
```

### etree

```bash
tree = ElementTree()
et = tree.parse(xml_in_path)

# curr节点为根节点的子节点，而非根节点名称。（大优点：可多级查询）
node = et.find("curr/child1/child2")

node.text = newValue

# 输出是会把xml_in_path中的注释性语句删除（这是个大缺点）。
tree.write(xml_out_path, , encoding='utf-8') 
```

### 额外知识

语法分析器

https://www.cnblogs.com/hongfei/p/python-xml-sax.html


## pandas

```python
from pandas import Series, DataFrame
import pandas as pd 
```

pandas建造在NumPy之上，它使得以NumPy为中心的应用很容易使用。

pandas也是围绕着 Series 和 DataFrame 两个核心数据结构展开的。
> Series 和 DataFrame 分别对应于一维的序列和二维的表结构。


### 数据类型
- Series
	- 一维数组对象，包含索引、及值.
		- 是一个定长的，有序的字典，因为它把索引和值映射起来了。
	- 排序
		- series.order() 默认为升序
		- series.sort_values()
	- 统计方法
		- quantile(0.1) 样本分位数，0-1
		- sum() 求和
		- mean() 均值
		- median() 中位数
		- min/max() 最小值、大值
		
	
	```python
	>>> import pandas as pd
	>>> ss = pd.Series([4, 7, -5, 3])
	>>> print ss
	0    4
	1    7
	2   -5
	3    3
	
	>>> print ss.index
	array([0, 1, 2, 3])
	>>> print ss.values
	[ 4  7 -5  3]
	
	# 构造：强制带上index
	>>> s2 = pd.Series([4, 7, -5, 3], index=['d', 'b', 'a', 'c'])       
	>>> print s2
	d    4
	b    7
	a   -5
	c    3
	
	# 迭代
	for index, value in series.iteritems():
		print index, value
	```

- DataFrame
	- 表示一个表格，类似电子表格的数据结构，每列都是经过排序的。
		- 具有行、列索引；
	- 获取列
		- df.year
		- df["year"]
	- 获取行
		- df.ix("row_index")

	- 行数
		- len(df)
	- 排序
		- df.sort(columns=None要排序的列名, ascending=True默认升序) 排序时，字段不能为一个object，df中有可能是一些列组成的一个object。
			- dataRate = df.sort(columns='rate', ascending=False)  低版本
			- dataRate = df.sort_values(by='rate', ascending=False) 高版本
		- df[''].order() 高版本才支持。
			- closeOrder = df['close'].order()  低版本
			- closeOrder = df['close'].sort_values()  高版本
	- 条件选择
		- df[df.A > 2]
		- df.loc[df.A > 2]
	
	```python
	#迭代
	for index, row in df.iterrows():
	    print row
		
	#构造
	data = pd.DataFrame(columns=['name','diff','rate'])
	for ...:
		se = pd.Series([file, diff, rate], index=['name','diff','rate'])  #创建series时指明index，否则插不到df中
		data = data.append(se, ignore_index=True)  #用默认的index
	print data
	```

## tushare

