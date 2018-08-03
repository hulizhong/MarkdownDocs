[TOC]

## ReadMe
介绍json语法、以及解析json的库；

## json语法
- json语法是 JavaScript 语法的子集。
	- 数据在名称/值对中：用key/value的形式，"key":"value"
	- 数据由逗号分隔
	- 花括号保存对象：{key/value1, key/value2}
	- 方括号保存数组：[obj1, obj2]


- json值可以是：
	- 数字（整数或浮点数）
	- 字符串（在双引号中）
	- 逻辑值（true 或 false）
	- 数组（在方括号中）
	- 对象（在花括号中）
	- null
	

## python中使用json库
### json库

- python class obj & jsonstr
	- python obj to json: json.dumps(obj.__dict__)
	- jsonobj to python obj: pythonobj.__dict__ = json.loads(jsonStr)

#### 序列化  pythonobj={dict}/[list]
jsonstr = json.dumps(pytonobj, skipkeys=False, ensure_ascii=True, check_circular=True, allow_nan=True, cls=None, indent=None, separators=None, encoding='utf-8', default=None, **kw)


#### 反序列化
pythoobj = json.loads(jsonstr, encoding=None, cls=None, object_hook=None, parse_float=None, parse_int=None, parse_constant=None, object_pairs_hook=None, **kw)

		
- jsonStr = json.dumps(pythonObj) is not json serializable

	pythonObj不符合json格式（如jsonObj中没有=只有:）
	



## c++中使用json库
### libjsoncpp
从文件加载
```cpp
ifstream ifs;
ifs.open("checkjson.json");
assert(ifs.is_open());

Json::Value root;
Json::Reader reader;
//读文件
if (!reader.parse(ifs, root, false)) {
	cerr << "parse failed \n";
	return;
}

//读内存
if (!reader.parse(json_data, json_data + sizeof(json_data), root)) {
	cerr << "json parse failed\n";
	return;
}
```

写入文件
```cpp
Json::Value root;
Json::FastWriter writer;
string json_file = writer.write(root);
```

从Json::Value中读出数据
```cpp
Json::Value value;
value["key"].size();
value["key"].asInt();
value["key"].asBool();
value["key"].asInt();
value["key"].asString();

Json::Value arr = value["array"]; //["ab", "bc"]
for (int = 0; i < arr.size(); i++) {
	std::cout << arr[i].asString() << std::endl;
}
Json::Value arr = value["array"]; //[{"key":"ab"}, {"key":"bc"}]
for (int = 0; i < arr.size(); i++) {
	std::cout << arr[i]["key"].asString() << std::endl;
}

value.isMember(char const*) const;  //没有则抛异常；

value["key"].toStyledString(); //将key这个节点的所有内容读作string.
```

往Json::Value中写入数据
```cpp
Json::Value value;
Json::Value arr;

//-----------------array下标赋值方式；
arr[0] = "ab";
arr[1] = "bc";
vlaue["array"] = arr; //["ab", "bc"]

//---------------array追加方式；
for () {
	Json::Value item. //construct 
	arr.append(item); //注意arr里面的每个元素格式可以不一致；
}
vlaue["array"] = arr;
```

```cpp
Json::Value::isMember(char const*) const
terminating with uncaught exception of type Json::LogicError: in Json::Value::find(key, end, found): requires objectValue or nullValue
Json::Value::find(char const*, char const*) const
Json::Value::isMember(char const*) const  //抛异常，如果没有这个成员；
```

