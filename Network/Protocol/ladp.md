[TOC]

## ReadMe
用户集中管理包含用户信息集中管理和用户集中授权，是实现“一站式登录”的基础。

“一站式登录”，即用户通过一次性鉴权登录后，即可获得需访问所有应用系统的授权。

> 参见：华为统一网络管理 http://developer.huawei.com/cn/ict/Products/NetworkManagement/eSight/authentication/SDK#section-3-2



## User Management In Company

一般内网渗透测试就是拿到特定服务器（域管理员的机器，或者域控制器，或者邮件服务器。。。OA系统）



### Work Group

管理模式）工作组实现的是分散的管理模式。
账号管理）每一台计算机都是独自自主的，用户账户和权限信息保存在本机中。
共享信息）同时借助工作组来共享信息，共享信息的权限设置由每台计算机控制。



### Domain

管理模式）域实现的是主/从管理模式。
账号管理）通过一台域控制器来集中管理域内用户帐号和权限，帐号信息保存在域控制器内。
共享信息）共享信息分散在每台计算机中，但是访问权限由控制器统一管理。



域 对比 工作组 的优点：

> 服务器（域内服务器）和用户的计算机都在同一个域中；
> 用户在域中只需要拥有一个账号，只需要在域中登录一次就可以访问域中授权的资源；





## LDAP

Lightweight Directory Access Protocol相关知识
- LDAP（轻量级目录访问协议）是实现提（供被称为目录服务的）信息服务。
- 核心要点：目录
	- 目录是一树形结构、具体信息存储在条目中；
	- 条目具有DN（Distinguished Name区别名），DN可以用来引用条目；
	- 条目的属性由一个type、多个value构成；
	- 条目的组织一般按照地理位置和关联关系组织、直观呈现；
- LDAP树形结构：
	- 树根一般定义国家(c=CN)或域名(dc=com)；
	- 树根下则往往定义一个或多个组织 (organization)(o=Acme)或组织单元(organizational units) (ou=People)



ldap中的常见关键词

- dc Domain Component
	- 域名的部分，其格式是将完整的域名分成几部分，如域名为example.com变成dc=example,dc=com
- uid User Id
	- 用户ID，如“tom”
- ou Organization Unit
	- 组织单位，类似于Linux文件系统中的子目录，它是一个容器对象，组织单位可以包含其他各种对象（包括其他组织单元），如“market”
- cn Common Name
	- 公共名称，如“Thomas Johansson”
- sn Surname
	- 姓，如“Johansson”
- dn Distinguished Name
	- 惟一辨别名，类似于Linux文件系统中的绝对路径，每个对象都有一个惟一的名称，如“uid= tom,ou=market,dc=example,dc=com”，在一个目录树中DN总是惟一的
- rdn Relative dn
	- 相对辨别名，类似于文件系统中的相对路径，它是与目录树结构无关的部分，如“uid=tom”或“cn= Thomas Johansson”



## AD

ActiveDirectory是微软提出来的概念；

AD利用LDAP协议<font color=red>动态</font>的建立整个域模式网络中的<font color=red>对象</font>的数据库或者索引。所以AD实质是一种存储地址的数据库，或者看成目录亦可。

安装了AD的服务器称为Domain Controller（域控制器），其中存储整个域内对象的信息、并周期性的更新。
负责每一台联入网络的电脑和用户的验证工作，相当于一个单位的门卫一样。



AD中的对象，可抽象成三大类，每个对象都有自己的属性及对应的属性值 ；

- 资源：计算机、打印机；
- 服务：电子邮件；
- 人物：用户、组；




问题：AD vs DNS
> Windows server 2003的AD与DNS密不可分，因为AD会使用DNS服务器来登记DC的ip、及各种资源的定位；





## RADIUS

Remote Authentication Dial In User Service 远程用户拨号认证系统由RFC2865，RFC2866定义，是目前应用最广泛的AAA协议。

RADIUS是一种C/S结构的协议，它的客户端最初就是NAS（Net Access Server）服务器，任何运行RADIUS客户端软件的计算机都可以成为RADIUS的客户端。RADIUS协议认证机制灵活，可以采用PAP、CHAP或者UNIX登录认证等多种方式。RADIUS是一种可扩展的协议，它进行的全部工作都是基于Attribute-Length-Value的向量进行的。RADIUS也支持厂商扩充厂家专有属性。

由于RADIUS协议<font color=red>简单明确，可扩充</font>，因此得到了广泛应用，包括普通电话上网、ADSL上网、小区宽带上网、IP电话、VPDN（Virtual Private Dialup Networks，基于拨号用户的虚拟专用拨号网业务）、移动电话预付费等业务。IEEE提出了802.1x标准，这是一种基于端口的标准，用于对无线网络的接入认证，在认证时也采用RADIUS协议。





## Single Sign On

SSO（Single Sign On）即单点登录，是目前企业应用软件系统的常用集成方案。  
SSO系统为各个应用系统提供统一的登录认证，用户只需要登录一次就可以访问所有相互信任的应用系统，而不需要重新登录。  
SSO不仅带来了更好的用户体验，更重要的是降低了安全的风险和管理的消耗。  

- SSO系统包括SSO Client和SSO Server，SSO Client与SSO Server一起配合完成统一认证服务。
  - SSO Server负责生成凭证及管理凭证。  
  - SSO Client和业务应用系统部署在一起，为业务应用系统提供凭证认证的调用接口，完成业务应用系统和SSO Server之间的通信。  



