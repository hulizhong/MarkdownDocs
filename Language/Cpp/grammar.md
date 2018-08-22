[TOC]

## ReadMe
描述c++语法信息；


## 类型转换
强制转换是将内存中一段代码以另一种不同类型的方式进行解读, 因此转换的空间必须与源空间一一对应。

> 结构体、类之间不能直接进行强制转换, 必须先转换成指针才可以进行结构体间的类型转换。



旧式强转
```cpp
(type-id)expression  //转换格式1
type-id(expression)  //转换格式2
```
c++中除了c中的强转（旧式强转）之外，还有如下几种（新式强转）
```cpp
static_cast<new_type>      (expression)
dynamic_cast<new_type>     (expression) 
const_cast<new_type>       (expression) 
reinterpret_cast<new_type> (expression)

//这些转换调用不需要std::打头、亦非c11的功能、亦不需要其它什么额外的头文件；
{
	B *b = new B(xx);
	A *a = std::dynamic_cast<A*>(b); //错误
		//会报error: expected unqualified-id .. std::dynamic_cast ..
	A *a = dynamic_cast<A*>(b);  //正确
}
```

旧式强转：**只用于基础的数据类型；**
新式强转：**主要运用于继承关系类间的强制转化；**

### static cast

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

### dynamic cast

e为type的公有派生类；e为type的基类；e与type的类型一致；

```cpp
dynamic_cast<type*>(e)   type为一指针
	指针类型转换失败，则结果为0
dynamic_cast<type&>(e)   type为一左值
	引用类型转换失败，则抛std::bad_cast异常
dynamic_cast<type&&>(e)  type为一右值
	引用类型转换失败，则抛std::bad_cast异常
```

### const cast

去除表达式的const, volatile属性；

```cpp
const int g = 20;
int *h = const_cast<int*>(&g);//去掉const常量const属性

const int g = 20;
int &h = const_cast<int &>(g);//去掉const引用const属性

const char *g = "hello";
char *h = const_cast<char *>(g);//去掉const指针const属性
```

### reinterpret cast

强转（和C风格一致），意图执行低级转型，实际动作（及结果）可能取决于编辑器，这也就表示它不可移植；

```cpp
reinterpret_cast<type>(e);
	//type必须是一个指针、引用、算术类型、函数指针或者成员指针；
```

