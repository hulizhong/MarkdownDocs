

### 字节序

字节序与系统有关还是cpu架构有关？？


- 字节序转换（双字节、四字节）

	```cpp
	#include <arpa/inet.h>

	uint32_t htonl(uint32_t hostlong);
	uint16_t htons(uint16_t hostshort);

	uint32_t ntohl(uint32_t netlong);
	uint16_t ntohs(uint16_t netshort);
	```

