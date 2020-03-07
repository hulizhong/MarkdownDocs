[TOC]

## ReadMe

Linux Bash相关语法；

http://www.runoob.com/linux/linux-shell-include-file.html)



```bash
set -e
	# 失败退出
	#这句语句告诉bash后面的语句 如果任何语句的执行结果不是true（0代表true）则应该退出

pass
	#空语句

if [ "${stra}"x = "strb"x ] 
	#字符串判断防空
	
commands_succeed && succeed_process
commands_failed || failed_process
	#巧用连接符判断成功与否
```



## 变量

shell变量分为：局部变量、环境变量

> 局部变量：当前shell实例可用的变量；
> 环境变量：所有的程序，包含shell启动的程序都能访问的变量；

```bash
var=value #变量定义不能有空格； 变量的二次赋值不能用$var，直接用变量名则可。
unset var #变量的删除

#变量的使用（）
$var
${var} #增加变量名边界的定位
```

/etc/bashrc vs /etc/profile中的环境变量关系？

​	

### 整数
请看如下demo.

```bash
let "a++"
	#自增自减
```



### 字符串
字符串形式
```bash
str='i am string.'
	#原样输出；
str="i am string, eq ${var}.\n"
	#可包含变量、转义字符、单引号；
${str:3:5}
	#从str的第3个字符开始，总共截取5个字符。
```



各种操作

```bash
$str="abc"
$str"def"
	#拼接：直接相连接即可，无需要连接符；
${#str}
	#字符串长度
${str:1:4}
	#字符串截取
`expr index "${str}" u`
	#查找字串
```



### 数组

只支持一维数组

```bash
array=(1 2 3 4)
	#值之间的隔离用 空格或者换行。



#-----------------------------各种操作
${array[0]}
	#单个元素获取

${array_name[@]}
${array_name[*]}
	#获取全部元素

${#array_name[@]}
${#array_name[*]}
	#数组的长度：

lengthn=${#array_name[n]}
	#单个元素的长度
```



## 分支控制

if

```bash
if [ $a -eq $b ]  # -ne/eq
then
	echo $a "==" $b
elif [ $a -gt $b ]; then  #要将分支控制的助词如then, do提前一句，那么需要在前一句话后加分号;再加此助词。
	echo $a ">" $b
else
	echo $a "<" $b
fi
```



for

```bash
#for ((i=0; i<5; i++))
#for (( i=0; i<5; i++ ))
for i in 0 1 2 3 4 #只能空格为间隔，加,则会解释成数据
do
  	echo 'hello', $i
done
```



while

```bash
#while(($a<=3)) ##cant use -le
while [ $a -le 3 ]
do
	   echo $a
    let "a++"
done
```





### 死循环

```bash
#while [ true/1/false ]  用方括号一定要两边有间隔！！
#while(true) 
for ((;;))
do
	if [ $? -ne 0 ]; then  #上一条命令执行错误；
		break [n]  # 跳出第n层循环，默认n为1；
		contine    # 跳过后面处理，继续本层循环；
	fi
done
```



## 函数

形式如下  

```bash
# 不带任何参数
# /bin/sh不能带function关键字，/bin/bash则可有可无function
[ function ] funname [()]
{
    action;
	# 如果不加，将以最后一条命令运行结果，作为返回值。
    [return int;] #return后跟数值n(0-255)
}
```



### 参数

参数名、意义
```bash
$#
	#传递到脚本的参数个数
$*
$@
	#传递到脚本的所有参数，但最好用$@，请看下面的demo.

$$
	#脚本运行的当前进程ID号
$!
	#后台运行的最后一个进程的ID号	
$-
	#显示Shell使用的当前选项，与set命令功能相同。
$?
	#显示最后命令的退出状态。0表示没有错误，其他任何值表明有错误。

$0
	#Shell脚本名
$1
	#第1个参数，。。
```

```bash 
#$* vs $@     不加‘’时两都一样，加‘’时$*为一整体，$@为原始参数
function showDollarVar()
{
	echo '-------------$*' 
	for arg in $*
	do
		echo $arg
	done
	echo '-------------$@'
	for arg in $@
	do
		echo $arg
	done
	
	echo '-------------"$*"'
	for arg in "$*"
	do
		echo $arg
	done
	echo '-------------"$@"'
	for arg in "$@"
	do
		echo $arg
	done
}

showDollarVar "hua jiang hu" zhi
root@shellLang# ./grammar.sh 
-------------$* #以一个单字符串的形式显示所有向脚本传递的参数。
hua
jiang
hu
zhi
-------------$@ #同于没有加“”的$*
hua
jiang
hu
zhi
-------------"$*" #全部参数视为一个整体
hua jiang hu zhi
-------------"$@" #原始参数一个个解析出来（默认为空格）
hua jiang hu
zhi
```





### 获取返回值

两种方式返回：return & echo .

```bash
return_ret()
{          
    return 1  
}
return_ret
aa=$?   #return返回的值，只能用$?获取；（但是只能返回0-255之间的整数）
echo $aa  #1
           
echo_ret()
{          
    echo 3
}          
bb=$(echo_ret)  #echo返回的值，只能用命令替换来获取；
echo $bb  #3
```

```bash
############## $?只能返回0-255之间的数字。高于它将会被除以256。
returnValue()               
{                           
    size=256  #### bash里面变量默认为全局
    return $size            
}                           
returnValue                 
echo "returnValue", $?, $size  ###0, 256
```





## 输入、输出

- echo

	```bash
	echo –e “…\x” 开启转义
	echo –e “…\c” 不换行
	```

- printf 比echo更强大，移植性更好；

	```bash
	printf  format-string  [arguments...]
	```

- 几个特殊是输入、输出文件
	- 0是标准输入（STDIN），1 是标准输出（STDOUT），2 是标准错误输出（STDERR）。  
	- /dev/null 是一个特殊的文件，写入到它的内容都会被丢弃；如果尝试从该文件读取内容，那么什么也读不到。  


### 输入出重定向

- 输入：n <& m	将输入文件 m 和 n 合并。
   输出：n >& m	将输出文件 m 和 n 合并。   
    标记块：<< tag	将开始标记 tag 和结束标记 tag 之间的内容作为输入。  

   看下面把标准输出、错误输出打印到file.log里
   ```bash
   $ command > file.log 2>&1  #只能将2写在前面
   $ command $> file.log
   ```



## 运算符
- 算术运算符
+、-、*、/、%、=、==、!=
> 原生的bash不支持，得借助（let,expr,..）之类的表达式计算工具。
```bash
let a=a+1
`expr $a + $b`  #注意：表达式与运行符号之间必须要空格。
# 注意：乘号*需要转义。
```

- 字符串运算符
> =判等、!=  
> -z长度是否为0、-n长度是否不为0  
> str是否不为空（这点如下）。  
```cpp
if [ $string ]; then 
	echo ‘string not empty’
fi
```

- 布尔运算符
> !非、-o或、-a与

- 逻辑运算符
> &&逻辑与、||逻辑或


### test测试
- 数值测试：
	* -eq等于则为真、-ne
	* -gt大于, -lt
	* -ge大于等于、-le
- 字符串测试：
	* =, !=, 
	* -z –n
- 文件测试：
	- -e文件存在
	- -f文件存在且为普通文件
	* -r, -w, -x, -s, -d, -c, -b

	> 文件测试运算符：[ -r $file ]
	-r file	检测文件是否可读，如果是，则返回 true。  
	-w file	检测文件是否可写，如果是，则返回 true。  
	-x file	检测文件是否可执行，如果是，则返回 true。  
	-s file	检测文件是否为空（文件大小是否大于0），不为空返回 true。  
	-e file	检测文件（包括目录）是否存在，如果是，则返回 true。  
	-d file	检测文件是否是目录，如果是，则返回 true。  
	-f file	检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true。  
	-b file	检测文件是否是块设备文件，如果是，则返回 true。  
	-c file	检测文件是否是字符设备文件，如果是，则返回 true。  
	-k file	检测文件是否设置了粘着位(Sticky Bit)，如果是，则返回 true。  
	-p file	检测文件是否是有名管道，如果是，则返回 true。  
	-u file	检测文件是否设置了 SUID 位，如果是，则返回 true。  
	-g file	检测文件是否设置了 SGID 位，如果是，则返回 true。  



## 高级话题

### Shell运行环境
登录到linux就有个登录shell
> 可以在此登录shell中执行语句，也可以执行shell脚本（开个子shell执行shell语句）；

- 与环境有关系的指令：
  - export var  在任何子shell中均有该变量的拷贝

    > 子shell继承了父shell， 但子shell不能影响父shell的变量值；  

  - source var  当前shell 中执行命令；





### 命令替换

命令替换

```bash
# echo $(date)
Sat Jun 10 16:28:00 HKT 2017
# echo `date` 
Sat Jun 10 16:28:02 HKT 2017
```





### 各种括号

() vs {} 将多个命令组合在一起执行，相当于一个命令组。

> () 是在子shell中执行。  
>
> > $() ``的命令替换，注意$()跟单纯的()是不一样（还有()$呢）。
>
> {} 是在当前shell中执行；{}两边必须有空格；{}中最后一条指令必须以;结束。

test vs []

> 等价  
> [[]] 是[]的加强版。如果你遇到[]搞不定的，记得用[[]]。

(()) 专门用于数值计算。 

> (())内不能出现-ne之类的test关键词的用语，而用> < ++之类的数值运行符。   
> 使用 (( )) 时，不需要空格分隔各值和运算符，使用[]和[[ ]] 时需要用空格分隔各值和运算符包括’[‘’]’符（如 elif [ $a -gt $b ]; then）。





## Tips demo

### directories traversal

目录遍历了解一下。

```bash
function dirlist() {
    for element in `ls $1`
    do
        fp=$1"/"$element
        if [ -d $fp ]
        then
            dirlist $fp
        else
            echo $fp
        fi
    done
}
dirlist ./
```



### file operation

按行读取文件，<font color=red size=4>只能读unix格式的文件，dos格式不能这样弄！！</font>

```bash
num=1
cat bk.list | while read line
do
    echo -e "downloadd -------$line, $num.html-----1\n"
    let "num++"
done
```



