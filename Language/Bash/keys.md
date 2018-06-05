[TOC]



```bash
#空语句
pass

#自增自减
let "a++"  

#命令替换
# echo $(date)
Sat Jun 10 16:28:00 HKT 2017
# echo `date` 
Sat Jun 10 16:28:02 HKT 2017

#字符串判断防空
if [ "${stra}"x = "strb"x ] 

#巧用连接符判断成功与否
commands_succeed && succeed_process
commands_failed || failed_process

# 失败退出（这句语句告诉bash后面的语句 如果任何语句的执行结果不是true（0代表true）则应该退出）
set -e

```


## 各种括号

() vs {} 将多个命令组合在一起执行，相当于一个命令组。
> () 是在子shell中执行。  
>> $() ``的命令替换，注意$()跟单纯的()是不一样（还有()$呢）。
>
> {} 是在当前shell中执行；{}两边必须有空格；{}中最后一条指令必须以;结束。

test vs []
> 等价  
> [[]] 是[]的加强版。如果你遇到[]搞不定的，记得用[[]]。

(()) 专门用于数值计算。 
> (())内不能出现-ne之类的test关键词的用语，而用> < ++之类的数值运行符。   
> 使用 (( )) 时，不需要空格分隔各值和运算符，使用[]和[[ ]] 时需要用空格分隔各值和运算符包括’[‘’]’符（如 elif [ $a -gt $b ]; then）。


## 获取返回值
echo方式只能$()这样获取？ return方式只能$?这样获取吗？
```bash
return_ret()
{          
    return 1  
}          
           
echo_ret()
{          
    echo 3
}          
           
return_ret
aa=$?   #不能aa=$(return_ret)的方式获取。
bb=$(echo_ret)  #只能这样获取，命令替换。
echo $aa  #1
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


