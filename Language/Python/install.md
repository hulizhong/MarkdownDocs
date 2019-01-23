[TOC]

## ReadMe

## install python.
下载源码包、编译、安装
```python
# https://www.python.org/downloads/source/  
# 下载源码包；
# 解压：`tar -zxvf Python-2.5.6.tgz`  or `bzcat Python-2.5.6.tar.bz2 | tar -xf -`
# 运行指令："./configure", "make", "make install" commands to compile and install Python. 
```

### 特殊目录

/usr/share/pyshared/
> 这些库不管是哪个版本的python都能用

/usr/lib/python2.6(2.7/3)
> 这下面的库，只能是特定版本的python能用



### 环境变量

sys.path —— 动态地改变Python搜索路径
```python
import sys  
sys.path.append('引用模块的地址')  
sys.path.insert(0, '引用模块的地址')  
print sys.path
	#['', '/usr/lib/python2.7', '/usr/lib/python2.7/plat-linux2', '/usr/lib/python2.7/lib-tk', '/usr/lib/python2.7/lib-old', '/usr/lib/python2.7/lib-dynload', '/usr/local/lib/python2.7/dist-packages', '/usr/local/lib/python2.7/dist-packages/docker_registry-1.0.0_dev-py2.7.egg', '/usr/local/lib/python2.7/dist-packages/setuptools-36.0.1-py2.7.egg', '/usr/local/lib/python2.7/dist-packages/pip-8.0.1-py2.7.egg', '/usr/lib/python2.7/dist-packages', '/usr/lib/python2.7/dist-packages/PIL', '/usr/lib/python2.7/dist-packages/gtk-2.0', '/usr/lib/pymodules/python2.7']
```

问题：dist-packages VS site-packages ？？
> 用工具安装(pip..)的库均会到dist目录，如/usr/local/lib/python2.7/dist-packages；手动安装的库会到site目录下。   
>
> > dist目录下的库，每个库对应一个同名的dist-info或者egg-info目录。应该分别对应两种对应的打包工具？？      



### 包的部署与管理

**distutils**是Python标准库的一部分。提供了打包、包安装的服务。 

**setuptools** distutils的增强版。   
引入包依赖管理；可为python包创建egg文件；提供**easy\_install**脚本来安装egg包。   
python的egg文件有点像java中的jar文件，是一个工程打包文件，便于安装部署。

**pip** easy\_install的增强版。  
无需要使用egg文件；
安装失败后不会出现只安装一部分的情况；
跟踪所有安装的包。  



### setup.py

一般源码包安装软件，借助`setup.py`。

> 依赖 setuptools，（安装：apt-get install python-setuptools）



普通软件源码安装包中，包含`setup.py`文件，如下操作即可安装：

```python
python setup.py build
python setup.py instal
python setup.py instal --record ~/pip/srcInstall/packagename.list
	#setup.py安装的办法，pip跟踪不了，只能依赖file.list来删除。
```





### wheel file







## pip 

安装软件时，如果软件内部发生异常，那么可以尝试更换源进行安装！
如果仅仅是下载不来，那么应该和源没什么多大关系！！



### install

法一，安装包安装。（下载pip的tar.gz 进行）

```python
python setup.py build
python setup.py install --record file.list
```



法二，在线安装。

```python
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
python get-pip.py --no-setuptools --no-wheel
    #https://pip.pypa.io/en/stable/installing/
```

> Retrying (Retry(total=3, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLError(1, '_ssl.c:504: error:14090086:SSL routines:SSL3_GET_SERVER_CERTIFICATE:certificate verify failed'),)': /simple/pip/



法三，安装系统默认带的`python-pip`包。

```bash
# apt-cache search python-pip
python-pip - alternative Python package installer

# apt-get install python-pip
```





### config

注意，<font color=red size=4>这个章节里面讲的根据pip版本不一致，会导致有些 特性不支持！！</font>

~/.pip/pip.conf

```bash
[global]
trusted-host = pypi.douban.com  #如果是ssl连接的话，则信任之，不需要验证。
no-cache-dir = ture      #重新下载包、不使用缓存包。
ignore-installed = ture  #安装了之后，不再安装（即使版本不一致）
no-dependencies = true

[install]
index = http://pypi.douban.com/simple
index-url = http://pypi.douban.com/simple  #源，貌似index-url比index更通用！！
   
[search]
index = https://pypi.python.org/pypi
index-url = https://pypi.python.org/pypi
	#pip的官网源
	#上面的库都只链接了最新的版本，如何下载历史版本？   
	#找到该项目的github地址，然后再在发布版本中去找（打了tag）。  
```

环境变量

```bash
export PIP_CONFIG_FILE=/root/.pip/pip.conf
export PIP_CERT=/etc/ssl/certs/
```



### usage

pip命令

```bash
pip install package_name  
pip install package_name==version  
	#如果安装成功，那么在最后会显示： Successfully installed markdoc 之类的。  
pip install --upgrade package_name==version  
pip uninstall package_name   
	#不能删除setup.py安装类的库（setup.py类型的只能用record方法）。  

pip list  #列出已安装包.  
pip list --outdated  #查看所有过期的库；
pip show name   #显示包详细信息。（包括版本、位置之类的）    
pip search name  #搜索包，类似yum里的search. 
```



### Q & A

#### QA, setuptools is too old

用pip安装软件失败，并显示以下内容？？

> your setuptools is too old (<12)
> setuptools_scm functionality is degraded



解决：下载 version > 12的setuptools，进行安装，如下：

```bash
python setup.py build
python setup.py install
```





#### QA, SNIMissingWarning, InsecurePlatformWarning

urllib3连接https站点时报InsecurePlatformWarning错误

> /usr/local/lib/python2.7/dist-packages/pip-18.1-py2.7.egg/pip/_vendor/urllib3/util/ssl_.py:369: SNIMissingWarning: An HTTPS request has been made, but the SNI (Server Name Indication) extension to TLS is not available on this platform. This may cause the server to present an incorrect TLS certificate, which can cause validation failures. You can upgrade to a newer version of Python to solve this. For more information, see https://urllib3.readthedocs.io/en/latest/advanced-usage.html#ssl-warnings
>   **SNIMissingWarning**
> /usr/local/lib/python2.7/dist-packages/pip-18.1-py2.7.egg/pip/_vendor/urllib3/util/ssl_.py:160: InsecurePlatformWarning: A true SSLContext object is not available. This prevents urllib3 from configuring SSL appropriately and may cause certain SSL connections to fail. You can upgrade to a newer version of Python to solve this. For more information, see https://urllib3.readthedocs.io/en/latest/advanced-usage.html#ssl-warnings
>   **InsecurePlatformWarning**



只是警告，不用管！！也可以如下：

> 法一：~~下载的pip-9.0.1安装包 基于 Python 2 >=2.7.9 or Python 3 >=3.4，但自己的python为2.7.3~~ 
> 法二：~~要不就换更低版本的pip，要不就更换更高版本的python语言。~~
> 法三：直接在源码中，将那一行代码注释了！！



#### QA, certificate verify failed

ssl连接问题之缺少CA证书，验证不了服务器的证书，问题返回信息如下：

> SSLError: HTTPSConnectionPool(host='pypi.org', port=443): Max retries exceeded with url: /pypi (Caused by **SSLError(SSLError(1, '_ssl.c:504: error:14090086:SSL routines:SSL3_GET_SERVER_CERTIFICATE:certificate verify failed'),)**)



解决1：`pip --cert /etc/ssl/certs/ install tushare`，完了之后报以下错误！

> SSLError('CA directories not supported in older Pythons',)

找到python用的ca-bundle.crt，然后赋值给--cert.  （参考，下面的certif module模块）



解决2, `pip --trusted-host pypi.org --trusted-host files.pythonhosted.org install tushare`



----

**certifi module**

certifi，python用于验证ssl/tls链接的根证书。

```python
>>> import certifi
>>> certifi.where()
'/usr/local/lib/python2.7/dist-packages/certifi/cacert.pem'
```

```bash
#-----------------------method 1
[global]
cert = /path/to/cacert.pem

#-----------------method2
export PIP_CERT=/usr/local/lib/python2.7/dist-packages/certifi/cacert.pem
echo export PIP_CERT=/etc/ssl/certs/ca-certificates.crt >> ~/.bashrc
```



----

总结：
要不在每次`pip install`时指定`--cert`, `--trusted-host`。
要不在`~/.pip/pip.conf`中指定`global/cert`。
要不在ssh会话中指定`PIP_CERT`。



#### QA, Cannot uninstall 'xx' ..

错误信息，如下：

> pip --trusted-host pypi.org --trusted-host files.pythonhosted.org install tushare
> **Cannot uninstall 'chardet'. It is a distutils installed project and thus we cannot accurately determine which files belong to it which would lead to only a partial uninstall.**

解决如下：

pip --trusted-host pypi.org --trusted-host files.pythonhosted.org install tushare `--ignore-installed chardet`



#### QA, pip不识别pip.conf

pip怎么没有用~/.pip/pip.conf，设置PIP_CONFIG_FILE也没用？

> 检查点：pip.conf配置对了吗？

<font color=red size=4>很大一部分估计是pip不识别pip.conf中的配置项</font>，所以加载了pip默认配置。
Refer, pip/config章节



#### QA，pip search获得http 301

pip search报301解析失败，如下：
pip search -v jieba

> ProtocolError: <ProtocolError for mirrors.aliyun.com/pypi/simple: 301 Moved Permanently>

curl -v http://mirrors.aliyun.com/pypi/simple
> Location: http://pub.mirrors.aliyun.com/pypi/simple/

vim ~/.pip/pip.conf

> ndex-url =  http://pub.mirrors.aliyun.com/pypi/simple/



总结：
aliyun把search功能关闭了（因为重定向的那个url也打不开），但不影响只要能install就行了！
要不就换源吧！





