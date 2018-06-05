[TOC]

> http://blog.csdn.net/rankun1/article/details/51247777



- NSI脚本只要有一个OutFile指令和一个Section就可以了。就是下面这个样子：

	```bash
	; nsi脚本用var关键字来定义变量，使用$来引用变量。
	; 注意：变量是全局的并且是大小写敏感的。
	
	;NSIS编译出来的程序默认的风格很Win 98。要使用MUI2库来生成一个比较XP风格的安装包。
	!include "MUI2.nsh"
	
	OutFile "Min.exe"
	
	Section
		...
	SectionEnd
	
	
	; 自定义函数
	Function GetNetFrameworkVersion
		..
	FunctionEnd
	
	; 回调函数
	Function .onInit
	..
	FunctionEnd

	```



- 指令
	- RMDir 只能删除空目录，可以加 -r
	- File ：Adds file(s) to be extracted to the current output path ($OUTDIR).
		- 所以在file之前特别注意OutPath值；
	

- Section

- Function
	- 自定义函数；
		- call functionname
	- 回调函数；
		- 安装逻辑回调函数
			- .onGUIInit
			- .onInit
			- .onInstFailed
			- .onInstSuccess
			- .onGUIEnd
			- .onMouseOverSection
			- .onRebootFailed
			- .onSelChange
			- .onUserAbort
			- .onVerifyInstDir
		- 卸载逻辑回调函数
			- un.onGUIInit
			- un.onInit
			- un.onUninstFailed
			- un.onUninstSuccess
			- un.onGUIEnd
			- un.onRebootFailed
			- un.onUserAbort

		
