[TOC]


- 问题：安全产品是用来保护什么的（资源 or 人）？  

	如WAF保护什么？ ---web站点

假如防火墙是一幢大楼的门锁，那么IDS就是这幢大楼里的监视系统。

fw topsec（采用了硬件加速ASIC, Application Specific Integrated Circuit，专用集成电路）
ips 绿盟



# 业务功能（这个业务可以单独做成产品中的一个模块、或者直接能做成一个产品）

## 带宽管理

- 带宽管理提供
	- 带宽限制
	- 带宽保证
	- 连接数限制

- FW如何识别应用
	- 特征识别：每种应用和协议都有其特有的消息和命令字；
	- 关联识别：某些应用使用动态端口通信，使用特征识别技术识别出控制通道后，可从控制通道中发现数据通道的通信地址；
	- 行为识别：加密应用没有特征关键字可以作为识别的特征。可以通过特定的行为分析来识别，如报文的商品范围、报文长度统计、报文发送频度、报文收发比例等；



## 反垃圾邮件

在FW上部署邮件过滤功能；

- 邮件过滤技术
	- RBL：基于IP信誉，对发布过垃圾邮件的IP加入黑名单；
	
## 企业网络安全分析报告
NSR network security report

FW能生成这个报告？


## 企业数据防泄漏

利用文件、内容过滤阻止信息通过网络外流；


## 企业安全合规
场景：员工网上发表不当言论；

http协议的应用行为控制，发帖一般都是http的post请求；

部署审计功能：一方面用来了解员工上网行为；另一方面通过审计回溯问题，定位责任人；




## IPS
能阻断名种入侵行为；

FW还有带IPS功能的？

与anti-virus类似IPS提取报文特征与特征库中的签名进行对比，如果匹配，那么该报文属于攻击报文；

- IPS使用
	- 要查看、分析IPS日志，不断调整、优化防御策略；
	- 定期更新IPS特征库；
	


## AV
将恶意代码植入可执行文件就成了病毒；

病毒有很多传播手段：ftp, http, email, 文件共享；

FW能提取文件的特征 并与 病毒库 进行特征对比；

- 防御：
	- 企业终端（电脑、手机、服务器）上都需要装杀毒软件；
	- 企业网络的关键位置（互联网出口处、服务器区域出口处）部署带反病毒功能的FW、专门的防毒网关，用以防止病毒在网络中传播；
	- 企业的网络管理中心能集中管理终端杀软、网络中的防火墙，并及时更新病毒库；
	- 员工要提高意识，不随便打开外网邮件的附件、链接，不访问可能带病毒的非法网站；
	









# 市面上的产品


## FW
### 定义与功能
- 定义：它是一种位于<font color=red>内部网络与外部网络之间的网络安全系统。依照特定的规则，允许或是限制传输的数据通过</font>。
	- 一般有4个网口各自连接：intranet, internet, dmz, 管理口；
	- 保护内部网免受非法用户的侵入（将fw同意的人、数据放入intranet），防火墙主要由服务访问规则、验证工具、包过滤、应用网关4个部分组成；

- 功能：

### 分类
HIDS, NIDS, DIDS(分布式IDS，集成了HIDS, NIDS)







## IDS(Intrusion Detection System)
### 定义与功能
- 定义（技术）：通过抓取网络上的所有报文，分析处理后，<font color=red>报告异常和重要的数据模式和行为模式</font>，使网络安全管理员清楚地了解网络上发生的事件，并能够采取行动阻止可能的破坏。
- 定义（部署）：通过软、硬件，<font color=red>对网络、系统的运行状况进行监视</font>，尽可能发现各种攻击企图、攻击行为或者攻击结果，以保证网络系统资源的机密性、完整性和可用性。








## IPS(Intrusion Prevention System)
[device-ips refer](deviceIPS.md)








## SWG --- security web gateway
### 定义与功能
- 定义（Gartner）：这是一种作用于互联网出口的产品方案，<font color=red>至少包括URL过滤、恶意代码防护和包括Web应用在内的应用控制功能</font>，在保护安全的同时强制执行企业的互联网访问策略。

### SWG(security web gateway)主要做什么？
- 网络中的恶意软件； 
	- 木马、蠕虫、僵尸网络；
- 网络中利用漏洞的攻击行为；
- 高风险的网络行为；
	- 访问高风险的网站、下载带有恶意程序的应用软件、进行不符合企业策略的网络应用；









## WAF
- 定义：web应用防火墙，能有效阻挡OWASP(Open Web Application Security Project)主流的攻击类型。
- 功能：










## CDN
- 定义：在现有的Internet中增加一层新的网络架构，将网站的内容发布到最接近用户的网络“边缘”，使用户可以就近取得所需的内容，提高用户访问网站的响应速度。
- 功能：

### smart CDN










## DLP --- Data leakage prevention






## 上网行为管理

- 作用、功能：
	- 企业网络审计：审计功能不仅是‘事后追责’，更是‘事前预防’；
		- 审计功能一般部署在网络出口设备上；

- 如何实现上网管理
	- URL过滤：基于白名单、黑名单、URL分类查询的过滤方式；
		- 有的设备还支持URL远程查询功能；（本地url库不够用时，去哪里找在线url分类查询的服务呢？）

	
### sangfor
- 定义：管理的目标应该是可视和可控（具体到用户/终端、应用和内容、流量的可视可控）。

- 产品特点
	- 应用和内容
		- 千万级URL库、2500多种应用的应用特征识别库（特征库每2周更新一次）；
		- 内容识别技术（<font color=red>SSL加密内容识别</font>、代理识别等），防止非法内容绕过监管；
		
	- 流量
		- P2P智能流控技术，100%全量识别P2P应用流量；
	
	refer: http://www.sangfor.com.cn/product/net-safe-network-safe-ac.html
		
	




## DDOS

- 定义：，可按层数来分类（4-7层的DDOS）；

[tech-ddos refer](techDDOS.md)











# 附录

## 附录：产品大分类
### 物理安全
### 网络安全（边界防护、结构安全）
- 边界防护
	- 抗DDOS
	- 网络防火墙
	- 下一代防火墙 
	- UTM 
	- 网闸 
	- CDN

- 网络审计
	- 日志审计系统 
	- 安全管理平台
	- IT运维管理平台
	
- 访问控制
	- VPN
	- 上网行为管理
	- 流量控制系统
	
- 入侵防范
	- 入侵检测系统
	- 入侵防御系统

- 恶意代码防范
	- 防病毒网关
	- 反垃圾邮件
	
- 网络行为安全
	- 网络行为分析
	
### 主机安全（身份鉴别、安全审计）
- 身份鉴别
	- 准入系统
	
- 安全审计
	- 数据库审计系统
	- 日志审计系统
	- 运维审计系统
	- 主机审计系统
	
- 入侵防范
	- 主机安全加固
	- 主机入侵防御系统
	
- 恶意代码防范
	- 防病毒软件
	- 杀毒U盘

- 内网安全
	- 终端安全管理

- 安全认证
	- 安全认证


### 应用安全（安全检测、入侵防范）
- 业务安全
	- 业务风险分析系统
	- 业务行为监测

- 应用加速
	- 缓存加速
	- 应用交付

- 安全检测
	- 应用系统漏洞扫描
	- 数据库漏洞扫描
	- 主机系统漏洞扫描
	- 木马检测工具
	- 三合一综合检测系统

- 安全监测
	- 漏洞态势感知系统
	- WEB安全监测系统
	- 应用态势感知系统

- 身份鉴别
	- 认证网关
	- 数字证书

- 应用审计
	- 日志审计系统

- 入侵防范
	- 应用动态防护
	- WEB应用防火墙
	- APT


### 数据安全（数据恢复、防篡改）
- 数据检索
	- 文档检索
	- 舆情系统

- 数据防泄密
	- 数据安全网关
	- 打印审计
	- 刻录审计

- 数据完整性
	- 防篡改软件

- 数据加密
	- 文档加密
	- 安全网盘
	- 加密U盘
	- 加密存储
	- 加密硬盘

- 数据备份及恢复
	- 备份恢复软件
	- 数据备份与恢复
	- 磁盘存储
	- 磁带存储

### 无线安全（信号检测、环境检测）
- 无线安全
	- 无线入侵防御系统

- 信号检测
	- LTE空口分析
	- 网络信令分析
	- 网络模拟平台
	- 无线信号探测
	- 宽频数字探测

- 环境检测
	- 无线摄像扫描
	- 涉密环境安全检查
	- 手机信号屏蔽器

### 移动安全（应用检测、应用加固）
- 病毒、木马检测
	- 软件检测
	- 硬件检测

- 手机加密
	- 手机加密

- 应用检测
	- 个人APP检测
	- 企业APP检测

- 应用加固
	- 个人APP加固
	- 企业APP加固

- 应用加密
	- Android APP加密
	- iOS APP加密

### 云安全（虚拟化安全）
- 云安全
	- 抗DDos
	- 云加密
	- 云安全平台
	- 安全认证
	- 云WAF
	- 云端攻击流量清洗

- 虚拟化安全
	- 虚拟化数据传输
	- 虚拟化认证
	- 虚拟化桌面审计
	- 服务器虚拟化
	- 终端虚拟化
	- 系统镜像加密

### 物联安全（工业安全、汽车安全）
- 物联网安全
	- 物联网监控系统

- 工业安全
	- 工业网闸
	- 工业安全网关
	- 工业安全预警平台
	- 工控安全防护
	- 工控行为追溯
	- 工控安全审计
	- 工控主机安全

- 汽车安全
	- 车辆定位

### 数据取证（安全检测、等保测评）
- 数据校验
	- 时间校验

- 计算机取证
	- 密码破解
	- 取证系统

- 数据取证
	- 数据恢复分析
	- Nuix邮件分析
	- 鉴证大师
	- 硬盘复制

- 手机取证
	- 无线信号屏蔽袋
	- 手机取证软件

