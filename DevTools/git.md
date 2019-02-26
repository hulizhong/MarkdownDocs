[TOC]

## 概念、理念

## 安装、配置

包含在Linux, Win平台下的安装、配置；

### linux Platform

#### 命令补全

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

### windows Platform

#### ssh key

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

#### 配置文件路径

git的配置文件（在安装目录下）
> gitInstallPath\mingw64\etc\gitconfig，注意linux下是个隐藏文件.gitconfig.
> gitInstallPath\etc\

#### 问题-乱码

git bash乱码

> 右键 options
> 菜单 text
> Local, Charcter set设置
>> https://blog.csdn.net/u013068377/article/details/52168434
> 



### 配置

可在git bash窗口进行设置、亦可在配置文件中进行设置；

git bash窗口中，如下：

```bash
git config --global user.name "root"  
git config --list
	#查看当前各种配置项；
git config user.name
	#查看user.name配置项；
```



配置文件中，如下：

```bash
[user]                       
    name=rabinhu
    email=hulizhong@yeah.net
[color]
    status=auto
    branch=auto
    interactive=auto
    diff=auto
[core]
    editor=vim
[push]
    default=simple #定义push的行为；
    #nothing - push操作无效，除非显式指定远程分支。
    #upstream - push当前分支到它的upstream分支上（这一项其实用于经常从本地分支push/pull到同一远程仓库的情景，这种模式叫做central workflow）。
    #simple - 和upstream是相似的，只有一点不同，simple必须保证本地分支和它的远程upstream分支同名，否则会拒绝push操作。
    #current - push当前分支到远程同名分支，如果远程同名分支不存在则自动创建同名分支。
    #matching - push所有本地和远程两端都存在的同名分支。
```



## 使用

收录了如下使用方法

### 分支的建立、删除、更名

```bash
git checkout branchName
	#切换到A，如果本地没有那么试图从远程建立本地分支
git checkout -b newBranchName
	#从当前所在分支拷贝建立新分支

git branch -d branchName
git branch -D branchName
	#删除

git branch -m oldName newName
git branch -M oldName newName
	#更改branch名称；M带强制功效；
```

### 日志查看

```bash
git log 
git log -p
	#详细信息
git log --oneline
	#一个提供只有一行的显示
```

### 分支对比

```bash
git diff master --name-only
	#当前分支与master分支对比，但只显示文件名称
git diff master xx.cpp
	#当前分支与master分支的xx.cpp文件对比
```

代码回滚到过去某一个commit
```bash
git checkout newBranch
git reset commitID  
	#但这样只能更改本地的分支、而不能更改远程的分支；
	#需要删除远程老分支、并将本地新分支推上去；
git reset --hard commitID 强制回滚
```

### 分支合并
分支的合并有两种方法：
rebase法，是将当前修改续在上游分支的尾部
一般用法为把master rebase到个人分支；
```bash
git rebase 上游分支 [下游分支（如果当前分支作为下游分支，则可省略）]

#在A上修改文件，并提交本地（不提交到远端）
git checkout A
git add xx.h
git commit  #注意：只是本地提交

#切换到上游分支B，并获取最新代码
git checkout B
git pull

#切换到下游分支A、并rebase上游分支B
git checkout A
git rebase B           #rebase B到本地分支上来；（B作为上游）
git push origin A:A
git push -f origin A:A  #如果远程A在这次rebase之前没有合到B上，那么需要强推

#合并步骤，git pull --rebase
git pull
	#git fetch; git merge fetch_head
git pull --rebase
	#git fetch; git rebase fetch_head
```

merge法，貌似是按照时间节点合并进去；
一般用法为把个人分支merge到master上来；（切不可把个人分支rebase到master上）

```bash
git checkout A  #切换到A上
git merge B     #把B合到A上来
git push origin A:A  #提交远程A
```

### tag
tag一般用于发布，分为轻量标签（指向提交对象的引用）、附注标签（为仓库中的一个独立对象，并不是引用），推荐使用附注标签；
```bash
git tag
git tag -l
	#查看tag列表
git tag -v tagName
git show tagName
	#查看对应tag的信息

git checkout -b branchName tagName
	#拉tagName来建立分支branchName
git checkout tagname  #切记没有-b
	#切换到tagname空分支，可以再接着空分支创建出有名分支。

git tag tagName
	#在当前commit上打轻量tag
git tag tagName commitID
	#在commitID上打轻量tagName
git tag -a v0.1.2 -m "0.1.2版本" commitid
	#创建附注标签，因为相当于是一次提交（有新对象产生）所以会有提交注释；

#-----git push 默认不会把标签推送至远端
git push origin tagName
	#提交单个tag到远程服务器，如git push origin v0.1.2
git push origin –tags
	#将本地所有标签一次性提交到git服务器  


git tag -d tagName
	#删除本地tag
it push origin :refs/tags/tagName
	#删除远程tagName
```



### 提交日志修改

提交到远程的（push）是不能修改的；

- 修改最后一次commit comment.
  - `git commit --amend`
- 修改非最后一次commit comment.
  - `git rebase -i HEAD~3`
  - 选择待修改那行的`pick`改成`edit`
  - `git commit --amend`
  - `git rebase --continue`



## Git理论

### HEAD版本

表示当前版本，上一个版本就是HEAD^，上上一个版本就是HEAD^^，当然往上100个版本写100个^比较容易数不过来，所以写成HEAD~100



### 空分支

```sh
root@mddoc# git branch 
* (no branch)
  master
```

git checkout -b newBranchName

> 可以让空分支变成有名的分支。