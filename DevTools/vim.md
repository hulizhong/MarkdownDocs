[TOC]

## vim

called vim on linux platform.

### 4种模式

vim一共有4个模式：

- 正常模式 (Normal-mode) ，其它模式到此按1/2次`esc`
- 插入模式 (Insert-mode)，正常到此按`i/I/a/A`
- 命令模式 (Command-mode)，正常到此按`:`
- 可视模式 (Visual-mode)，正常到此按`v/V/ctrl+v`

### 配置

如下，只收录了一些初始配置

```bash
:set ignorecase  #搜索大小写不敏感
:set hls #搜索高亮
	set hlsearch
:set nu  #显示行号

#自动缩进
set autoindent
set cindent

#Tab键的宽度
set tabstop=4
#统一缩进为4
set softtabstop=4  #在 insert 模式下，一个 tab 键按下后，展示成几个空格。
set shiftwidth=4   #选中行后左移<右移>的空格数.
#用空格代替制表符
set expandtab
#在行和段开始处使用制表符
set smarttab

```



### 指令集

vim -o a b 打开a b于上下两窗口
vim -O a b 打开a b于左右两窗口
vim -d a b 等效于vimdiff a b

命令模式下各种指令

```bash
#替换
:m,ns/str1/str2/c    #m到n行
:%s/str1/str2/c      #整个文件

:r !ls #读取后接指令执行结果到当前文本

:g!/ERROR/d   #删除非ERROR行，即只保留ERROR行。

:retab   #按照expandtab, noexpandtab的指示将tab替换成空格、空格替换成tab.

:e #重载文件
```



### 奇异文件

**了解**：dos vs unix格式的文件。

```bash
:set ff?
	#查看file format，格式可为dos/unix.
:set ff=unix

file bk.list
	#bk.list: ASCII text, with CRLF line terminators
```

unix/dos格式的文件，会影响文件的按行读取！！---refer. language/bash.md#Tips demo/file operation.





### 神操作

**No 1**，查看文本内`\r\n`符号。

```bash
:set list
```



**No 2**，字数统计。

选中需要统计的字符，按`g`，再按`ctrl+g`。



**No 3**，输入unicode码。

**查看字符的unicode码**。------4位16进制！
在非输入状态下，移动光标到某一字符，按<font color=red>`ga`</font>，就会显示该字符对应10，16，8进制的值。

> <s>  115,  Hex 73,  Octal 163

**输入unicode字符**。
在输入状态，<font color=red>`ctrl+v`</font>，再按<font color=red>`u`</font>，此时输入对应字符的unicode码！





## gvim

called gvim on win platform.

- 配置文件
	> .vimrc ==> _vimrc
	> 注释"打头

- 插件
	- 插件直接解压到gvim安装目录即可；
	- 相同目录不会覆盖，而是新增；

- win上的修改
	- ctrl+v不是选择，而是弄成了粘贴；
		> 找到安装目录 vim80/mswin.vim 删除以下快捷键映射； 
		>> " CTRL-V and SHIFT-Insert are Pastemap 
		>> <C-V> "+gP
	- 待续



## vimdiff

vimdiff == vim -d a b

- 合并
    - dp (diff put)
    - do (diff got)

- 对比
    - :diffupdate == :diffu 更改后再次刷新对比

- 指令加a
    - wa 两边都写入
    - qa 两边都退出


## ctags

usage as follow.

```bash
ctags * -R
ctrl + ]
ctrl + t
```


