[TOC]


## ReadMe



## 特殊句柄

包含标准输入、输出、错误输出，。。



## fsteam

包含ifsteam, ofsteam.

```cpp
// ----------------------------------------文件指针相关
// ifsteam
tellg(); //当前指针位置。
seekg(pos); //绝对移动
seekg(pos, refer); //相对移动

// ofsteam
tellp();  //当前指针位置。
seekp(pos);
seekp(pos, refer);

std::ios::beg = 0; //相对于文件头
std::ios::cur = 1; //相对于当前位置
std::ios::end = 1; //相对于文件尾
```





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



## 二进制读写文件

疑问点rabinw?

```cpp
ifs.clear();
ifs.seekg(flvPos); //移动读标之前为什么要clear.（前提场景：读标之前已经讲到文件末尾了！！）
```

二进制操作文件有以下注意点：

- 读写只能用`read, write`，其它的概不能用如`<<, put, >>, get, getline`。
- 数据对比，只能按字节来对比，不能字符串对比。

```cpp
std::ifstream ifs;
ifs.open(argv[1], std::ios::binary);
char n; //byte data.
ifs.read(&n, 1);

long sz = 5;
std::string dt(sz, '\0');
ifs.read(&dt[0], sz);
if (dt[0]=='0' && dt[1]=='\r' && dt[2]=='\n' && dt[3]=='\r' && dt[4]=='\n') {
    //字符串匹配是不行了，只能按字节对比。
}
```

