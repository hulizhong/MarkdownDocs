

- 如何在组织内评估计算机安全风险？


## DDoS

- DoS 拒绝服务：是一类攻击（而非一种攻击），可以使你无法为你的最终用户进行服务；
- DDoS（Distributed Denial of Service） 分布式拒绝服务；

### 分类

> 分类（由ddos攻击OSI层分类，一般说的基础设施层攻击指的是网络、传输层）

- 3层攻击：每秒（bps）位进行测定。
	- ICMP-flood：ping洪水
		- 手法：
		- 防御：禁Ping则可；
	
	- Ping of Death
		- 手法：发送大于64k（65536）的ping包给目标，让目标在重组包时造成缓冲区溢出而挂掉；
		- 防御：现在OS都无法发现这样的包了，不用担心；
			

- 4层攻击：以每秒数据包进行测量。
	- SYN-Flood：	
		- 手法：用原始socket假造源地址虚假的syn包，使目标主机无法完成3次握手，从而占用连接资源；   
		- 防御：syn proxy, syn cookies, 首包(第1个syn)丢弃；
	
	- ACK-Flood：
		- 手法：亦是用虚假的ack包，但系统对于陌生的ack默认动作为rst，所以没有syn-flood伤害大；
		
	- UDP-Flood：。
		- 手法：亦是用原始socket创建虚假源地址的udp包，目前主要针对DNS协议；
			
		
		
- 7层攻击：这些攻击是更复杂的，因为它们模仿人类行为与用户界面交互。以每秒的请求数进行测量。
	- CC（ChallengeCollapsar）：是http-flood在国内的叫法；
		- 手法：通过各种技术（botnet,匿名代理服务器）向目标发送大量真实的Http请求；  
			 
			这些请求是筛选过的（一般避开了缓存而需要读取数据库，且数据库查询也要找最耗资源的进行查询），起到了最小攻击资源换取最大拒绝服务的功效；
				
		- 防御：有一些办法，但不能完美解决这个问题；
			
	- DNS-Flood
		- 手法：伪造源地址的海量DNS请求，用于是淹没目标的DNS服务器。
		
			对于攻击特定企业权威DNS的场景，可以将源地址设置为各大ISP DNS服务器的ip地址以突破白名单限制，将查询的内容改为针对目标企业的域名做随机化处理，当查询无法命中缓存时，服务器负载会进一步增大。
				
		- 防御：因为DNS服务不止于dns-53，所以防御的一种思路就是将UDP的查询强制转为TCP，要求溯源，如果是假的源地址，就不再回应。；
			
			对于企业自有权威DNS服务器而言，正常请求多来自于ISP的域名递归解析，所以将白名单设置为ISP的DNS server列表。对于源地址伪造成ISP DNS的请求，可以通过TTL值进一步判断。
		
	- 慢速连接：针对http协议；
		- Slowloris
			- 手法：先建立http连接，设置一个较大的content-length，每次只发送很少的字节，让服务器一直以为http头部没有传输完成，这样的连接一多很快就会出现连接耗尽。
		- slowloris的变种：http慢速的post请求和慢速的read请求都是基于相同的原理。
		
	- DoS
		- 手法：有些服务器程序存在bug、安全漏洞，或架构性缺陷，攻击者可以通过构造的畸形请求发送给服务器，服务器因不能正确处理恶意请求而陷入僵死状态，导致拒绝服务。（已属于漏洞的范畴）
		
	- NTP Reply Flood Attack
		

### 攻击类型

- 混合型：攻击的时候往往混合了多种技术、包括了网络层的、应用层的、TCP的、UDP的；
- 反射型：攻击包的src_ip写成目标者的，然后把攻击包发向真实存在的一些服务（这些机器很有可能是平时常见的公有的）；
- 流量放大型：用递归的效应产生出比攻击流量更多倍的流量；
	
	
		

### 解决：DDOS攻击本质上是一种只能缓解而不能完全防御的攻击



http://www.ayazero.com/?p=75
	
问题：MStream Flood是什么？？
