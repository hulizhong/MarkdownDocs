[TOC]



### 网络转发
网络设备里面区分不同的路径是一个很自然的选择，因为网络设备的首要任务是转发网络包，不同的网络包，在设备里面的处理路径不同。路径一般会有如下概念：
- fast path 就是那些可以依据已有状态转发的路径，在这些路径上，网关，二层地址等都已经准备好了，不需要缓存数据包，而是可以直接转发。  
- slow path 是那些需要额外信息的包，比如查找路由，解析MAC地址等。  
- first path 是设备收到流上第一个包所走过的路径，比如tcp里面的syn包，有些实现把三次握手都放到first path里面处理，而有些只需处理syn包，其他包就进入fast path的处理路径。



在NP或者ASIC里面也需要区分slow path和fast path，fast path上的包都放在SRAM里面，而slow path的包放在DRAM里面。不过这也非绝对。决定处理路径的是对于速度的考虑。处理器的速度，内存的速度等。访问内存的速度由速度和带宽两个因素决定。因此，把SRAM放到fast path上，也是很自然的选择。
系统的性能要从两方面考虑，硬件的设计和软件的设计，这两个方面的配合或者是分工，决定着系统的性能。

### NP
网络处理器（Network Processor，简称NP），根据国际网络处理器会议（Network Processors Conference）的定义：网络处理器是一种可编程器件，它特定的应用于通信领域的各种任务，比如包处理、协议分析、路由查找、声音/数据的汇聚、防火墙、QoS等。




## IDC

- ICP（Internet Content Provider，internet内容提供商）
	- 需要获取ICP证；

- ISP（Internet Service Provider，internet服务提供商）
	- internet接入服务：用网线把你的计算机接入到Internet； 
	- 需要获取ISP证；

icp是厨子，为你提供食物；isp是传菜，把食物送到你面前。

- ASP（Application Service Provider，网络应用服务商）
	- 提供基于internet的应用服务；
	- 以“月租”或“日租”等形式代替“购买” 
	- 当今的科技新服务已经发展到自来水般，打开水龙头便递送到家的“自来水式科技（Technology on Tap）”
	- 不同于ICP, ISP, IDC，这个不需要资质；

- IDC（internet data center，互联网数据中心）
	- 业务
		- 主机托管：企业将自己的服务器放置在IDC内对外服务；
		- 机柜租用：租用IDC的机柜，存放自己的服务器；（rabin：应该比主机托管的量更多吧）
		- 带宽租用：
		- 空间租用：不是存储空间，而是IDC机房空间（可以是1U,2U，或者是整机柜空间之类的）；

	- 主机托管 VS 虚拟主机：虚拟主机是多个用户共享一台服务器；
		

- 虚拟机主 virtual host virtual server
	- 利用软硬件技术将一台主机分成一台台虚拟的‘主机’，这些‘主机’具有完整的Internet服务器的功能；

	- VPS virtual private server
		- 可利用VPS在一台服务器上创建出相互隔离的虚拟专用服务器；

- chinanet 中国骨干网
	- 电信、网通南北分治，故南北互联速度较慢；

- CN2
	- 中国电信下一代承载网（ChinaNetNextCarryingNetwork）。
	- 中国电信构建的CN2网络，力图奠定未来10-20年里中国电信顶级运营商的基础。


### IDC网络建设（机房网络分层结构）
- 网络出口层：
	- 网络带宽比核心层高，如10G；
	
- 核心层：
	- 使用到核心层级别的交换机；
	- 网络带宽比汇聚层高，如GE（Gigabit Ethernet千兆以太网，速率是1000Mbit/s）；
	
- 汇聚层：
	- 使用到汇聚层级别的交换机；
	- 网络带宽比接入层高，如FE（Fast Ethernet快速以太网，指的是百兆，速率是100Mbit/s）；

- 接入层：
	- 使用到接入层级别的交换机；
	- 网络带宽一般为FE、GE；

