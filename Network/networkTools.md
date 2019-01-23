[TOC]

## wireshark

### TCP Error String When Capture

https://blog.csdn.net/faithc/article/details/52832617?locationNum=1&fps=1

| 信息提示                            | 意义                                                         |
| ----------------------------------- | ------------------------------------------------------------ |
| [RST]                               | 重置连接                                                     |
|                                     |                                                              |
| [TCP Window Full]                   | 发送方的发送窗口全是等确认的数据；即在途字节数等于对方的接收窗口； |
| [TCP ZeroWindow]                    | 接收方的接收窗口已满，发送方不能再发数据                     |
| [TCP Window Update]                 | 接收方接收窗口为0后，待消费了其中的数据之后，主动告知发送方用以更新接收窗口大小； |
| [TCP ZeroWindowProbe]               | 零窗口探测包。<br />当接收方窗口为0后发送方停止发送数据，但此时如果有数据待发送，那么发送方用此包对接收方滑动窗口最新值进行探测。 |
|                                     |                                                              |
| [TCP Out-Of-Order]                  | 收到的seq不是期望值。 -----<font color=red>网络中存在大量的乱序或者丢包</font>。 |
| [TCP segment of a reassembled PDU]  | 分片重组。<br />连接建立时会协议出MSS值用于分片（1460），如果后续有报文被分片，那么<font color=red>其中的每片ACK是一样的，但seq不一样</font>。这些分片正常应该是连续接收的，即前一个分片指示的next seq number即为下一个收到的分片的seq number。 |
| [TCP Previous segment not captured] | 前一分片未被捕获。<br />只要捕获不到连接的分片那么就会报此错误，需要注意的是，<font color=red>有可能是网络中确实丢失了，或者晚到了，也有可能是wireshark本身并没有抓到</font>。 |
|                                     |                                                              |
| [TCP Spurious Retransmisson]        | 虚假重传。<br />当抓到2次同一包数据时，wireshark判断网络发生了重传，同时，wireshark抓到初传包的反馈ack，因此wireshark判断初传包实际并没有丢失，因此称为虚假重传。 |
| [TCP Retransmission]                | 重传。<br />当抓到2次同一包数据时，wireshark判断发生了重传，同时wireshark没有抓到初传包的反馈ack，因此，wireshark判定重传有效。 |
| [TCP Fast Retransmission]           | 快速重传。<br />TCP协议设定了快速重传机制以避免过多的慢启动对传输速率的影响。快速重传通过接收到3个或3个以上重复的ack反馈触发。快速重传不需要等待RTO超时。 |
|                                     |                                                              |
| [TCP Dup ACK]                       | 重复ACK。<br />当网络中存在<font color=red>乱序或者丢包</font>时，将会导致接收端接收到的seq number不连续。此时接收端会向发送端回复重复ack，ack值为期望收到的下一个seq number。重复ack数大于等于3次将会触发快速重传。 |
| [TCP ACKed unseen segment]          | 反馈ACK指向了一个未知的TCP片段。<br />ACK反馈的是一个wireshark上不存在的TCP包。很可能是wireshark漏抓了这个包，但却抓到了对端反馈的该报的ack包。 |
|                                     |                                                              |
|                                     |                                                              |
|                                     |                                                              |





### Capture localback addr

当客户端和服务器都在自己的机器上时,用wireshark是看不到发送和接受到的数据包的.
一种方法是创建一个loopback网卡. 这个比较麻烦而且不一定有效.. 参见http://wiki.wireshark.org/CaptureSetup/Loopback
如果你的电脑再局域网中, 也就是说是有网关服务器的情况下, 你可以通过改变路由设置. 把客户端发往服务的包指定到网关服务器上.
这样数据包就是通过网关饶了一圈再回来, wireshark就能拦截到数据包了. 路由设定配置如下: 我在win7上试过是可以的. 其他的os不清楚



假设你的ip是172.17.8.32(不是127.0.0.1是实际的网卡地址), 网关服务器是172.17.8.253
通过下面的命令把数据包指向网关服务器
route add 172.17.8.32 mask 255.255.255.255 172.17.8.253 metric 1
要想再设置回来可以用下面的命令
route delete 172.17.8.32
route add 172.17.8.32 mask 255.255.255.255 172.17.8.32 metric 1
ip地址和网关地址可以通过 ipconfig来查看.



---

法二：使用工具 rawcap






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


- 过滤器中的关键词（**以下这些都不加参数连接符-**）
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

