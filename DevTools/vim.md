[toc]

## vim



## gvim

vim是跨平台的，在win下叫vim.

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

## 常用指令
vim -o a b 打开a b于上下两窗口
vim -O a b 打开a b于左右两窗口
vim -d a b 等效于vimdiff a b

命令模式下各种指令
```bash
替换
:m,ns/str1/str2/c    m到n行
:%s/str1/str2/c    整个文件

:r !ls 读取后接指令执行结果到当前文本

设置
:set ignorecase  搜索大小写不敏感
:set hls 搜索高亮
:set nu  显示行号
```
