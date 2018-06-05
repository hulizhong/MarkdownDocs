[TOC]

## 语法
### 符号
- 符号
	- + 续行符
- 算术运算
	- +, -, *, /, ^乘方, %*%矩阵乘法, %%求余, %/%整数除法
- 赋值
	- =, <-
- 求助符号
	- ?, help()
	



### 输入、输出

- 输入
	- read.table(文件名, header=FALSE第1行为变量名默认为否即自动添加列名) 读取表格形式的文件
		- read.csv/csv2()
		- read.delim/delim2()
		- read.fwf()
		- readLines(filename) 每行有不同的结构
	- scan() 可直接读取纯文本文件数据，从屏幕上读取数据
	
- 输出
	- 直接输入变量名即可
	- print(x)
	- digits指定数字的有效位数，options(digits=3)
	- write.table()系列
		- write.csv/csv2()
		- write.matrix()
	- cat() 可把多个参数连接起来再输出，也可输出到文件中；


sink("E://output.txt") 该语句后面的输出从屏幕重定向到文件中；  
sink() 输出恢复到屏幕；  


### 分支结构
- if
	
	```bash
	if(condition) command else command2
	
	switch()
	```

- for 

	```bash
	for(name in expr1) expr2
	
	while(condition) expr
	
	repeat expr + break
	```
	
### 函数
```bash
定义：
	变量名=function(参数列表) 函数体

调用：
	变量名(参数值)
```

- 函数参数可以有缺省值，直接用=
- 函数可以没有return，此时最后一个值为return的值
	- 为了输出多个值，最好使用list

	
### 注意点

- r区分大小写；

- windows中路径写法： 'c:/my.csv' or 'c:\\my.csv'



## 数据结构
### 向量
其中的元素类型必须相同，包括数值、逻辑、复值、字符型

- 数值向量
	- seq() 向量（序列）有简单的规律
	- rep()  向量（序列）有复杂的规律
	- c() 向量（序列）没有什么规律

	- 计算
		- abs(x) 绝对值
		- sort(x, decreasing=FALSE) 返回按x的元素从小到大的排序的结果向量
		- order(x) 使得x从小到大排列的元素下标向量 sort(x) == x[order(x)]

- 逻辑向量
	元素为逻辑值TRUE, FALSE
- 字符型向量
- 复数向量
- 向量下标运算
	- 正整数下标  取值，取范围
	- 负整数下标  删除
	- 逻辑下标  取范围，如x[x<10]


### 矩阵

- 矩阵函数
	- A=matrix(data=数据向量, nrow=行数, ncol=列数, byrow=FALSE是否按行写入数据，默认为列, dimnames=NULL)
		- c(A) 显示A的所有向量，按列；
	- cbind()把向量横向拼成一个矩阵；
	- rbind()把向量纵向拼成一个大矩阵；
	- diag() 对角和单位矩阵

- 访问矩阵
	- A[2, 3] 访问矩阵的(2, 3)元素
	- A[i,] 第i行
	- A[,j] 第j列
	- A[]=0 改变元素值为0
	- rownames(A)=c('a', 'b') 行标重新命名
	- colnames(A)=paste("X", 1:4, sep="")

- 矩阵运算
	- +, -, *, /, ^	
	- apply(X矩阵, MARGIN1按行计算2按列, FUN计算函数, ...)  对某行、列进行某种计算
	- det(A) 行列式
	- solve(A) 求逆
	- eigen(A) 特征值和特征向量

### 因子
- factor() 把一个向量编码为一个因子
	- is.factor() 检验对象是否是因子
	- as.factor() 把向量转化为因子
	- levels() 可以得到因子的水平
	- table() 对因子向量统计各类数据的频数

- tapply(x对象一般为向量, index与x有同样长度的因子，fun是要计算的函数) 

- gl() 方便的产生因子
	- gl(n水平数, k重复的次数, length=n*k结果的长度, labels=1:n为n维向量，表示因子水平, ordered=FALSE是否为有序因子)
	
### 列表list、数据框data.frame
#### list
- 构造
	- list(key=value, ...)
	
- 获取
	- lst[['itemName']] or lst$itemname
	- lst[[idx]] 用[[]]获取元素，且不同于向量不能取range
		- lst[idx], lst[idx,idx2] 返回一lst

- 修改
	- 直接赋值就行了
	- NA代表空元素
	- list.abc=c(list.a, list.b, list.c) 连接

- edit()


#### 数据框
数据框：通常是矩阵形式的数据，但矩阵各列可以是不同类型。数据框每列是一个变量，每行是一个观测。

- 生成
	- data.frame() 类似于list()
	- as.data.frame(lst) 从list转
		- is.data.frame(d) 判断
	
- 引用
	- 元素  同于矩阵d[1:2, 2:3]
	- 变量  同于列表 [[]], $
	- rownames 定义行名字, names()获取名字
	
- attach(d) 把变量调入内存, 因为d[[]]是不方便的
	- detach(d) 取消连接

- edit()			

## 图形

### plot(x, y, ...)
常用的画图函数，亦可做x, y的散点图

- main 图的主题
- xlab x轴的标题
- ylab y轴的标题
- type 图的类型
	- l 线
	- p 绘制单独的点（默认值）
    - b 绘制由线连接的点（both）
    - o 将点绘在线上
	- s 阶梯式图

### stem() 茎叶图

### boxplot 盒形图 
可看出数据的大体分布，因为有中位数，1/4位数，3/4位数，最小值，最大值；

### hist() 直方图


### 其它

- 画图函数
	- qqnorm(), qqline() 正态qq图及相应的直线
	- points() 在已有的图形上加点
	- lines() 在已有的图上加直线
	- text() 在图上作标记
	- legend() 在指定的位置给出一个盒子形的解释
	- locator() 在鼠标点击的地方做标记
	- identify() 在指定的点做标记
	- 在图上加直线
		- abline()
			- h=1 y轴值
			- v=1 x轴值
			- lwd=4 线宽
			- col="blue/red" 颜色，可以用 colors() 来查看支持哪些类型。
		- polygon()
	
	
- 画图相关
	- dev.new()
	- dev.off()
	- 保存图片
		- postscript("xx.ps")
		- jpeg(), bmp, png, tiff



## 技巧、常用
### 技巧
- 清屏 ctrl+l 

### 常用

绝对值：abs  sqrt

简单统计量：sum, mean, var, sd, min, max, median中位数, range, sort, order, rank,


source("E://test.R")  执行一个r文件；



- 帮助
	- ?查询的函数  函数帮助
	- q() 退出R


- 包
	- install.packages("包名") 安装包
	- library(包名) 载入包
 
- 空间
	- ls()/objects() 列出当前空间的所有对象；
	- str(x) 给出对象x的一些信息。 不同于str("x")


