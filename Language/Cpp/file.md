[TOC]


## ReadMe



## 特殊句柄

包含标准输入、输出、错误输出，。。



--------

**标准输入、输出、错误**

一套是标准的，为`int`类型，直接被应用于系统调用 ，如下：

```cpp
#include <unistd.h>

#define STDIN_FILENO 0 /* Standard input. */
#define STDOUT_FILENO 1 /* Standard output. */
#define STDERR_FILENO 2 /* Standard error output. */
	//用于open/read/write/close()系列系统调用中。
```



一套是较高级的，为`FILE*`类型，带有buffer的高级IO操作。

```cpp
stdin;
stdout;
stderr;
	//用于fopen/fread/fwrite/fclose()系列；
```







## ifstream
```cpp
;
```

### 一次读取整个文件
读到char数组
```cpp
int length;
std::ifstream t;
t.open("file.txt");
t.seekg(0, std::ios::end);
length = t.tellg();
t.seekg(0, std::ios::beg);
buffer = new char[length];
t.read(buffer, length);
t.close();
```

读到string
```cpp
#include <fstream>  
#include <streambuf>  
  
std::ifstream t("file.txt");  
std::string str((std::istreambuf_iterator<char>(t)), std::istreambuf_iterator<char>()); 
```

读到string
```cpp
#include <fstream>  
#include <sstream>  

std::ifstream t("file.txt");  
std::stringstream buffer;  
buffer << t.rdbuf();  
std::string contents(buffer.str());
```

## ofstream
```cpp
#include <fstream>

std::string outName;
std::ofstream ofs(outName.c_str());  //参数不能为std::string.
ofs << content;
ofs.close();
```

