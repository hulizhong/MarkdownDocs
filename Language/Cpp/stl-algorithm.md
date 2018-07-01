[toc]

## ReadMe
stl算法库；

## lower upper bound
```cpp
lower_bound(firstIt, lastIt, value&, compareFun);
	在[first, last]有序容器中找到一个位置可以存储value；（第一个大于等于某个数的位置）
upper_bound(firstIt, endIt, value, compareFun);
	是指找到第一个大于某个数的位置
```
