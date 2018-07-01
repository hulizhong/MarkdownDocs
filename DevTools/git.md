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
收录了如下使用方法
[对比](#对比)
[回滚](#回滚)
[tag](#tag)
[分支](#分支)
[](#)

### 对比
对比有如下指令：
```bash
git diff master --name-only
当前分支与master分支对比，但只显示文件名称

git diff master xx.cpp
当前分支与master分支的xx.cpp文件对比
```

### 回滚
代码回滚到过去某一个commit
```bash
git checkout newBranch
git reset commitID  
	但这样只能更改本地的分支、而不能更改远程的分支；
	需要删除远程老分支、并将本地新分支推上去；

git reset --hard commitID 强制回滚
```

### tag
tag的操作
```bash
git tag
git tag -l
	查看tag列表
git tag -v tagName
git show tagName
	查看对应tag的信息

git checkout -b branchName tagName
	拉tagName来建立分支branchName

git tag tagName
	在当前commit上打tag
git tag tagName commitID
	在commitID上打tagName

git push origin tagName
	提交远程tag

git tag -d tagName
	删除本地tag
it push origin :refs/tags/tagName
	删除远程tagName
```

### 分支
分支的操作
```bash
git checkout branchName
	从远程分支建立本地分支
git checkout -b newBranchName
	从当前所在分支拷贝建立新分支

git branch -d branchName
git branch -D branchName
	删除

git branch -m oldName newName
git branch -M oldName newName
	更改branch名称；M带强制功效；

```



