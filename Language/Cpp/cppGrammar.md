[TOC]

## ReadMe
描述c++语法信息；

[Refer website1](https://www.labri.fr/perso/nrougier/teaching/c++-crash-course/index.html)

**c++是类型安全的吗？**
不是，----因为不同类型的指针可以强转`reinterpret_cast`.



### c, cpp code style

c会有许多static、全局变量、全局函数； 

c++会多类；



### cpp verison

- c++0x
  - version 1. 1998 c++98
  - version 2. 2003 c++03
- c11
  - version 3. 2011 c++11
- c14
- c17
- c20
- ...



## Base

### data type

浮点数（没有无符号分类、可存储正、负数）：这些数据类型的确切大小取决于当前使用的计算机。唯一可以保证的是可以确认的是`double`至少与`float`一样大，`long double`至少与`double`一样大。

```cpp
float f;  //单精度，4字节、7位有效数字；
double d; //双精度，8字节、16位有效数字；
long double ld;  //高双精度，8字节、16位有效数字；
```



wstring

```cpp
#include <string>
std::wstring ws;
std::wcout << ws << std::endl;  //只有wcout,没有wendl.
```





## From C to C++

不同点概要如下：

| 不同点         | c                               | cpp                                                          |
| -------------- | ------------------------------- | ------------------------------------------------------------ |
| 变量           | --                              | bool                                                         |
|                | --                              | pair                                                         |
|                | --                              | 引用                                                         |
|                | 局部变量只能声明在函数最开始处  | --                                                           |
|                | ~~变长数组（c99的编译器即可）~~ | ~~同于c，另外vector是否也满足？~~                            |
|                | 类型强转(type)var               | 显示转换                                                     |
|                | static全局变量                  | 匿名空间                                                     |
|                | define常量                      | const常量（有类型、可类型安全检查）                          |
| 函数           | --                              | 重载（函数名后缀参数类型）                                   |
|                |                                 | 默认参数                                                     |
|                |                                 | 调用前必须有原型声明                                         |
| 改进           | macro                           | inline                                                       |
|                | malloc/free                     | new/delete                                                   |
|                | --                              | 命名空间                                                     |
| 异常机制       | 只有assert                      | try/catch                                                    |
| 面向对象       | 过程性                          | 过程性；<br />基于对象(类)，面向对象(继承、多态)；<br />泛型编程(template)； |
| 数据结构、算法 |                                 | STL                                                          |
|                |                                 |                                                              |

<font color=blue>面向对象</font>
封装：把客观事物抽象成类，在类内部维护属性及行为。
继承：实现、可视（窗体）、接口三种继承实现了功能复用。
多态：可以把不同子类的对象赋值给父类的指针、引用，让父类指针的一次调用展现出不同的形态。
  作用：在功能复用的同时实现差异化？---隐藏实现细节使代码模块化、易扩展？
所以面向对象开发，<font color=red>就是将事物抽象`概念化`，并进行分析、设计、实现</font>。



### Input/Output

as follow

```cpp
std::cout << "xx" << std::endl;
```



### New/Delete

They are "object-aware" so you'd better use them instead of  `malloc` and `free`.
`delete` does two things.  call the destructor and deallocates the memory.  ---所以有了显示调用析构函数。
`new`会调用构造函数；---malloc则不会对申请的空间进行对象初始化；且需要传入空间大小。

```cpp
int *a = new int;
int *a = new int(1); //初始化为1
delete a;

int *aa = new int[2]; //分配2个int.
delete []aa;
```



### Refernces

References are extremely非常 useful when used with function arguments since it saves the cost of copying parameters into the stack when calling the function.

引用目的：

> 主要用于在函数参数传递中，解决大块数据或对象的传递效率和空间不如意的问题。（不产生副本、提高传递效率）---只是目标变量的别名，本身不是一种数据类型，所以不占用空间。
> 用法：声明需初始化、不能再作为其它变量的引用、不能建立数组的引用。


使用引用的时机

> 流操作符<<和>>
> 赋值操作符=的返回值、参数
> 拷贝构造函数的参数
> .....其它情况都推荐使用引用

引用规则：

> 不能返回局部变量的引用。
> 原则上：不能返回函数内部new分配的内存的引用。--可能会内存泄漏。
> 可以返回类成员的引用，但最好是const。
> 引用与一些操作符的重载：流操作符<<和>>，赋值操作符号=。

因为常量和引用初始化必须赋值。那函数中的参数怎么解释（具有{}的函数定义哦）---形参不能算是定义吧！！！ 



**引用 vs 指针**
指针：是保存地址的一个变量，使用指针使代码可读性差。
引用：仅仅是变量的别名（不可空、不可赋值）。



### Default Parameters



### Namespaces

Namespace allows to group classes, functions and variable under a common scope name that can be referenced elsewhere在别处.

```cpp
//不像其他名字空间，未命名的名字空间的定义局部于一个特定的文件，不能跨越多个文本文件。
//替代c中的static全局变量.
namespace {
    void swap(int &a, int &b);
}
swap(a, b);

namespace alias=name;  //设置别名；
using namespace std;   //用std空间；
using std::cout;       //用std空间下的cout.
::a;                   //全局空间下的变量a.
```



### Overloading

Function overloading重载 refers to the possibility of creating multiple functions with the same name as long as they have different parameters (type and/or number).

It is not legal合法 to overload a function based on the return type (but you can do it [anyway](http://stackoverflow.com/questions/442026/function-overloading-by-return-type))

**重载解析**
根据函数调用处（的实参属性）从侯选重载函数集中找到最匹配的函数。



**重载 vs 覆盖 vs 隐藏？**

| 名称       | 效果                                       | 范围           | 特征                                                         |
| ---------- | ------------------------------------------ | -------------- | ------------------------------------------------------------ |
| 重载       |                                            | 相同范围内     | 函数名相同 + 参数不同；                                      |
| 覆盖、重写 | 指向子类的指针、引用<br />会调用子类的实现 | 基类与派生类中 | virtual + 函数名相同 + 参数相同；                            |
| 隐藏       | 基类的实现被隐藏                           | 基类与派生类中 | 函数名相同 + 参数不同；<br />函数名相同 + 参数相同 + nonVirtual； |

```cpp
class A {
public:
    void print() { std::cout << "A" << std::endl;}
};
class B : public A {
public:
    void print() { std::cout << "B" << std::endl;}  //与A.print构成隐藏。
}
A *pa = new B; pa->print();  //A，即只看指针的类型，而非指向的类型。
A a; a.print(); //A
B b; b.print(); //B
```



### Const & Inline

For macros, prefer the inline. as follow. 
c++中用内联函数替代宏。（**在编译时会在内联函数调用时进行展开，运行时不产生函数调用堆栈**。）

```cpp
#define SQUARE(X) X*x
int res = SQUARE(3+3);

int square(int x);
int inline square(int x) //1.inline只有放在“函数定义”时生效。
{
    //2.函数的内联请求可能被编译器拒绝。
    //3.函数被内联编译后，函数体直接扩展到调用的地方。
    //4.内联函数有地址吗？---应该没有（从源代码层看，有函数的结构，而在编译后，却不具备函数的性质）
    //5.内部不能有：循环、switch、递归调用自己。
    return x * x;
}
```



**const vs mutable vs volatile**
const（常量的）
  修饰变量指常量不能变。
  修饰类属性，针对于对象是常量、对类不是常量，所以不能类定义时进行初始化。（但c11可以）
  修饰类方法，不能改变对象的属性。
  修饰类对象、指针、引用，只能调用类的const方法。
mutable（可变的）. c++中用于突破const限制的，即使属性被声明为const.
  如const类方法可更改mutable的类属性。
  不能修饰static属性。
volatile（易变的），变量可能会被意想不到地改变，所以cpu读取值时从内存中拿而非寄存器。



### Mixing C and C++

c++ call c function, as follow.

```cpp
//法一：--------------------c++中引c
#ifdef __cplusplus
extern "C" { //连接声明：变量、函数按照c语言方式编译、连接。
#endif

#include "some-c-code.h"
    //函数名、参数排列的顺序可能不同
    //用c风格进行编译，因为some-c-code.so是用gcc编译出来的，即fun(int, int)不会变成funii。

#ifdef __cplusplus
}
#endif


//法二：------c中引c++（因为extern c不能用于.c中，会编译错误，所以放cpp.h中，在c.c中extern.）
//cpp.h
extern "C" int add(int x, int y);
//c.c
extern int add(int x, int y);  //not include cpp.h
```



疑问：`结构体`需要放在`extern "c"`中吗？
<font color=red>不需要，因为gcc/g++编译出来的二进制中看不到结构体的符号表</font>。---但这是为什么呢？是不是变量本身就代表中一段内存，而且变量本身有类型，即程序在运行时`按这样的类型`去解析`这段内存`，可有些语言变量在定义时没有类型，这又如何？ ---如python中`a=10`其实10才是变量，对应了一段内存，而a只是这段内存的引用名。

> 这个是正解. 编译器需要看到定义, 确定如何生成汇编码. 到了汇编码阶段, 汇编器哪还管你高级语言中的什么结构体不结构体啊, 那是编译器的事情. 这说明了, 编译型的高级语言, 往下转换成汇编代码之后, 会丢失一切高级语言中才有的概念/信息. 那么也就是说, 什么变量类型, 什么函数原型, 什么类啊函数指针啊访问控制啊(C++ 中的private...)之类的所有一切都是编译器向语言使用者维护或者说维持的概念, 到了汇编级, 一切都是扯淡. 所以C++编译器实现者真是些个苦逼啊...





### type cast

强制转换是将内存中一段结构以另一种不同类型的方式进行解读, 因此转换的空间必须与源空间一一对应。

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

#### static cast

相当于传统C中的强转，用来强迫隐式转换；
编译时检查，用于非多态的转换；（没有运行时类型检查来保证转换的安全性）

```cpp
//可转换：
	//继承类的上行转换（因为安全性），不适用于下行转换；
		//上行转换：是指派生类指针转换为基类指针类型; 安全的；
		//下行转换：是指基类指针类型转换为派生类类型; 不安全的；
	int/char; int/enum;
	void*;

//不能转换掉表达式中的const, volatile, __unaligned属性
```

#### dynamic cast

e为type的公有派生类（向上转）；e为type的基类（向下转）；e与type的类型一致；
RTTI运行时刻类型识别：用基类的指针、引用操作派生类的对象。是在运行时刻执行的。
  操作符一：dynamic_cast.
  操作符二：typeid.

<font color=red>dynamic_cast转换类指针时，需要虚函数</font>.

```cpp
dynamic_cast<type*>(e)   //type为一指针
	//指针类型转换失败，则结果为0
dynamic_cast<type&>(e)   //type为一左值
	//引用类型转换失败，则抛std::bad_cast异常
dynamic_cast<type&&>(e)  //type为一右值
	//引用类型转换失败，则抛std::bad_cast异常
```

#### const cast

去除、增加表达式的const, volatile属性；

```cpp
const int g = 20;
int *h = const_cast<int*>(&g);//去掉const常量const属性

const int g = 20;
int &h = const_cast<int &>(g);//去掉const引用const属性

const char *g = "hello";
char *h = const_cast<char *>(g);//去掉const指针const属性
```



#### reinterpret cast

强转（和C风格一致），意图执行低级转型，实际动作（及结果）可能取决于编辑器，这也就表示它不可移植；

```cpp
reinterpret_cast<type>(e);
	//type必须是一个指针、引用、算术类型、函数指针或者成员指针；
```







## Classes

class hold both <font color=red>data and funciton</font>. 
By default, all attributes and function of class are private. (The struct are public.)



类大小：
函数（virtaul, non-virtual, static, non-static）不占空间，数据占空间。
空类：占1个字节，因为每个类都是唯一的，所以编译器会隐式增加1字节的数据以区别。
虚函数：类开头会有个v(f)ptr指向vtable.
虚继承：会有个指向虚基表的指针vbptr。



### Constructors

a class has zero, one or more constructors.  -----0个怎么弄，显式delete?
不能被virtual化。（析构函数则只要类有继承关系，应该设成virtual.）

```cpp
class Foo
{
    Foo(); // 缺省构造函数
    Foo( const Foo& ); // 拷贝构造函数
    	//可不止一个拷贝构造，只要参数满足：第一个参数为Too&的变种、无其它参数或其它参数有默认值。
    	//不能用模板来定义，其它几个构造函数也一样。
    	//深浅拷贝：是否只复制了指针的值，而指针指向的内容没有进行拷贝。
    ~Foo(); // 析构函数
    Foo& operator=( const Foo& ); // 赋值运算符
    
    Foo* operator&(); // 取址运算符
    const Foo* operator&() const; // 取址运算符 const
};
Foo f;
Foo *f = new Foo();
Foo *f = new Foo;

//赋值运算符 vs 拷贝构造。
	//是否产生新的对象：赋值运算符，不会创建新的对象。拷贝构造，会创建新的对象。
	//类中有指针的，拷贝一般是赋值，赋值一般是引用。为了避免默认的错误可以把它俩注成private.
```



<font color=blue>explicit</font>
将构造函数声明为explicit，用以防止构造函数的隐式类型转换。（禁止编译器执行非预期（往往也不被期望）的类型转换）
  explicit关键字只能用于类内部的构造函数声明上，而不能用在类外部的函数定义上。
  c11可允许用在非构造函数的类型转换操作符上。
隐式转换：可以用单个实参来调用的构造函数定义了从形参类型到该类类型的一个隐式转换。



### Destructor

There can be only one destructor per class. It takes no argument and returns nothing.
<font color=gree>为了安全，尽可能的不要抛出异常。如果非抛不可，那么自己异常自己吃掉</font>！
  原因1，如果析构函数抛出异常，则异常点之后的程序不会执行，如果析构函数在异常点之后执行了某些必要的动作比如释放某些资源，则这些动作不会执行，会造成诸如资源泄漏的问题。
  原因2，通常异常发生时，c++的机制会调用已经构造对象的析构函数来释放资源，此时若析构函数本身也抛出异常，则前一个异常尚未处理，又有新的异常，会造成程序崩溃的问题。  

```cpp
class Foo
{
    ~Foo(void) {}
};
```



**虚析构函数**：
delete指向派生类的基类指针、引用时，只会调用基类的析构函数而不调用派生类的析构函数，即不会触发动态绑定。这样有可能会千万内存泄漏。



### Acess Control

public

protected

private



### Initialization List

Object's member should be initialized using initialization lists.  It's cheaper, better and faster.
但是，`const, 引用，基类构造函数`必须放在初始化列表中。

```cpp
class Foo
{
public:
	Foo(int v) : mV(v) {}    
private:
    int mV;
};
```



### Operator Overloading

as follow

```cpp
class Foo
{
    int mV;
    
public:
    Foo(int v) : mV(v) {}
    Foo operator+(const Foo &other) { //will create new object.
        return Foo(this->mV + other.mV);
    }
    Foo& operator=(const Foo&);
    Foo& operator[](int) const;
    type operator()();
    
    Foo& operator++/--(); //前置，还可友元；
    Foo& operator++/--(int); //后置，还可友元；
    
    //---------以下new,delete同于拷贝构造，只需要保证第一个参数，其它参数可以重载。
    void *operator new( size_t );
    	//自动属于类的静态static成员（没有this指针），因为new之前对象还没创建出来。
    void operator delete( void* );
    	//自动属于类的静态static成员，因为delete时对象已被销毁。
    void *operator new[]( size_t );
    void operator delete[]( void* );
    
    operator Foo2() { return val; }  //类型转换操作符重载。不需要写返回类型。
};
```



<font color=blue>重载如何定为友元、类成员？</font>
只有在`左操作数`是该类类型的对象时，才会考虑作为类成员的重载操作符。
哪些操作是必须是友元的？哪些必须是成员的？



### Friends

Friends are either functions or other classes that are granted授予 privileged access to a class.

不受`public, protected, private`的限制。



### nested class

嵌套类：在一个类A中定义另一个类B。
B受A的`public/protected/private`限制。B只对A可见。
A只能访问B的`public`部分，除非把A设置成B的`friend`.



### Others

**UML**,可参照`designPattern/golfPattern.md`

继承：is-a的关系。 
依赖：use-a的关系。
  `class A{}; class B{void fun(A a1, A *a2, A &a3);}` 
关联：两个类的一般性关系，包含强、弱关联。
聚合：has-a的弱关联；聚合类不对被聚合类负责。
  `class A{}; class B{A *a;};`
组合：contains-a的强关联；组合类对被组合类负责，生命周期相同。
  `class A{}; class B{A a;};`



**成员函数指针**

```cpp
void (A::*ptr)() = &A::nonstatic_fun;   //(a.*ptr)();  (a->*ptr)();
void (*ptr)() = &A::static_fun;

//&A::virtual_fun; 对于虚函数，这样只会取得其在虚函数表的偏移位置。（即编译期是不知道虚函数地址的）
```

类成员函数作为`回调函数`，那么只能是<font color=gree>`静态成员函数`</font>。



## Inheritance

Inheritance is done at the class definition level by specifying the base class and the type of inheritance.
是`"is a（是一种）"`的关系。

```cpp
class A : [public|protected|private] B
{
	//注意：A会继承所有B的资源（包含public, protected, private），但能否访问则依赖该属性、方法原来的访问权限 & 继承权限 取低(public>protected>private)值；
};
```

构造的时候：先调用基类的构造函数，再调用派生类的构造函数。
析构的时候：先调用派生类的析构函数，再调用基类的析构函数。



### Virtual Methods

A virtual function allows derived派生的 classes to replace the implementation provided by the base class (yes, it is not automatic...). Non virtual methods are resolved statically (at compile time) while virtual methods are resolved dynamically (at run time).

<font color=red>Make sure your destructor is virtual when you have derived class</font>.

```cpp
class Foo {
public:
    Foo( void );
    void method1( void );
    virtual void method2( void );
};

class Bar : public Foo {
public:
    Bar( void );
    void method1( void );  //同名同参，隐藏基类的实现。
    void method2( void );  //同名同参+virtual，覆盖基类的实现。
};

Foo *bar = new Bar();
bar->method1(); //will call Foo::method1
	//因为non-virtaul函数在编译时绑定，即只关心指针的类型而不关心指针所指向的类型。
bar->method2(); //will call Bar::method2
```



**Virtual Table, V-Table**
只有类中有virtual函数才会有vtable, vptr。vtable ptr为对象的最开始的4个字节，指向vtable.
vtable按虚函数声明的顺序记录了一个类的虚函数地址。
单继承：子类的虚函数续在父类的vtable后面；覆盖的保持位置，不过变为derive::xx而非base::xx.
多继承：在子类中按顺序有各自基类（如果有vtable）的vtable，子类的虚函数续在第1个基类的vtable中。
虚继承：会多一个虚基表指针vbptr（virtual base table pointer）。



### Abstract classes

You can define pure virtual method that prohibits阻止 the base object to be instantiated实例化. Derived classes need then to implement the virtual method.

```cpp
class Foo {
public:
    Foo(void);
    virtual void method(void) = 0;  //纯虚函数。
};

class Bar: public Foo {
public:
    Foo(void);
    void method(void) {};
};
```



### Multiple inheritance

A class may inherit from multiple base classes but you have to be careful:

```cpp
class Foo { protected: int data; };
class Bar1 : public Foo { /* ... */ };
class Bar2 : public Foo { /* ... */ };
class Bar3 : public Bar1, public Bar2 {
    void method( void ) {
       data = 1; // error: reference to ‘data’ is ambiguous
    }
};
```

In class Bar3, the data reference is ambiguous模糊不清的 since it could refer to Bar1::data or Bar2::data. This problem is referred as the **diamond菱形 problem**. You can eliminate消除 the problem by explicitely 明确地 specifying the data origin (e.g. Bar1::data) or by using virtual inheritance in Bar1 and Bar2.

菱形继承问题：函数用virtual继承来解决。

```cpp
class Foo { protected: int data; };
class Bar1 : virtual public Foo { /* ... */ };
class Bar2 : virtual public Foo { /* ... */ };
class Bar3 : public Bar1, public Bar2 { //not add virtual at here!!!
    void method( void ) {
       data = 1; // error: reference to ‘data’ is ambiguous
    }
};
```

**注意：**<font color=gree>在虚继承中，虚基类是由最终的派生类初始化的。而且编译器总是先调用虚基类的构造函数，再按照继承的顺序调用其他的构造函数</font>。





### virtual inheritance

虚拟继承又称作<font color=gree>共享继承</font>，这种共享其实也是编译期间实现的。
虚继承是解决C++`多重继承问题`的一种手段，如下：
从不同途径继承来的同一基类，会在子类中存在多份拷贝。这将存在两个问题：
	其一，浪费存储空间；
	第二，存在二义性问题；





## c++ class object model

c++类对象模型（<font color=red>new一个对象时， 只为类中nonstatic成员变量分配空间， 对象之间共享成员函数</font>），概要如下：

| 一级分类         | 二级分类          | 在类对象中 | 实现机制     |
| ---------------- | ----------------- | ---------- | ------------ |
| 方法（函数）     | nonstatic成员函数 | 不在       |              |
|                  | static成员函数    | 不在       |              |
|                  | virtual成员函数   | vptr指针   | 指向虚表vtbl |
| 属性（数据成员） | nonstatic属性     | 在         |              |
|                  | static属性        | 不在       |              |

virtual table：vtbl/vftbl存放着一堆指针，这些指针指向该类的每一个虚函数（按声明顺序）。
vtbl/vftbl的前面是一个指向type_info的指针，用以支持RTTI（Run Time Type Identification，运行时类型识别）。

vptr/vfptr：虚表指针，其位置由编译器决定，gcc/g++是把它放在对象的最前端，即对象地址就是vptr地址。
同类间的不同对象共享一个vtbl，但有各自的vptr.



继承引起的vptr, vtbl变更如下：

| 继承分类 | 细分     | 特征                                                         |
| -------- | -------- | ------------------------------------------------------------ |
| 单继承   |          | 子类与父类拥有各自的一个虚函数表；<br />若子类并无overwrite父类虚函数，用父类虚函数；<br />若子类重写（overwrite）了父类的虚函数，则子类虚函数将覆盖虚表中对应的父类虚函数；<br />若子声明了自己新的虚函数，则该虚函数地址将扩充到虚函数表最后； |
| 多继承   |          | 若子类新增虚函数，放在声明的第一个父类的虚函数表中；<br />若子类重写了父类的虚函数，其所涉及的所有父类的虚函数表都要改变；<br />内存布局中，父类按照其声明顺序排列； |
| 虚继承   | 简单继承 | 如果子类新增虚函数，会产生自己的vptr, vtbl，该vptr位于最前端；<br />子类有vbptr虚基类表指针，位于自身vptr之后（即相对对象偏移0或4字节）；<br />子类单独保留了父类的vptr, vtbl; |
|          | 菱形继承 |                                                              |

vbptr：指向虚基类表vbtbl，该表由多个条目组成：
  条目1：存储`vbptr所在地址`相对该类内存首地址的偏移量（0或-4）；
  条目2..n：依次为该类的从左到右的虚继承父类的内存地址相对于`vbptr所在地址`的偏移量；



## Exceptions

### try catch structure

catch exception as follow.

```cpp
try {
    //...
    throw exception;
}
catch (std::bad_alloc &e) {  //参数最少用引用，减少拷贝时间。
    cout << e.what() << endl;
}
catch (...) {
    cout << "catch all type exceptions." << endl;
}
```



### Standard exceptions

c++ standard exceptions.

```cpp
#include <exception>
class exception;
class bad_exception : public exception;

#include <stdexcept>
class logic_error : public exception;
//class domain_error : public logic_error;
//class invalid_argument : public logic_error;
//class length_error : public logic_error;
//class out_of_range : public logic_error;
class runtime_error : public exception
//class range_error : public runtime_error
//class overflow_error : public runtime_error
//class underflow_error : public runtime_error 
```





### Creating ur own exception

creating your own exception.

```cpp
#include <stdexcept>
clsss ExceptionMy : pulblic std::runtime_error
{
public:
	ExceptionMy : std::runtime_error("ExtionMy") {}
};
```





## Streams

iostream class provide the stream concept. 
iXXstream for input. oXXstream for output.



### iostream and ios

screen output and keyboard inputs.

```cpp
#include <iostream>

std::cout << "xx" << std::endl;
int i;
std::cin >> i;

double f = 3.1415926;
cout.unsetf(ios::floatfield);
cout.percision(5);
cout << f << endl;
cout.setf(ios::fixed, ios::floatfield);
cout << f << endl;
```



### Class Input/Output

using friends function.

```cpp
//友元函数实现法，调用形式为：cout << obj; 
class Foo
{
public:
    friend std::ostream& operator<< (std::ostream &output, Foo const &that) {
        return output << that.mValue;
    }
    friend std::istream& operator>> (std::istream &input, Foo const &that) {
        return input >> that.mValue;
    }
    
private:
    int mValue;
};
Foo f;
std::cout << f << std::endl;

//成员函数实现法，不常用，因为调用形式为 obj << cout;  //obj<<(cout)
class Foo
{
public:
    std::ostream& operator<< (std::ostream &output) {
        return output << this->mValue;
    }
};
```



### Working with files

file stream.

```cpp
#include <fstream>

std::ifstream ifs(filename);
std::ofstream ofs(filename);
```





### Working With Strings

string stream.

```cpp
#include <sstream>

std::istringstream istream;  //i string stream.
std::ostringstream ostream;
```



## Templates

Templates are special operators that specify that a class or a function is written for one or several generic types that are not yet known. 
The format for declaring function templates is

```cpp
template <typename T>
T foo1(void)
{
    //..Implement
}
int f1 = foo1<int>();

template <typename T>
class Foo3
{
    //..Implement
};
Foo3<int> f3;
```



**模板编译模式(template compilation model)**
包含模式 Inclusion Model：模板的定义放在.h文件中；使用时包含该头文件。  ---<font color=red>编译器只实现了这种</font>。
分离模式 Separation Model：模板的声明放在.h文件中，在.cpp文件中`export`实现；使用时包含头文件。



### Template parameters

there are three types.

```cpp
tempalte<typename T> T foo(void) {}  //was a type.类型

tempalte<int I> foo(void) {}  //no type.非类型

tempalte< template<typename T> > foo(void) {} //was another type.另外一个模板
```



### Template function

函数模板，如下：

```cpp
tempalte <typename T)
T max(T a, T b)
{
    return a > b ? a : b;
}

max<int>(4, 8);  //模板实例化(类型替换过程)template instantiation.
	//不指定，则靠模板实参推演（template argument deduction）。
	////显式指定（explicitly specify）模板实参。
max<float>(4.0, 8.0);
```



### Template class

类模板，如下：

```cpp
template <typename/class T, int size>  //T为类型参数。size为非类型参数。
class Foo
{
    T mV;
public:
    Foo(T t) : mV(t) {}
};

Foo<int> f(5);
```



### Template Instantiation

模板实例化，函数模板不是真正的函数定义，他只是如其名提供一个模板，模板只有在运行时才会生成相应的实例。（类模板也一样）

**implicit instantiation**
隐式实例化，只有当使用模板时，才会产生实例。（即运行期间）

```cpp
int main() {
    swap<int>(a, b);//它会在运行到这里的时候才生成相应的实例，很显然的影响效率。
}
```

**explicit instantiation**
显式实例化，相对于隐式实例化在编译期间就会产生实例，提高运行效率。

```cpp
template void swap<int>(int &a,int &b);  //显式实例化在编译期间就会生成实例。
template class classname<typename>;  //显式实例化类模板。
	//VS 特化：实例化只需要声明，不需要重新定义。
```



### Template specialization

模板特化、具体化、偏特化：针对所支持的泛型中的某一特殊类型进行不一样的处理，as follow.

```cpp
//--------------------------------------------------class template specialization.
template <typename T>
class Foo
{
    T _value;
public:
    Foo( T value ) : _value(value)
    {
        std::cout << "Generic constructor called" << std::endl;
    };
};

template <>         //空声明
class Foo<float>    //类名后，缀特化类型<type>
{
    float _value;
public:
    Foo( float value ) : _value(value)
    {
        std::cout << "Specialized constructor called" << std::endl;
    };
};

Foo<int> f1(5);
Foo<float> f2(5.0);  //will print "Specialized constructor called"


//-----------------------------------------------function template specialization.
template<>                    //空声明
int fun<float>(int a, int b)  //函数名后，缀特化类型<<type>
{
    //...
}
```



## Standard Template Library

### Containers

| 大类                                                         | 子分类         | 描述                                                         |
| ------------------------------------------------------------ | -------------- | ------------------------------------------------------------ |
| Sequence containers：拥有由单一类型元素组成的一个有序集合。  | vector.向量    |                                                              |
|                                                              | deque.双端队列 | 提供了与 vector 相同的行为，但是对于首元素的有效插入和删除提供了特殊的支持。 |
|                                                              | list.链表      |                                                              |
| Associative containers：支持查询一个元素是否存在，并且可以有效地获取元素。 | set            | 包含一个单一键值，有效支持关于元素是否存在的查询。           |
|                                                              | multiset       |                                                              |
|                                                              | map            | 键值对（字典）。                                             |
|                                                              | multimap       |                                                              |
|                                                              | bitset         |                                                              |
| Container adaptors.容器适配器                                | stack栈        | 后进先出                                                     |
|                                                              | queue队列      | 先进先出                                                     |
|                                                              | priority_queue | 优先级队列入队放在比它优先级低的前面。                       |



更多容器信息[参考这里](stl-container.md)



**Q, 容器的元素有什么特征？**
可默认构造；+ 可拷贝构造、拷贝赋值；+ 可析构；
关联容器，要求元素具有可比较性；
如此，`std::shared_ptr, std::unique_ptr`是可以作为容器元素的，但是`std::auto_ptr`是不行的。



### Iterators

can iterate over a container. 常被用于泛型算法中。
迭代器是一种检查容器内元素并遍历元素的数据类型。vs 指针？
  类似于指针，提供了对对象的间接访问。但迭代器功能更丰富。
  迭代器提供了对对象的访问方法，+ <font color=gree>定义了容器的范围（begin(), end()）</font>。

```cpp
std::map<std::string,int>::iterator iter;
for( iter=m.begin(); iter != m.end(); ++iter ) {
    std::cout << "map[" << iter->first << "] = " << iter->second << std::endl;
}

vector< type >::const_iterator iter = vec.begin();  //const.
vector< type >::reverse_iterator r_iter =  vec0.rbegin();  //反向
inserter();  //插入迭代器

```



**迭代器类型：**

| 类型           | 支持的操作                                  | 权限         |
| -------------- | ------------------------------------------- | ------------ |
| 输入迭代器     | p++, *p是右值, =, ==, !=                    | 可读、不可写 |
| 输出迭代器     | p++, *p是左值, =                            | 可写、不可读 |
| 前向迭代器     | 输入 + 输出迭代器                           | 可读、可写   |
| 双向迭代器     | p++, p--                                    | 可读、可写   |
| 随机访问迭代器 | p++, +=, -=, +, -, [], <位置对比, <=, >, >= | 可读、可写   |



**Q, 容器支持的迭代器类型？**
随机：vector, deque.
双向：list, set, multiset, map, multimap.
stack, queue, priority_queue，<font color=gree>不支持迭代器操作</font>。



### functional

函数对象，是一个类，重载了函数调用操作符()。常被用于泛型算法中，因为相对于函数指针具有：
  如果被重载的调用操作符是inline函数，则能够执行内联编译。
  函数对象可以拥有额外数据，提供其它一些属性。

```cpp
bool typename::operator()(const A& a) {...}

#include <functional> //包含了很多预定义的函数对象：算术函数对象、关系函数对象、逻辑函数对象。
```



**函数对象的适配器**
绑定器：将二元函数对象的一个实参绑定到一个特殊值上，将其转成一元函数对象。
  bind1st, 绑定到第1个实参。
  bind2nd, 
取反器：将函数对象的值翻转。
  not1, 翻转一元函数对象的真值。
  not2, 翻转二元。



### Algorithms

Algorithms from the STL offer fast, robust, tested and maintained code for a lot of standard operations on ranged elements. Don't reinvent the wheel !
这里所说的算法是`泛型算法`，已经脱离了对特定容器的依赖！----常用到迭代器、函数对象。



- 查找算法
  - adjacent_find(), binary_search(), count(), count_if(), 
    find(), find_end(), find_first_of(), find_if(),  search(), search_n() 
  - 二分查找
    - equal_range
    - lower_bound
    - upper_bound
- 排序（sorting）和通用整序（ordering）算法
- 删除和替换算法
  - copy(), copy_backwards(), iter_swap(), remove(), remove_copy(),
    remove_if(), remove_copy_if(), replace(), replace_copy(),
    replace_if(), replace_copy_if(), swap(), swap_range(), unique(),
    unique_copy
- 排列组合算法
- 算术算法
- 生成和异变算法
- 关系算法
  - equal(), includes(), lexicographical_compare(), max(), max_element(),
    min(), min_element(), mismatch() 
- 集合算法
  - set_union(), 并集。
  - set_intersection(), 交集。
  - set_difference(), 差集。
  - set_symmetric_difference()，对称差集。
- 堆算法
  - make_heap(), pop_heap(), push_heap(), sort_heap() 



排序算法不能用于：list、关联容器上。

更多stl算法信息[参考这里](stl-algorithm.md)



https://blog.csdn.net/csdn_chai/article/details/78041050

https://blog.csdn.net/csdn_chai/article/details/77600135

http://c.biancheng.net/cplus/80/



## The End

### undefined & unspecified behavior 

**Undefined behavior (UB)** means that the standard guarantees nothing about how the program should behave.  ---未定义。
  •解引用空指针或野指针。
  •访问未初始化内存，比如超出数组的边界或读取未初始化的本地变量。
  •删除相同的内存两次，或者更一般地删除一个野生指针。
  •算术错误，比如除以0。

**Unspecified未规定 (or implementation-defined) behavior** means that the standard requires the behavior to be well-defined, but leaves the definition up to the compiler implementation. ---定义了未实现，留给编译器。







