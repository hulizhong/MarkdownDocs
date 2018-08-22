[TOC]



## ReadMe

讨论apache相关；



## apache2 feature

### rewrite

1. load rewrite.mod

2. httpd.conf

   > Options FollowSymLinks  
   > AllowOverride ALL  

3. .htaccess in webdata dir.

   > RewriteEngine On    
   > RewriteCond ..  
   > RewriteRule ..  
   >
   > > 打开webdata dir目录的FollowSymLinks属性且在.htaccess里要声明 RewriteEngine on  

域名跳转、URL跳转；
http://blog.csdn.net/chamtianjiao/article/details/6268746  





## LinuxApacheMysqlPhp

### install process

```bash
# 如果是阿里云的服务器，那么配置阿里云的源，这样会快点；
apt-get install apache2

apt-get install php5
#如果是php7，那么还要安装个libapache2-mod-php7.0 
apt-get install php5-gd

apt-get install mysql-server
apt-get install php5-mysql
```



### install issue

#### apache

client访问网页报错：403 Forbidden: client denied by server configuration
查看apache error log如下：

```sh
[authz_core:error] [pid 779] [client 127.0.0.1:36184] AH01630: client denied by server configuration: /var/webdata/hello.php
[authz_core:error] [pid 777] [client 106.121.15.124:40040] AH01630: client denied by server configuration: /var/webdata/hello.php
```

apache2.2到2.4网站访问控制发生了改变。
apache2.2

> Order, Allow, Deny

apache2.4

> Require

```sh
Require all granted #允许所有
Require all denied #拒绝所有
Require env env-var [env-var] ... #允许匹配环境变量中任意一个
Require method http-method [http-method] ... #允许特定的HTTP方法（GET/POST/HEAD/OPTIONS）
Require expr expression #允许，表达式为true
Require user userid [ userid ] ... #允许特定用户
Require group group-name [group-name] ... #允许特定用户组
Require valid-user # #允许，有效用户
Require [not] ip 192.100 192.168.100 192.168.100.5 #允许特定IP或IP段，多个IP或IP段间使用空格分隔

# 关键字前加not表示取反。
```



-------

client访问网页报错：Permission denied: because search permissions are missing on a component of the path
站点目录的文件权限应该是755

> chmod a+x * -R    
> chmod -R 755 www/*



#### php

error>> PHP Fatal error:  Call to undefined function mysql_connect()
php缺少对mysql的支持

> apt-get install php5-mysql 



#### mysql

error>> allow mysql remote connection
打开Mysql的远程连接功能。

```sh
mysql -u root -p 
mysql>use mysql; 
mysql>update user set host = '%' where user = 'root'; 
mysql>select host, user from user;
```



### php5 config

其配置文件在/etc/php5/*，如下图：  
![这是一张图片](img/lamp-php5_configFile.png)

其中的apache2, cli是php提供的两种不同的(SAPI)接口类型。
Server Application Programming Interface（服务器应用编程接口）。PHP通过SAPI提供了一组接口，供应用和PHP内核之间进行数据交互。 

> PHP提供很多种形式的接口，包括apache、apache2filter、apache2handler、caudium、cgi 、cgi-fcgi、cli、...

