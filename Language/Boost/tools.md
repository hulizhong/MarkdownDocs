[TOC]

## ReadMe
boost工具类；


## XML Parser
相关类如下：
```cpp
boost::property_tree::ptree pt;
boost::property_tree::wptree pt;

boost::property_tree::read_xml
boost::property_tree::xml_parser::xml_writer_settings
boost::property_tree::xml_parser::write_xml

//https://blog.csdn.net/qq_33053671/article/details/78795636
```

xml解析类
```cpp
#include <boost/property_tree/ptree.hpp> //还有wstring对应的wptree
#include <boost/property_tree/xml_parser.hpp>

boost::property_tree::ptree pt;
boost::property_tree::read_xml("conf.xml", pt);  //write_xml()
std::string name = pt.get<std::string>("root.students.name");
std::string name = pt.get<std::string>("root.students.name", "defaultName"); //赋上默认值不会导致没有值时crash.
```


## Regex
正则匹配
```cpp
#include "boost/regex.hpp"

boost::regex reg("//d{3}([a-zA-Z]+).(//d{2}|N/A)//s//1");
boost::regex_match(str, reg);
```

## 转码
conv类来实现宽、窄字符串的转换；
```cpp
#include <boost/locale/encoding_utf.hpp>
#include <string>

using boost::locale::conv::utf_to_utf;

std::wstring utf8_to_wstring(const std::string& str)
{
    return utf_to_utf<wchar_t>(str.c_str(), str.c_str() + str.size());
}

std::string wstring_to_utf8(const std::wstring& str)
{
    return utf_to_utf<char>(str.c_str(), str.c_str() + str.size());
} 
```



## uuid

源码目录架构   

```bash
▾ uuid/                 
  ▸ detail/             
    name_generator.hpp  
    nil_generator.hpp   
    random_generator.hpp
    seed_rng.hpp        
    sha1.hpp            
    string_generator.hpp
    uuid.hpp            
    uuid_generators.hpp 
    uuid_io.hpp         
    uuid_serialize.hpp  
```

生成uuid

```cpp
#include <boost/uuid/uuid.hpp>
#include <boost/uuid/uuid_io.hpp>
#include <boost/uuid/uuid_generators.hpp>
 
boost::uuids::uuid a_uuid = boost::uuids::random_generator()();
	// 这里是两个() ，因为这里是调用的 () 的运算符重载.
const string str_uuid = boost::uuids::to_string(a_uuid);
	//uuid转string.
```



## program_options

程序参数解析类，主要有以下三个插件：

> options_description，选项描述器，定义可以接收哪些参数。
> parse_command_line，选项分析器，分析用户输出的参数。
> variables_map，选项存储器，将分析好的参数存储在其中。

boost::program_options::value 不能用？std::wstring

```cpp
#include <boost/program_options.hpp>
	//typedef basic_option<char> option;
	//typedef basic_option<wchar_t> woption;
namespace po = boost::program_options;

po::options_description desc;
po::variables_map vm;

// Step, 添加选项
desc.add_options()
    ("from", po::value<std::string>(&from)->required(), "from data")
    ("count", po::value<unsigned int>(&count)->default_value(count), "count data");

// Step, 解析参数、存储在vm中
po::store(po::parse_command_line(argc, argv, desc), vm);

// Step, 让vm更新所有的外部变量
po::notify(vm);

// Step, 使用；
if(vm.count("count")) {
    std::string xx = vm["count"].as<string>();
    std::string xx = vm["count"].asString();
}
```



## lexical_cast

lexical cast为数值之间的转换提供了一揽子方案
可以顶替`std::`的[诸多函数](c11.md#标准库扩展#数字、字符串转换)；

```cpp
#include <boost/lexical_cast.hpp>
string s = "123";  
try {
    int a = boost::lexical_cast<int>(s); 
    //如果转换发生了意外，会抛出一个bad_lexical_cast异常；
}
catch(boost::bad_lexical_cast& e) {
    //...
}
```

## 特殊功能类

收录一些功能类，我们只需要继承这些类就能实现特定功能，如下：
[noncopyable](#boost::noncopyable)

### boost::noncopyable

禁止拷贝类
主要看点：拷贝构造函数、=重载符 置为私有的属性。如下：  

```cpp
class noncopyable
{
 protected: //为什么是protected？
    noncopyable() {}
    ~noncopyable() {}
 private:  // emphasize the following members are private
    noncopyable( const noncopyable& ); 
    const noncopyable& operator=( const noncopyable& );
};
```



