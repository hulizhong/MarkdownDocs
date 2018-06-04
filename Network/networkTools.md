[TOC]


## wireshark


## tcpdump

参数
> -n : 不把网络地址显示为域名  
> -nn : 不显示域名和端口名  
> -X : 在输出行同时打印ASCII字符和HEX十六进制显示的包信息  
> -XX : 在-X基础上，增加了数据链路层的头部信息的显示  
> -v, -vv, -vvv : 逐步提高抓取信息的详细程度  


- 抓包
	* -i eth0 -s 0 –w xx.pcap …filter…

- 选包
	* -r in.pcap –w out.pcap –s 0 …filter..


- 过滤器中的关键词
	* 类型：host、net、port，默认为host；
	* 传输方向：src、dst，默认为dst or src；
	* 协议：ip、arp、tcp、udp；默认所有协议；
	* 逻辑：not !、and &&、or ||；
	* 其它：gateway、broadcast；  （packesize）：less、greater；  portrange 20-21




## ssldump

## tcpreplay
它先使用嗅探工具抓取的数据包，再将流量拆分(prep)客户端和服务端流量，重写(rewrite)2、3、4层头信息，最后再将流量回放(replay)到网络中。  
> 它是一套工具的集合，包括Tcpprep 。。。

### Tcpprep

8种流量拆分模式：
- Auto/Bridge
- Auto/Router
- Auto/Client
- Auto/Server
- IPv4/v6 matching CIDR
- IPv4/v6 matching Regex
- TCP/UDP Port
- MAC address

