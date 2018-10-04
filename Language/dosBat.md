[TOC]

- start
	- start "d:" 以d:目录命名，来复制一个cmd；
	- start d:  用文件夹管理器打开此目录；

- 变量
	- 定义 set 变量名=变量值
	- 引用 %变量名%

## ReadMe

<font color=red>只有在脚本文件中变量才有%%i，而在cmd窗口中变量则用%i</font>






## Grammar

-------
for
四种参数

```bash
rem https://www.cnblogs.com/yang-hao/p/6003923.html
```
```bash
@echo off
for %%i in (*.docx) do (
	echo %%i
)

# /l means i=1; i<=100; i=i+1
for /l %%i in (1, 1, 100) do (
	echo %%i;
)

pause
```


## KeyWords

## Command
dir遍历目录
```bash
dir /b dirname  :只显示文件名；
```

