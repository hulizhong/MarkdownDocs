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



## no type



std::vector xx;

>  no type named 'vector' in namespace 'std'; did you mean 'hecto'?



实现与声明不符合

> error: return type of out-of-line definition of 'xxx' differs from that in the declaration ...
> note: previous declaration is here ...

