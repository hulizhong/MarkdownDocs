[TOC]

## ReadMe
利用Lambda表达式，可以方便的定义和创建匿名函数。

## 表达式
```cpp
[capture list] (params list) mutable exception-> return type { function body } (params2 list)
	capture list   //捕获外部变量列表
	params list    //形参列表（注意：不能有默认参数；必带参数名；不支持可变参数）
	mutable        //指示可以在函数体内修改按值捕获到的变量；（是不是有点多余，那还不好按引用捕获）
	exception      //异常设定
	return type    //返回类型
	function body  //函数体
	params2 list   //给形参传值，相当于是实参；
```

一个demo
```cpp
#include <vector>
#include <algorithm>
using namespace std;

bool cmp(int a, int b) {
    return  a < b;
}

int main()
{
    vector<int> myvec{ 3, 2, 5, 7, 3, 2 };
    vector<int> lbvec(myvec);

	//旧式做法
    sort(myvec.begin(), myvec.end(), cmp);
	//Lambda表达式
    sort(lbvec.begin(), lbvec.end(), [](int a, int b) -> bool { return a < b; });
}
```

## 捕获方式
### 值捕获
被捕获变量的值通过值拷贝方式传入lambda；

### 引用捕获
被捕获变量的值通过引用方式传入lambda；

### 隐式捕获
在捕获列表中不填写特定的变量名称，而是根据函数体中的代码来推断需要捕获哪些变量；
```cpp
int main()
{
    int a = 123;

	// 值捕获
    auto f = [=] { cout << a << endl; };

	// 引用捕获
    auto f = [&] { cout << a << endl; };
    a = 321;
    f(); // 输出：321
}
```

### 混合方式

[=, &x]	变量x以引用形式捕获，其余变量以传值形式捕获
[&, x]	变量x以值的形式捕获，其余变量以引用形式捕获

### 总结
[]       不捕获任何外部变量
[var, …] 默认以值得形式捕获指定的多个外部变量（用逗号分隔），如果引用捕获，需要显示声明（使用&说明符）
[this]	 以值的形式捕获this指针
[=]	     以值的形式捕获所有外部变量
[&]	     以引用形式捕获所有外部变量
[=, &x]  变量x以引用形式捕获，其余变量以传值形式捕获
[&, x]   变量x以值的形式捕获，其余变量以引用形式捕获

