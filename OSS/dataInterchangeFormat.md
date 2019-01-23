[TOC]



## ReadMe

数据交换格式相关。
为了保证信息数据传递的高效性，通信双方一定会将数据做成某种参与者都理解的格式。同理在**系统间交换的数据也必须要有特定的格式，目标系统才能识别传递过去的数据**。

两个系统间通信的过程中：发送系统要`序列化`发送的数据、接收方要`反序列化`接收到的数据。



## xml



## json



### c++ jsoncpp

几个大的结构，如下：

```cpp
Json::Value v; //json中任何一种数据的抽象，可为如下：
	//signed int, unsigned int.
	//double, string, bool, null
	//在序列表
	//一个json对象

Json::Reader r;  //反序列化，即将string解析成json对象。
r.parse(); //对文件、字符串进行解析。

Json::Writer w;  //序列化抽象类，将json对象写到string对象中。
Json::FastWriter fw;  //实现类，无格式写入string.
Json::StyledWriter sw;  //实现类，有格式写入string.
std::string Json::Value::toStyledString();
```

一些具体的api.

```cpp
bool Json::Value::isMember(const char * key) const;  //是否有这个key.
 	//root.isMember("anything-not-exist"); //true，针对map对象，同于c++会得到一个null的value.
	//root["anything-not-exist"].isNull(); //false, 这应该是true呀，按照上面的理论？？？
bool Json::Value::isNull() const; //判断value是否为null.
bool Json::Value::empty() const; //判断vlaue是否为空。
bool Json::Value::size() const; //得到value大小。

Value Json::Value::removeMember(const char* key); //删除key.
```





## yaml



## protoBuf



