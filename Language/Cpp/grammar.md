[toc]

## ReadMe
描述c++语法信息；


## 类型转换
c++中除了c中的强转（旧式强转）之外
```cpp
(type-id)expression  //转换格式1
type-id(expression)  //转换格式2
```
还有如下几种（新式强转）
```cpp
static_cast<new_type>      (expression)
dynamic_cast<new_type>     (expression) 
const_cast<new_type>       (expression) 
reinterpret_cast<new_type> (expression)
```

旧式强转：**只用于基础的数据类型；**
新式强转：**主要运用于继承关系类间的强制转化；**

static cast
相当于传统C中的强转，用来强迫隐式转换；
编译时检查，用于非多态的转换；（没有运行时类型检查来保证转换的安全性）
```cpp
可转换：
	继承类的上行转换（安全），不适用于下行转换；
		上行转换：是指派生类指针转换为基类指针类型; 
		下行转换：是指基类指针类型转换为派生类类型; 
	int/char; int/enum;
	void*;

不能转换掉表达式中的const, volatile, __unaligned属性
```

dynamic cast
e为type的公有派生类；e为type的基类；e与type的类型一致；
```cpp
dynamic_cast<type*>(e)   type为一指针
	指针类型转换失败，则结果为0
dynamic_cast<type&>(e)   type为一左值
	引用类型转换失败，则抛std::bad_cast异常
dynamic_cast<type&&>(e)  type为一右值
	引用类型转换失败，则抛std::bad_cast异常
```

const cast
去除表达式的const, volatile属性；
```cpp
const int g = 20;
int *h = const_cast<int*>(&g);//去掉const常量const属性

const int g = 20;
int &h = const_cast<int &>(g);//去掉const引用const属性

const char *g = "hello";
char *h = const_cast<char *>(g);//去掉const指针const属性
```

reinterpret cast
意图执行低级转型，实际动作（及结果）可能取决于编辑器，这也就表示它不可移植；
```cpp
reinterpret_cast<type>(e);
	type必须是一个指针、引用、算术类型、函数指针或者成员指针；
```
