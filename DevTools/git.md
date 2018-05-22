[toc]


## 概念、理念

## linux git
### 命令补全
从git工程里把补全脚本放到环境变量即可；
```bash
git clone https://github.com/git/git
cd contrib/completion
scp git-completion.bash Lizhong@172.22.48.48:~/
mv ~/git-completion.bash ~/.git-completion.bash 
vim ~/[.bashrc|.bash_profile]
	#添加如下语句
	[ -f ~/.git-completion.bash ] && source ~/.git-completion.bash
```


## windows git
### ssh key
位置
> c:/Users/hulizhong/.ssh/id_rsa

多个id_rsa配置
> 可以配置哪个域名用哪个id_rsa，在~/.ssh/目录下，新建config文件；
>> Host xx.com
>> HostName xx.com
>> User xx
>> IdentityFile path\.ssh\id_rsa-xx
>> 
>> Host yy.com
>> HostName yy.com
>> User yy
>> IdentityFile path\.ssh\id_rsa-yy
>

### 配置
git的配置文件（在安装目录下）
> gitInstallPath\mingw64\etc\gitconfig
> gitInstallPath\etc\


git bash乱码
> 右键 options
> 菜单 text
> Local, Charcter set设置
>> https://blog.csdn.net/u013068377/article/details/52168434
> 


## 使用


