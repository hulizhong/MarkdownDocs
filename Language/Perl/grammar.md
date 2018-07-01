## ReadMe
perl的语法讲解

## 字符串
字符串的常见处理
```perl
my $val = 'Rabin';
my $sz = length($val);
	计算长度
my $newVal = substr($string,offset,length);
	offset开始截取的位置；如果是负数，那么从右边开始截取；
	length省略，那么截取到最后一个字符；
	截取子串
```

## 输出
输出重定向
```perl
print "$var\n";
	默认向标准输出进行输出

print STDERR "$var\n";
	重定向到错误输出

open FP1 ">>log.txt" or die "cant open the log file";
print FP1 "$var\n";
close FP1;
	重定义输出到文件
```

## 特殊符号
```perl
\p{xx}
	xx代表一个unicode属性；
```
