[TOC]


## POC(Proof of Concept)课题

### 制作镜像
> 制作Image有两种方法
>> 1. 把更改的container进行commit保存成新的镜像； 
>> 2. 用dockerfile在父镜像上一层一层的叠加；


- 问题
	- 怎么在image中安装在本地的安装包；
		- add的时候，注意src路径是从Dockerfile目录计算的。
	
	- 制作镜像失败时可以前台运行成container，进去之后再运行失败的指令看表现或者环境。

	

#### 从当前运行系统的基础上构建一个os image
> https://linux.cn/article-5427-1.html#3_2726

1. 从当前运行的系统构建基础系统

	```bash
	#参数1：发行版代码，其脚本文件在/usr/share/debootstrap/scripts/目录中；
	##‘发行版代号’可以在lsb_release  -c中看到。
	debootstrap xenial bootstrapXenial
	```

2. 将构建好的系统目录导入到docker中作为系统镜像

	```bash
	tar -c . | docker import - rabin/ubuntu16.04:base 
	```

3. 编写Docerfile

	```bash
	#默认名字为Dockerfile，存放于builddir目录下；
	```

4. 生成镜像
	
	```bash
	docker build /home/builddir -t rabin/debian:xx
	```
	
5. 如果用了缓存生成镜像，那么删除临时镜像

	```bash
	#删除中间的临时镜像
	docker ps -a | awk '{print $1}' | xargs -i docker rm {}
	docker images -a | grep none | awk '{print $3}' | xargs -i docker rmi {}	
	```

#### Dockerfile
> Dockerfile是自动构建docker镜像的配置文件，Dockerfile中的命令非常类似linux shell下的命令，注释用#号。
>> docker build -t [registry_url/namespace/image_name:tag] [Dockerfile_dir]  
>> 1. 在Dockerfile中每执行一条指令（ENV、ADD、RUN等命令），都会生成一个docker image layer。  
>> 2. 构建docker镜像时，如果构建失败，仍会有部分镜像layer生成，再次构建会基于第一次构建所生成的layer（use cache）。  
>> 3. docer镜像由layers叠加而成，体现了docker镜像是分层的。   
>> 4. 一般，Dockerfile分为4部分
>>> 基础镜像（父镜像）信息	 
>>> 维护者信息  
>>> 镜像操作命令  
>>> 容器启动命令  

- from, maintainer
	```bash
	# 基础父镜
	FROM       centos:centos7.1.1503 
	
	#维护者
	MAINTAINER Carson,C.J.Zeong <zcy@nicescale.com>
	```

- env
	> 指定一个环境变量，会被后续 RUN 指令使用，并在容器运行时保持
	
	```bash
	#时区
	ENV TZ "Asia/Shanghai"
	
	ENV TERM xterm
	```
	
- add, copy 
	> 拷贝本地文件到容器中；
	>> add除了本地文件，还可以是url所指内容；自动解压本地tar文件成容器中的目录；
	
	```bash
	ADD aliyun-mirror.repo /etc/yum.repos.d/CentOS-Base.repo
	```

- run 运行指令
	```bash
	#格式1：将在 shell 终端中运行命令，即 /bin/sh -c
	RUN <command>
	#格式2：使用 exec 执行。
	RUN ["executable", "param1", "param2"]
	
	RUN apt-get install -y apache2 || (apt-get -f install && apt-get install -y apache2)
	```

- expose 暴露端口
	```bash
	#对象暴露22端口。EXPOSE <host_port>:<container_port>
	EXPOSE 22
	
	#expose的端口必须在container启动时加参数
	## 将宿主机的2222端口映射到容器的22端口
	docker run -p 2222:22
	## 将宿主机的一个未使用的随机端口映射到容器的22端口
	docker run -P 22
	```

- entrypoint/cmd
	> 同是container启动时执行的命令；只能执行一条；并存时cmd作为entrypoint的参数；
	>> cmd指令可被docker run参数替换；可当作entrypoint参数使用；  
	>> entrypoint指令不可被替换；
	
	```bash
	#格式一
	ENTRYPOINT ["executable", "param1", "param2"]
	#格式二：
	ENTRYPOINT command param1 param2（shell中执行）
	
	#格式1：使用 exec 执行，推荐方式；
	CMD ["executable","param1","param2"]
	#格式2：在 /bin/sh 中执行，提供给需要交互的应用；
	CMD command param1 param2
	#格式3：提供给 ENTRYPOINT 的默认参数；
	CMD ["param1","param2"]
	
	#一个Dockerfile中只有最后一条ENTRYPOINT生效，并且每次启动docker容器，都会执行ENTRYPOINT
	ENTRYPOINT ["/usr/bin/supervisord", "-n", "-c", "/etc/supervisord.conf"] 
	```

- user
	> 指定运行容器时的用户名或UID，后续的 RUN 也会使用指定用户；
	
- volume
	> 将本地文件夹或者其他container的文件夹挂载到container中。

- workdir
	> 为后续的 RUN、CMD、ENTRYPOINT 指令配置工作目录。

- onbuild
	> 配置当所创建的镜像作为其它新创建镜像的基础镜像时，所执行的操作指令。
	>> 其指定的命令在构建镜像时并不执行，而是在它的子镜像中执行。


## 建立本地仓库

> 我们通过官方提供的registry镜像来简单搭建本地的私有仓库环境。

1. 拉registry 

	```bash
	#从docker hub上拉取仓库镜像
	docker pull registry 
	```

2. 运行镜像

	```bash
	mkdir -p /var/data/registry
	
	#把仓库容器跑起来
	##存储镜像的地方需要修改配置文件/etc/docker/registry/config.yml，更正到/tmp/registry中。
	docker run -d -p 5000:5000 -v /var/data/registry:/tmp/registry registry
	
	#确保registry镜像运行状态
	docker ps
	``` 

3. 本地镜像打tag、上传

	```bash
	#先将本地镜像debian:7.8打tag
	docker tag debian:7.8 172.22.111.131:5000/debian:7.8 
	
	#push镜像
	docker push 172.22.111.131:5000/debian:7.8	
	> Get https://172.22.111.131:5000/v1/_ping: http: server gave HTTP response to HTTPS client
	>> server开启了http, client开启了https，解决办法如下：
	>>> vim /etc/docker/daemon.json 增加以下内容
	>>> {"insecure-registries" : ["172.22.111.131:5000"]}
		
	## 这个办法没能解决
	> Get https://172.22.111.131:5000/v1/_ping: http: server gave HTTP response to HTTPS client
	>> 报错是因为docker启的时Https，而交互用的http。在以下配置中新增 --insecure-registry 参数即可。
	>>> vim /etc/init/docker.conf
	>>> exec "$DOCKERD" $DOCKER_OPTS --raw-logs --insecure-registry

	```

4. 查看本地仓库

	> 不能用docker指令操作。查询、删除均需要http api来实现吗？

	1. 用http api查看： https://docs.docker.com/registry/spec/api/#deleting-a-layer

		```bash
		# 浏览器查看以下链接，是否有刚推上去的内容
		http://172.22.111.131:5000/v2/_catalog
		> {"repositories":["debian"]}
	
		http://172.22.111.131:5000/v2/debian/tags/list
		> {"name":"debian","tags":["7.8"]}
		```

## xx应用集群
> 用xx镜像作出一个xx集群的场景；


- docker编排工具
	- Docker Machine
	- Docker Compose
	- Docker Swarm
	- k8s
	

### kubernetes
> google出口的基于容器技术的分布式系统；

k8s概念总结： https://www.cnblogs.com/zhenyuyaodidiao/p/6500720.html


- 组件
	- namespace：关联所有对象，除了网络；
	- pods: collocated group of docker containers that share an ip and storage volume.
	- service: single, stable name for a set of pods, also acts as LB.
	- replication controller: manages the lifecycle of pods and ensures specified number are running.
	- used to organize and select group of objects.
	
#### k8s安装（单机部署）

- 问题
	- 关闭selinux, iptables
		- ubuntu貌似不装selinux, 关闭了就如同win系统c2级别；有则为b1系统；
		- selinux不关会影响一些连接吧；
	
	- 容器引用有：docker, rtk


- 基础环境安装
	```bash
	# 编译make所需要
	apt-get install cmake
	
	apt-get install golang
	```


##### 单机部署

1. 下载或者生成k8s组件；

	|名称|功能|
	|--------------|-----------|
	|cloud-controller-manager |
	|hyperkube |
	|kubeadm |
	|kube-aggregator |
	|kube-apiserver |作为整个系统的控制入口，以REST API服务提供接口。
	|kube-controller-manager |用来执行整个系统中的后台任务，包括节点状态状况、Pod个数、Pods和Service的关联等。
	|kubectl |客户端命令行工具，将接受的命令格式化后发送给kube-apiserver，作为整个系统的操作入口。
	|kubefed |
	|kubelet |运行在每个计算节点上，作为agent，接受分配该节点的Pods任务及管理容器，周期性获取容器状态，反馈给kube-apiserver。
	|kube-proxy |运行在每个计算节点上，负责Pod网络代理。定时从etcd获取到service信息来做相应的策略。
	|kube-scheduler |负责节点资源管理，接受来自kube-apiserver创建Pods任务，并分配到某个节点。

2. 安装etcd
	```bash
	apt-get install etcd
	
	#配置文件
	vim /etc/default/etcd
	ETCD_DATA_DIR="/var/lib/etcd/default"
	## 客户端操作etcd API的端口，默认指定端口4001，v2中改变为2379，在k8s中我们要使用4001端口
	ETCD_LISTEN_CLIENT_URLS="http://localhost:4001"
	## 作为分布式的客户端连接端口
	ETCD_ADVERTISE_CLIENT_URLS="http://localhost:4001" 
	```

3. 拷贝master端相关组件
	```bash
	cp kube-apiserver kube-scheduler kube-controller-manager /usr/bin/
	```
	
	1. vim /etc/default/kube-apiserver
	
		```bash
		KUBE_APISERVER_OPTS="\
		--insecure-bind-address=0.0.0.0 \   api监听地址
		--insecure-port=8080 \  api监听端口
		--service-cluster-ip-range=72.22.0.0/16 \ service给pod分配的Ip范围
		--etcd_servers=http://127.0.0.1:4001 \  etcd连接地址
		--logtostderr=true"
		```
	
	2. vim /etc/default/kube-controller-manager
	
		```bash
		KUBE_CONTROLLER_MANAGER_OPTS="\
		--master=127.0.0.1:8080 \
		--logtostderr=true"
		```
	
	3. vim /etc/default/kube-scheduler
	
		```bash
		KUBE_SCHEDULER_OPTS="\
		--master=127.0.0.1:8080 \
		--logtostderr=true"
		```

4. 拷贝minion相关组件 
	```bash
	cp  kubelet kube-proxy /usr/bin/
	```

##### k8s组件下载/生成

- 法一：源码编译
	```bash
	# 在官网 https://github.com/kubernetes/kubernetes/releases 下载 source code .tar.gz
	make quick-release
	```
	
- 法二：根据运行平台下载二进制  
	```bash
	# 在官网 https://github.com/kubernetes/kubernetes/releases 下载 kubernetes.tar.gz
	./cluster/get-kube-binaries.sh
	```
	
- 法三：直接从google仓库下载二进制
	```bash
	# 内部包含kubefed, kubectl两工具；
	curl -k https://storage.googleapis.com/kubernetes-release/release/v1.6.2/kubernetes-client-linux-amd64.tar.gz
	
	# 其实已经包含了client的内容；
	curl -k https://storage.googleapis.com/kubernetes-release/release/v1.6.2/kubernetes-server-linux-amd64.tar.gz
	```


### swarm
### mesos	

### 网络怎么搞
### 如何管理调度这些containers
