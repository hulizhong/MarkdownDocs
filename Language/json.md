[TOC]

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


