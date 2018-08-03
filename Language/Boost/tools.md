[TOC]

## ReadMe
boost工具类；


## xml parser
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


## regex
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

