[TOC]



## ReadMe

数据结构之串；



## 串

定义： 0个或多个字符组成的 有限 序列，又叫字符串；

字符编码：在对应字符集中的序号；

 

字串的定位操作通常称作串的模式匹配。

​	包括BF、KMP、BM等

 



## 模式匹配

### BF

BF二维循环匹配算法



严蔚敏版c

int idx(string s, string t, int pos) {

​    i=pos; j=1;

​    while (i<=s[0] && j<=t[0]) {

​        if (s[i]==t[j])

​            {i++; j++}

​        else 

​            {i=i-j+2; j=1}

​    }

​    if (j>t[0]) return i-t[0];

​    else return 0;

}



### 朴素模式匹配



### KMP模式匹配

kmp算法通过一个O(m)的预处理，使匹配的复杂度降为O(n+m)。



### KMP的改进

