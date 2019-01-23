[TOC]



## ReadMe

记录c/cpp编译错误集。



## undefined symbols

```bash
Undefined symbols for architecture x86_64:
  "aa::bb::cc::create(std::1::basic_string<char, std::1::char_traits<char>, std::__1::allocator<char> > const&)", referenced from:
      xx::yy() in xx.cpp.o
ld: symbol(s) not found for architecture x86_64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
```

重新编译就好了，（删除cmake编译目录！）



## const

非const引用const

```cpp
void fun(std::string &str) {}
fun("hehe");

//eroor info.
//error: invalid initialization of non-const reference of type ‘std::string& {aka std::basic_string<char>&}’ from an rvalue of type ‘const char*’
void fun(const std::string &str) {}
```



## no type

```cpp
#include <vector>
std::vector vec;
//error info.
//no type named 'vector' in namespace 'std'; did you mean 'hecto'?
```



实现与声明不符合

```cpp
void fun(int a, int b);
bool fun(int a, int b) {}
//error info.
//error: return type of out-of-line definition of 'xxx' differs from that in the declaration ...
//note: previous declaration is here ...
```

