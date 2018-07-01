[toc]

## ReadMe
boost工具类；


## xml parser
xml解析类
```cpp
#include <boost/property_tree/ptree.hpp>
#include <boost/property_tree/xml_parser.hpp>

boost::property_tree::ptree pt;
boost::property_tree::read_xml("conf.xml", pt);
std::string name = pt.get<std::string>("root.students.name");
```

## regex
正则匹配
```cpp
#include "boost/regex.hpp"

boost::regex reg("//d{3}([a-zA-Z]+).(//d{2}|N/A)//s//1");
boost::regex_match(str, reg);
```
