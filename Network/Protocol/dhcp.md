[TOC]



## ReadMe

dhcp & dhcpv6.





## DHCP

Dynamic Host configuration protocol， 动态主机配置协议。
为客户机自动分配IP地址、子网掩码以及缺省网关、DNS服务器的IP地址等TCP/IP参数。这些信息配置于dhcp的服务器中，当dhcp client有请求时，dhcp server从地址池中拿一个给dhcp client。

基于UDP
dhcp client监听在udp.546上；dhcp server和中继代理监听在udp.547上。



### 租约交互过程

1. client -----dhcp discover------> server.
   此时client没有任何tcp/ip的参数，向全网发出dhcp discover广播，后续只要是dhcp server均会有响应此请求。
2. server -------dhcp offer ----> client.
   dhcp server在收到discover之后，会进行一个offer的响应，包含如下：
   此用户之前是否有用过的ip，并且此ip是否被别人占用了？如果没有被占用，那么提供给客户；
   针对此用户是否有特殊的ip绑定设置？如果有，那么提供给客户；
   如果上述两条不满足，那么随机选一条ip，连同租约信息，封装在offer包中单播给客户；（<font color=red>这个只是client的备选！</font>）
3. client ------------dhcp reqeust ----> server.
   client从众多的offer中选择一个使用，并将此offer信息封装成reqeust广播出去（让其它dhcp server回收预分配的ip）；
   同时发出一个arp探测，选中的ip是否被网络中其它用户使用。如果有，那么发出dhcp decline请求，并重新discover.
4. server ----------------dhcp ack -----------client.
   收到reqeust请求，之后将回复ack信息，其中包含了其它网络配置，client将正式进行网卡的绑定！（租约确认！）



### 重新登录

client登录时，不会进行discover的步骤，而是将原来分配给自己的ip封装成dhcp reqeust进行请求！
如果server不满足让client使用该地址，那么就会回复dhcp NAck，相反应该是dhcp ack.

### 更新租约

client从server获取的ip是有租借期限的！满期之后会被server收走。

client延长租约。
当client登录、租约过一半时均会向server发送租约信息。
当租约过50%时，会单播reqeust请求，用于延长租约。--dhcp ack则是同意。
当租约过87.5%时，会广播reqeust请求，用于延长租约。---dhcp ack则是同意。
当client到期时会发dhcp release向server进行ip及相关配置的释放。提前结束租约也是release包。



## DHCPV6

