[TOC]

open web application security project.

- 每年都会有个统计并发布top 10的概念：（如下面是16年的top 10）
	- 注入   Injection
	- 失效身份认证和会话管理   Broken Authentication and Session Management
	- 跨站   Cross-Site Scripting (XSS)
	- 不安全的对象直接引用
	- 伪造跨站请求   Cross-Site Request Forgery (CSRF)
	- 安全误配置   Security Misconfiguration
	- 限制URL访问失败
	- 未验证的重定向和转发
	- 应用已知脆弱性的组件
	- 敏感数据暴露


[top10 good-site](http://blog.csdn.net/lifetragedy/article/details/52573897)


## Injection

- 定义：注入往往是app缺少对输入数据进行安全检查引起的。
	- 攻击者把一些包含指令的数据发送给解释器，解释器会把收到的数据转换成指令执行。
	- 常见的注入有；
		- sql注入：一般可造成数据库信息被读取、篡改，甚至获得管理员权限；
		- OS shell；
		- LDAP
		- Xpath
		- Hibernate

- 危害
	- 执行更多非app意愿的指令；

- 防范
	- 法一：使用ESAPI（enterprise security api）
		- google提供的esapi旨在为编写出更加安全的应用层代码设计出来的一些API；
		- 项目地址：https://github.com/ESAPI






## XSS
- 定义：当app在发送给浏览器的页面中包含用户提供的数据，但没有经过适当的验证、转译，就有可能导致跨站脚本漏洞；（相当于在用户浏览器环境执行脚本）
	- 种类  [site](https://www.lvtao.net/dev/xss.html)
		- 存储式：XSS代码直接储存在服务端；
		- 反射式：XSS代码随着请求提交给服务端，而后再传回到客户端；
		- 基于DOM：服务端不参与，完全在客户端浏览器发生；
	
- 危害：攻击者能在受害者浏览器中执行脚本；
	- 有可能支持用户会话、迫害网站、插入恶意内容、重定向用户、使用恶意软件劫持浏览器。。

- 防范
	- 法一：验证输入
	- 法二：使用ESAPI验证；
	- 法三：编码输出；（需要编码的部分：html实体、html属性、javascript, css, url）







## CSRF
- 定义：攻击者盗用了你的身份，以你的名义发送恶意请求。
	- vs XSS
		- XSS利用站点内的信任用户；
		- CSRF则通过伪装来自受信任用户的请求来利用受信任的网站；
	
	- 要完成一次CSRF攻击，受害者必须依次完成两个步骤：（只要打破以下依赖则可避免csrf）
		1. 登录受信任网站A，并在本地生成Cookie。
		2. 在不登出A的情况下，访问危险网站B。
	
- 危害：以用户的名义（权限）作坏事；
	

- 防范
	-法一： 给每个http请求添加不可预测的token；



[csrf good-site](http://www.cnblogs.com/hyddd/archive/2009/04/09/1432744.html)



