[TOC]


## ReadMe



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

