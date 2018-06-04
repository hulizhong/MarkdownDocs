[TOC]


## 安装、配置
配置文件 /etc/mysql/*

- /etc/mysql/mysql.conf.d/mysqld.cnf 

	```bash 
	log_error = /var/log/mysql/error.log

	datadir     = /var/lib/mysql

	port        = 3306
	bind-address        = *
	```

## mysql常用命令行

|指令|意义|
|----|----|
|show databases |查看有哪些库
|use databaseName |连接到库
|||
|show tables |查看库的表
|describe tableName |查看表的结构
|||
|show engines |查看存储引擎信息


### mysql控制指令

- 用户操作

	```bash
	# 创建用户，注意@符号两边的字符串引号
	create user 'user'@'host or %' IDENTIFIED BY "password"; 
	# 对用户授权，注意@符号两边的字符串引号
	grant all privileges on opensips.* to opensipss@'%' identified by 'opensipsrw';

	drop user 'username'@'host';
	mysql.user;
	```

- 远程连接
	
	```bash
	某一用户默认只支持本地客户端连接进来；
	
	更改mysql.user表中的Host字段为'%'，而非'localhost'；
	```


## 数据库备份与恢复

### 备份
#### mysqldump备份
- 备份（一个库的表）

	> mysqldump -u userName -p dbName table1 table2 ... > backup.sql     
	> mysqldump -u root --password=debian rabin_db tb_test > backup.sql    
	> mysql -u root -p rabin_db < backup.sql 导入库  


- 备份（多个库）

	> mysqldump -u username -p --databases dbname2 dbname2 > backup.sql


- 备份（全部的库）

	> mysqldump -u -root -p -all-databases > all.sql


#### 拷贝文件备份
- 直接将MySQL中的数据库文件直接复制出来。  

	> 只适合MyISAM引擎的。 --InnoDB存储引擎的表不行。     
	> 还原时，亦最好用同样版本的mysql。  



### 还原

mysql -u root -p [dbname] < backup.sql


