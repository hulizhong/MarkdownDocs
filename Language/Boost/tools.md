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



