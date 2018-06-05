[toc]

## 从内网ip查找用户
cmd
nbtstat -a 172.22.48.100
```bash
C:\Users\hulizhong>nbtstat -a 172.22.48.100

	本地连接:
	节点 IP 址址: [172.22.48.100] 范围 ID: []
	
			   NetBIOS 远程计算机名称表
	
		   名称               类型         状态
		--------------------------------------------
		HULIZHONG-EXT  <00>  唯一        已注册
		SKYGUARDMIS    <00>  组          已注册
		HULIZHONG-EXT  <20>  唯一        已注册
		SKYGUARDMIS    <1E>  组          已注册
	
		MAC 地址 = 34-64-A9-7C-C8-B7
```
