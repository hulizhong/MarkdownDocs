[TOC]

## ReadMe
描述c++语法信息；

[Refer website1](https://www.labri.fr/perso/nrougier/teaching/c++-crash-course/index.html)





## From C to C++

### Input/Output

as follow

```cpp
std::cout << "xx" << std::endl;
```



### New/Delete

They are "object-aware" so you'd better use them instead of  malloc and free.
delete does two things.  call the destructor and deallocates the memory.

```cpp
int *a = new int;
delete a;

int *aa = new int[2];
delete []aa;
```



### Refernces

References are extremely useful when used with function arguments since it saves the cost of copying parameters into the stack when calling the function.

引用目的：

> 主要用于在函数参数传递中，解决大块数据或对象的传递效率和空间不如意的问题。（不产生副本、提高传递效率）


使用引用的时机

> 流操作符<<和>>
> 赋值操作符=的返回值、参数
> 拷贝构造函数的参数
> .....其它情况都推荐使用引用

引用规则：

> 不能返回局部变量的引用。
> 原则上：不能返回函数内部new分配的内存的引用。
> 可以返回类成员的引用，但最好是const。
> 引用与一些操作符的重载：流操作符<<和>>，赋值操作符号=。

因为常量和引用初始化必须赋值。那函数中的参数怎么解释（具有{}的函数定义哦）！！！ 



### Default Parameters



### Namespaces

Namespace allows to group classes, functions and variable under a common scope name that can be referenced elsewhere.



### Overloading

Function overloading refers to the possibility of creating multiple functions with the same name as long as they have different parameters (type and/or number).

It is not legal to overload a function based on the return type (but you can do it [anyway](http://stackoverflow.com/questions/442026/function-overloading-by-return-type))



### Const & Inline

For macros, prefer the inline. as follow.

```cpp
#define SQUARE(X) X*x
int res = SQUARE(3+3);

int inline square(int x)
{
    return x * x;
}
```



### Mixing C and C++

c++ call c function, as follow.

```cpp
#ifdef __cplusplus
extern "C" {
#endif

#include "some-c-code.h"

#ifdef __cplusplus
}
#endif
```





### type cast

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

#### static cast

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

#### dynamic cast

e为type的公有派生类；e为type的基类；e与type的类型一致；

```cpp
dynamic_cast<type*>(e)   type为一指针
	指针类型转换失败，则结果为0
dynamic_cast<type&>(e)   type为一左值
	引用类型转换失败，则抛std::bad_cast异常
dynamic_cast<type&&>(e)  type为一右值
	引用类型转换失败，则抛std::bad_cast异常
```

#### const cast

去除表达式的const, volatile属性；

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
By default, all attributes adn function of class are private. (The struct are public.)

### Constructors

a class has zero, one or more constructors.

```cpp
class Foo
{
    Foo() {}  //no return type.
};
```





### Destructor

There can be only one destructor per class. It takes no argument and returns nothing.

```cpp
class Foo
{
    ~Foo(void) {}
};
```



### Acess Control

public

protected

private



### Initialization List

Object's member should be initialized using initialization lists.  It's cheaper, better and faster.

```cpp
class Foo
{
public:
	Foo(int v) : mV(v) {}    
private:
    int v;
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
    Foo operator+ (const Foo &other) { //will create new object.
        return Foo(this->mV + other.mV);
    }
};
```



### Friends

Friends are either functions or other classes that are granted privileged access to a class.



## Inheritance

Inheritance is done at the class definition level by specifying the base class and the type of inheritance.

```cpp
class A : [public|protected|private] B
{
};
```



### Virtual Methods

A virtual function allows derived classes to replace the implementation provided by the base class (yes, it is not automatic...). Non virtual methods are resolved statically (at compile time) while virtual methods are resolved dynamically (at run time).

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
    void method1( void );
    void method2( void );
};

Foo *bar = new Bar();
bar->method1(); //will call Foo::method1
bar->method2(); //will call Bar::method2
```



### Abstract classes

You can define pure virtual method that prohibits the base object to be instantiated. Derived classes need then to implement the virtual method.

```cpp
class Foo {
public:
    Foo( void );
    virtual void method( void ) = 0;
};

class Bar: public Foo {
public:
    Foo( void );
    void method( void ) { };
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

In class Bar3, the data reference is ambiguous since it could refer to Bar1::data or Bar2::data. This problem is referred as the **diamond problem**. You can eliminate the problem by explicitely specifying the data origin (e.g. Bar1::data) or by using virtual inheritance in Bar1 and Bar2.



## Exceptions

### try cathe structure

catch exception as follow.

```cpp
try {
    //...
}
catch (std::bad_alloc e) {
    cout << e.what() << endl;
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

std::istringstream istream;
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



### Template parameters

there are three types.

```cpp
tempalte<typename T> T foo(void) {}  //was a type.

tempalte<int I> foo(void) {}  //no type.

tempalte< template<typename T> > foo(void) {} //was another type.
```



### Template function

as follow.

```cpp
tempalte <typename T)
T max(T a, T b)
{
    return a > b ? a : b;
}

max<int>(4, 8);
max<float>(4.0, 8.0);
```



### Template class

as follow.

```cpp
template <typename T>
class Foo
{
    T mV;
public:
    Foo(T t) : mV(t) {}
};

Foo<int> f(5);
```



### Template specialization

场景：针对所支持的泛型中的某一特殊类型进行不一样的处理，as follow.

```cpp
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

template <>
class Foo<float>
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
```



## Standard Template Library

### Containers

Sequence containers.

vector

deque

list



----

Container adaptors

stack

queue

priority_queue



--------

Associative containers

set

multiset

map

multimap

bitset





### Iterators

can iterate over a container, as follow.

```cpp
std::map<std::string,int>::iterator iter;
for( iter=m.begin(); iter != m.end(); ++iter ) {
    std::cout << "map[" << iter->first << "] = " << iter->second << std::endl;
}
```





### Algorithms

Algorithms from the STL offer fast, robust, tested and maintained code for a lot of standard operations on ranged elements. Don't reinvent the wheel !

