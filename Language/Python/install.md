[toc]

## ReadMe

## 各种版本安装
下载源码包、编译、安装
```python
# https://www.python.org/downloads/source/  
# 下载源码包；
# 解压：`tar -zxvf Python-2.5.6.tgz`  or `bzcat Python-2.5.6.tar.bz2 | tar -xf -`
# 运行指令："./configure", "make", "make install" commands to compile and install Python. 
```

## 特殊目录
/usr/share/pyshared/
> 这些库不管是哪个版本的python都能用

/usr/lib/python2.6(2.7/3)
> 这下面的库，只能是特定版本的python能用


## 环境变量
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
>> dist目录下的库，每个库对应一个同名的dist-info或者egg-info目录。应该分别对应两种对应的打包工具？？      


## 包的部署与管理
distutils是Python标准库的一部分。提供了打包、包安装的服务。 

setuptools distutils的增强版。   
引入包依赖管理；可为python包创建egg文件；提供easy\_install脚本来安装egg包。   
python的egg文件有点像java中的jar文件，是一个工程打包文件，便于安装部署。

pip easy\_install的增强版。  
无需要使用egg文件；
安装失败后不会出现只安装一部分的情况；
跟踪所有安装的包。  


### pip 
安装pip
```python
python setup.py install --record file.list
	#下载pip的tar.gz 进行
	#setup.py安装的办法，pip跟踪不了，只能依赖file.list来删除。  
```

~/.pip/pip.conf
```bash
trusted-host = pypi.douban.com
	#如果是ssl连接的话，就需要验证证书了。  
index-url = http://pypi.douban.com/simple
	#源  

https://pypi.python.org/pypi
	#pip的官网源
	#上面的库都只链接了最新的版本，如何下载历史版本？   
	#找到该项目的github地址，然后再在发布版本中去找（打了tag）。  
```

pip命令
```bash
pip install package_name  
pip install package_name==version  
	#如果安装成功，那么在最后会显示： Successfully installed markdoc 之类的。  
pip install --upgrade package_name==version  
pip uninstall package_name   
	#不能删除setup.py安装类的库（setup.py类型的只能用record方法）。  

list  #列出已安装包.   
show  #显示包详细信息。（包括版本、位置之类的）    
search  #搜索包，类似yum里的search.  
```


#### 问题
用pip安装软件失败，并显示以下内容？？
```bash
your setuptools is too old (<12)
setuptools_scm functionality is degraded

	#解决：下载 version > 12的setuptools，并解压
python setup.py build
python setup.py install
```

urllib3连接https站点时报InsecurePlatformWarning错误
```bash
#是下载的pip-9.0.1安装包 基于 Python 2 >=2.7.9 or Python 3 >=3.4，但自己的python为2.7.3  
#要不就换更低版本的pip，要不就更换更高版本的python语言。  
```


## 帮助系统
```bash
$ python 
$ import xx
$ help(xx)
$ help(xx.fun)
```
