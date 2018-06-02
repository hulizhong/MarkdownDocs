[TOC]

## ReadMe
boto是Python的AWS开发工具包。
其中又分为很多子库，用于对AWS的各项服务进行API操作。 

安装如下，也可以直接下载boto库来进行编程
```bash
# apt-cache search boto
python-boto - Python interface to Amazon's Web Services
# apt-get install python-boto
```

s3
> 对象存储服务，可以用ceph来实现，不知道Amazon是不是也用的这个东西。 


Amazon Web Services (AWS) 开始以 Web 服务的形式向企业提供 IT 基础设施服务，现在通常称为云计算。
> 计算、存储、数据库、迁移、网络和内容分发 。。。  
> 其中存储服务里面就有项Amazon Simple Storage Service (S3)服务。



```python
# 上传数据时，可以加http协议的控制头；
myheader = {"Connection" : "Keep-Alive"}
key.set_contents_from_filename(filename = "/home/install/install.sh", headers = myheader)
```

## ceph
ceph是一种对象存储的开源软件；


## boto-s3操作
以下操作都进行了抓包分析
[s3连接](#s3连接)
[bucket查询](#bucket查询)
[bucket创建](#bucket创建)
[key查询](#key查询)
[key创建](#key创建)
[key删除](#key删除)
[bucket删除](#bucket删除)

### s3连接
```python
self.conn = boto.connect_s3(
        aws_access_key_id = cephid,
        aws_secret_access_key = cephkey,
        host = cephip,
        port = cephport,
        is_secure = False,                                         
        calling_format = boto.s3.connection.OrdinaryCallingFormat()
        )
```

### bucket查询
bucket = self.conn.lookup(bucketname)
```bash
HEAD /hulzdegub/ HTTP/1.1
Host: 172.22.132.210:8070
Accept-Encoding: identity
Date: Fri, 25 Aug 2017 05:30:24 GMT
Content-Length: 0
Authorization: AWS C8BIVY2F3ODJVKRR14ZM:rvrLaygkC7WpWdwpWQ5YvzC7D7g=
User-Agent: Boto/2.34.0 Python/2.7.3 Linux/3.2.0-4-amd64

HTTP/1.1 200 OK
X-RGW-Object-Count: 1
X-RGW-Bytes-Used: 4627506
x-amz-request-id: tx000000000000000000052-00599fb65c-3729-default
Content-Length: 0
Date: Fri, 25 Aug 2017 05:32:12 GMT
```
	
```bash
HEAD /hulzdegub/ HTTP/1.1
Host: 172.22.132.210:8070
Accept-Encoding: identity
Date: Fri, 25 Aug 2017 05:30:23 GMT
Content-Length: 0
Authorization: AWS C8BIVY2F3ODJVKRR14ZM:HtbwGRJTKDJ5tDdPgg6jKg1Pw2w=
User-Agent: Boto/2.34.0 Python/2.7.3 Linux/3.2.0-4-amd64

HTTP/1.1 404 Not Found
x-amz-request-id: tx00000000000000000004e-00599fb65c-3729-default
Content-Length: 219
Accept-Ranges: bytes
Content-Type: application/xml
```

### bucket创建
bucket = self.conn.create_bucket(bucketname)
```bash
PUT /hulzdegub/ HTTP/1.1
Host: 172.22.132.210:8070
Accept-Encoding: identity
Date: Fri, 25 Aug 2017 05:30:23 GMT
Content-Length: 0
Authorization: AWS C8BIVY2F3ODJVKRR14ZM:K2qU7XZWpQY4NUwQbgPTXDrN1rg=
User-Agent: Boto/2.34.0 Python/2.7.3 Linux/3.2.0-4-amd64

HTTP/1.1 200 OK
x-amz-request-id: tx00000000000000000004f-00599fb65c-3729-default
Content-Length: 0
Date: Fri, 25 Aug 2017 05:32:12 GMT
```

### key查询
key = bucket.get_key(objectname)
```bash
HEAD /hulzdegub/dfafadsf HTTP/1.1
Host: 172.22.132.210:8070
Accept-Encoding: identity
Date: Fri, 25 Aug 2017 05:30:23 GMT
Content-Length: 0
Authorization: AWS C8BIVY2F3ODJVKRR14ZM:O/YF3iU7ViLq1k+BhBhtyKHQc1w=
User-Agent: Boto/2.34.0 Python/2.7.3 Linux/3.2.0-4-amd64

HTTP/1.1 404 Not Found
x-amz-request-id: tx000000000000000000050-00599fb65c-3729-default
Content-Length: 216
Accept-Ranges: bytes
Content-Type: application/xml
Date: Fri, 25 Aug 2017 05:32:12 GMT
```
	
### key创建
key = bucket.new_key(objectname)
```bash
PUT /hulzdegub/dfafadsf HTTP/1.1
Host: 172.22.132.210:8070
Accept-Encoding: identity
Content-Length: 4627506
Content-MD5: 9Uz+8nr87uGAjrXl9ySb/Q==
Expect: 100-Continue
Date: Fri, 25 Aug 2017 05:30:23 GMT
User-Agent: Boto/2.34.0 Python/2.7.3 Linux/3.2.0-4-amd64
Content-Type: application/octet-stream
Authorization: AWS C8BIVY2F3ODJVKRR14ZM:c5Dhd13ht/TOcVr0La89wJmJbyk=

....data...

HTTP/1.1 200 OK
X-RGW-Object-Count: 1
X-RGW-Bytes-Used: 4627506
x-amz-request-id: tx000000000000000000052-00599fb65c-3729-default
Content-Length: 0
Date: Fri, 25 Aug 2017 05:32:12 GMT
```


### key删除
bucket.delete_key(objectname)
```bash
DELETE /hulzdegub/dfafadsf HTTP/1.1
Host: 172.22.132.210:8070
Accept-Encoding: identity
Date: Fri, 25 Aug 2017 05:30:24 GMT
Content-Length: 0
Authorization: AWS C8BIVY2F3ODJVKRR14ZM:x56AnlchESmxrL1KI4PiKCCAsZs=
User-Agent: Boto/2.34.0 Python/2.7.3 Linux/3.2.0-4-amd64

HTTP/1.1 204 No Content
x-amz-request-id: tx000000000000000000053-00599fb65c-3729-default
Date: Fri, 25 Aug 2017 05:32:12 GMT
```


### bucket删除
self.conn.delete_bucket(bucketname)
```bash
DELETE /hulzdegub/ HTTP/1.1
Host: 172.22.132.210:8070
Accept-Encoding: identity
Date: Fri, 25 Aug 2017 05:30:24 GMT
Content-Length: 0
Authorization: AWS C8BIVY2F3ODJVKRR14ZM:SBnnmeTmfHQ3RqPGPzOt6ugAT0U=
User-Agent: Boto/2.34.0 Python/2.7.3 Linux/3.2.0-4-amd64

HTTP/1.1 204 No Content
x-amz-request-id: tx000000000000000000054-00599fb65c-3729-default
Date: Fri, 25 Aug 2017 05:32:12 GMT
```
	


## 问题集
- 403-forbidden  error.code-RequestTimeTooSkewed
> ceph的默认事务处理时间为15min，超过这个时间就会报错；
> 注意一个事情，如果事务客户端、服务端的机器时差超过这个值亦会报这个错误；（客户端请求的http请求中带了客户端的时间截）
>> ceph配置项在 MaxAllowedSkewMilliseconds


- boto能否支持到S3的https连接？ ceph呢？
> boto好像不行吧？？


