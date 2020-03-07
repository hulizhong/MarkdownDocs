[TOC]

## ReadMe
**Q. 什么时候用boost ?**

- 是std的补充（TR1, C++11, C++14很多特性都是从boost库来的）。
- 什么时候用boost；
  - std不够用了；
  - gcc太老不支持std=c++11/14之类的；
- 学习方法
  - 它是很多库的集合，只需要做到用哪些库看哪些库。
  - 源码很复杂，会用就行；（到后期发展需要或者好奇心可以看下源码）



**Q. 需要知道boost的哪些特性？**

好，那么请看以下章节。



## Boost源码结构

Boost源码概要结构（一般一个模块内`boost下的ipp`与`libs下的cpp`不共存！）

> boost/boost/asio/\*.h  申明的头文件
> boost/boost/asio/impl/\*.ipp 模板的实现文件（用于将模板的声明与实现分离）
> boost/libs/filesystem/src/\*.cpp  实现文件



- RAII & Memory Management
    - smartPointers
    - pointerContainer
    - scopeExit
    - pool
- String Handling
    - stringAlgorithms
    - lexicalCast
    - format
    - regex
    - xpressive
    - tokenizer
    - spirit
- Containers
    - multiIndex
    - bimap
    - array
    - unordered
    - circularBuffer
    - heap
    - intrusive
    - multiArray
    - container
- Data Structures
    - optional
    - tuple
    - any
    - variant
    - propertyTree
    - dynamicBitset
    - tribool
    - compressedPair

- [V. Algorithms](https://theboostcpplibraries.com/algorithms)
    - [29. Boost.Algorithm](https://theboostcpplibraries.com/boost.algorithm)
    - [30. Boost.Range](https://theboostcpplibraries.com/boost.range)
    - [31. Boost.Graph](https://theboostcpplibraries.com/boost.graph)
- [VI. Communication](https://theboostcpplibraries.com/communication)
    - [32. Boost.Asio](https://theboostcpplibraries.com/boost.asio)
    - [33. Boost.Interprocess](https://theboostcpplibraries.com/boost.interprocess)
- [VII. Streams and Files](https://theboostcpplibraries.com/streams-and-files)
    - [34. Boost.IOStreams](https://theboostcpplibraries.com/boost.iostreams)
    - [35. Boost.Filesystem](https://theboostcpplibraries.com/boost.filesystem)
- [VIII. Time](https://theboostcpplibraries.com/time)
    - [36. Boost.DateTime](https://theboostcpplibraries.com/boost.datetime)
    - [37. Boost.Chrono](https://theboostcpplibraries.com/boost.chrono)
    - [38. Boost.Timer](https://theboostcpplibraries.com/boost.timer)
- [IX. Functional Programming](https://theboostcpplibraries.com/functional-programming)
    - [39. Boost.Phoenix](https://theboostcpplibraries.com/boost.phoenix)
    - [40. Boost.Function](https://theboostcpplibraries.com/boost.function)
    - [41. Boost.Bind](https://theboostcpplibraries.com/boost.bind)
    - [42. Boost.Ref](https://theboostcpplibraries.com/boost.ref)
    - [43. Boost.Lambda](https://theboostcpplibraries.com/boost.lambda)
- [X. Parallel Programming](https://theboostcpplibraries.com/parallel-programming)
    - [44. Boost.Thread](https://theboostcpplibraries.com/boost.thread)
    - [45. Boost.Atomic](https://theboostcpplibraries.com/boost.atomic)
    - [46. Boost.Lockfree](https://theboostcpplibraries.com/boost.lockfree)
    - [47. Boost.MPI](https://theboostcpplibraries.com/boost.mpi)
- [XI. Generic Programming](https://theboostcpplibraries.com/generic-programming)
    - [48. Boost.TypeTraits](https://theboostcpplibraries.com/boost.typetraits)
    - [49. Boost.EnableIf](https://theboostcpplibraries.com/boost.enableif)
    - [50. Boost.Fusion](https://theboostcpplibraries.com/boost.fusion)
- [XII. Language Extensions](https://theboostcpplibraries.com/language-extensions)
    - [51. Boost.Coroutine](https://theboostcpplibraries.com/boost.coroutine)
    - [52. Boost.Foreach](https://theboostcpplibraries.com/boost.foreach)
    - [53. Boost.Parameter](https://theboostcpplibraries.com/boost.parameter)
    - [54. Boost.Conversion](https://theboostcpplibraries.com/boost.conversion)
- [XIII. Error Handling](https://theboostcpplibraries.com/error-handling)
    - [55. Boost.System](https://theboostcpplibraries.com/boost.system)
    - [56. Boost.Exception](https://theboostcpplibraries.com/boost.exception)
- [XIV. Number Handling](https://theboostcpplibraries.com/number-handling)
    - [57. Boost.Integer](https://theboostcpplibraries.com/boost.integer)
    - [58. Boost.Accumulators](https://theboostcpplibraries.com/boost.accumulators)
    - [59. Boost.MinMax](https://theboostcpplibraries.com/boost.minmax)
    - [60. Boost.Random](https://theboostcpplibraries.com/boost.random)
    - [61. Boost.NumericConversion](https://theboostcpplibraries.com/boost.numeric_conversion)
- [XV. Application Libraries](https://theboostcpplibraries.com/application-libraries)
    - [62. Boost.Log](https://theboostcpplibraries.com/boost.log)
    - [63. Boost.ProgramOptions](https://theboostcpplibraries.com/boost.program_options)
    - [64. Boost.Serialization](https://theboostcpplibraries.com/boost.serialization)
    - [65. Boost.Uuid](https://theboostcpplibraries.com/boost.uuid)
- [XVI. Design Patterns](https://theboostcpplibraries.com/design-patterns)
    - [66. Boost.Flyweight](https://theboostcpplibraries.com/boost.flyweight)
    - [67. Boost.Signals2](https://theboostcpplibraries.com/boost.signals2)
    - [68. Boost.MetaStateMachine](https://theboostcpplibraries.com/boost.msm)
- [XVII. Other Libraries](https://theboostcpplibraries.com/other-libraries)
    - [69. Boost.Utility](https://theboostcpplibraries.com/boost.utility)
    - [70. Boost.Assign](https://theboostcpplibraries.com/boost.assign)
    - [71. Boost.Swap](https://theboostcpplibraries.com/boost.swap)
    - [72. Boost.Operators](https://theboostcpplibraries.com/boost.operators)




## RAII
构造函数时资源申请、析构函数时资源释放；

### boost\_scope\_exit
退出时调用{}中的内容；

demo
```cpp
#include <boost/scope_exit.hpp>
int fun()
{
	int fd = open("tst.md", "rb");
	//1. fd此时为按值传递（捕获），亦可按引用传递（捕获）；多个参数之间用逗号分隔；
	//2. 引用捕获时有些编译器不支持指针的引用捕获；
	//3. 类成员函数捕获类对象本身用this_；
	BOOST_SCOPE_EXIT(fd){
		//当程序离开fun()函数时，执行exit,exit_end之间的代码；
		close(fd);
	}BOOST_SCOPE_EXIT_END
}
```



## SmartPointer



## Thread & Lock



## Lambda & Function & Bind

lambda即匿名函数，代表着**就地声明、就地调用**的思想。
function代表着**延后调用**的思想。

### lambda

一个 lambda 表达式通常也称为匿名函数(*unnamed function*)。它在需要的时 候进行声明和定义，即就地进行。

```cpp
#include "boost/lambda/lambda.hpp"

boost::lambda::_1; //参数占位符；为lambda表达式指出了延后的参数。
```



**T. bind**

在lambda操作符不够用时就用绑定吧；

```cpp
#include <boost/lambda/bind.hpp>
```





### function

通常，我们要把函数对象传入算法，把 lambda 表达式存入另一个延后调用的函数，名为 `boost::function`

```cpp
#include <boost/function.hpp>

boost::function<bool (int, int)> f;
if (f) { //f可以判空；
    f(1, 2);
}
```



**T. bind**

需要封装function、包装参数时，就用bind吧；

```cpp
#include <boost/bind.hpp>
```

