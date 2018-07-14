

## 各种转换函数
### 字节序
字节序与系统有关还是cpu架构有关？？

字节序转换（双字节、四字节）
```cpp
#include <arpa/inet.h>

uint32_t htonl(uint32_t hostlong);
uint16_t htons(uint16_t hostshort);

uint32_t ntohl(uint32_t netlong);
uint16_t ntohs(uint16_t netshort);
```

### 地址转换
[各种地址的定义查看：](./type.md#网络地址)
```cpp
char *inet_ntoa(struct in_addr in);
	//将ip地址转成字符串；
in_addr_t inet_addr(const char *cp);
	//转换cp为in_addr_t，即32位一整数；
int inet_aton(const char *cp, struct in_addr *inp);
	//将cp转换成inp；
	//返回非0如果cp为合法Ip；0如果cp无效；
```
