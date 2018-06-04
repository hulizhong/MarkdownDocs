[TOC]



Refer: http://www.w3school.com.cn/sql/



## 增
INSERT INTO 表名称 VALUES (值1, 值2,....)

SELECT INTO 语句从一个表中选取数据，然后把数据插入另一个表中。



## 删
DELETE FROM 表名称 WHERE 列名称 = 值


## 改
UPDATE 表名称 SET 列名称 = 新值 WHERE 列名称 = 某值


## 查
SELECT 列名称 FROM 表名称


### 查询约束
DISTINCT  
在表中，可能会包含重复值。这并不成问题，不过，有时您也许希望仅仅列出不同（distinct）的值。


SQL UNION 和 UNION ALL  
UNION 操作符用于合并两个或多个 SELECT 语句的结果集。


SQL JOIN1
有时为了得到完整的结果，我们需要从两个或更多的表中获取结果。我们就需要执行 join。


SQL Alias（别名）2
表的 SQL Alias 语法
列的 SQL Alias 语法


LIKE 操作符 & 通配符
LIKE 操作符用于在 WHERE 子句中搜索列中的指定模式。  
SQL 通配符必须与 LIKE 运算符一起使用。  

问题：Like vs =
> = 搜索的值绝对相等，而非模式匹配。  
> Like 模式匹配。


BETWEEN 操作符
操作符 BETWEEN ... AND 会选取介于两个值之间的数据范围。这些值可以是数值、文本或者日期。


SQL ORDER BY 子句2
SELECT Company, OrderNumber FROM Orders ORDER BY Company, OrderNumber
SELECT Company, OrderNumber FROM Orders ORDER BY Company DESC


SQL GROUP BY 语句1
GROUP BY 语句用于结合合计函数，根据一个或多个列对结果集进行分组。



## 函数
SQL 拥有很多可用于计数和计算的内建函数。  
SELECT function(列) FROM 表


## 视图 view
在 SQL 中，视图是基于 SQL 语句的结果集的可视化的表。  
视图包含行和列，就像一个真实的表。视图中的字段就是来自一个或多个数据库中的真实的表中的字段。我们可以向视图添加 SQL 函数、WHERE 以及 JOIN 语句，我们也可以提交数据，就像这些来自于某个单一的表。
> 视图是一个虚表，即视图所对应的数据不进行实际存储，数据库中只存储视图的定义。



## 索引 index
CREATE INDEX 语句用于在表中创建索引。
在不读取整个表的情况下，索引使数据库应用程序可以更快地查找数据。



## 约束 Constraints
约束用于限制加入表的数据的类型。

AUTO INCREMENT  
我们通常希望在每次插入新记录时，自动地创建主键字段的值。


DEFAULT  
DEFAULT 约束用于向列中插入默认值。


CHECK  
CHECK 约束用于限制列中的值的范围。


FOREIGN KEY  
一个表中的 FOREIGN KEY 指向另一个表中的 PRIMARY KEY。


PRIMARY KEY  
PRIMARY KEY 约束唯一标识数据库表中的每条记录。

