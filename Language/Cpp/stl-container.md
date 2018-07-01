[toc]

## readme

- 容器分类
	- 顺序容器
		- vector
			- 一段连续的内存地址，基于数组的实现。
		- list
			- 非连续的内存，基于链表实现。
		- deque
			- 与vector类似，但是对于首元素提供删除和插入的双向支持。

	- 关联容器（就是有key-value形式的，其中set的key,value一样）
		- map：key-value形式的。
			- 元素默认升序
		- multimap：
			- key值可以相同。
		- set：
			- 元素默认升序
			- key,value一样
		- multiset：
			- key值可以相同。

容器类自动申请和释放内存，我们无需new和delete操作。


## vector
### 增、删、改、查
定义
```cpp
vector<int> vec1;    //默认初始化，vec1为空
vector<int> vec2(vec1);  //使用vec1初始化vec2
vector<int> vec3(vec1.begin(),vec1.end());//使用vec1初始化vec2
vector<int> vec4(10);    //10个值为0的元素
vector<int> vec5(10,4);  //10个值为4的元素
```

增、删、改、查
```cpp
vector v1, v2;

insert()
v1.insert(v1.end(),5,3);    //从vec1.back位置插入5个值为3的元素
v2.assign(v1.begin(), v1.end())  //根据v1给v2赋值
push_back() //在Vector最后添加一个元素（参数为要插入的值）
pop_back(); //删除末尾元素

cout << vec1[0] << endl;   //取得第一个元素

iterator erase( iterator loc );                    //要删除元素的迭代器
iterator erase( iterator start, iterator end);    //要删除的第一个元素的迭代器，要删除的第二个元素的迭代器
clear() //清空所有元素

empty() //判断Vector是否为空（返回true时为空）
size()  //返回元素个数；
```

遍历
```cpp
//c11
for (auto val : valList) {
	cout << val << endl; //这样迭代出来的是vector中的元素类型，而非迭代器
}
```

### 编译告警
vector使用超出了其空间
```bash
std::vector<int> va(-1);  初始化为-1个空间？
libc++abi.dylib: terminating with uncaught exception of type std::length_error: vector
critical handler: signal 6 is triggered.
```

## deque
### 增、删、改、查
定义
```cpp
std::deque dq;
```

增、删、改、查
```cpp
//与vector类似，支持随机访问和快速插入和删除
//与vector不同，deque还支持从开始端插入数据：push_front()
push_front()
```

遍历
```cpp
//no data
```


## list
### 增、删、改、查
定义和初始化
```cpp
list<int> lst1;          //创建空list
list<int> lst2(3);       //创建含有三个元素的list
list<int> lst3(3,2);     //创建含有三个元素为2的list
list<int> lst4(lst2);    //使用lst2初始化lst4
list<int> lst5(lst2.begin(),lst2.end());  //同lst4
```

增、删、改、查
```cpp
lst1.assign(lst2.begin(),lst2.end());  //分配值,3个值为0的元素
lst1.push_back(10);                //末尾添加值
lst1.pop_back();                   //删除末尾值
lst1.insert(lst1.begin(),3,2);     //从指定位置插入个3个值为2的元素

lst1.begin();     //返回首值的迭代器
lst1.rbegin();    //返回第一个元素的前向指针
lst1.end();       //返回尾值的迭代器
lst1.front();     //返回第一个元素的引用
lst1.back();      //返回最后一个元素的引用

lst1.erase(lst1.begin(),lst1.end());     //删除元素
lst1.clear();         //清空值

lst1.size();                       //含有元素个数
bool isEmpty1 = lst1.empty();          //判断为空
lst1.remove(2);                        //相同的元素全部删除

lst1.reverse();          //反转
lst1.sort();             //排序
lst1.unique();                         //删除相邻重复元素
```
 
遍历
```cpp
for(list<int>::const_iterator iter = lst1.begin();iter != lst1.end();iter++) {
   cout<<*iter;
}
```


## map
### 增、删、改、查
定义
```cpp
map<int,string> map1;  //空map
```

增、删、改、查
```cpp
//添加元素
map1[3] = "Saniya";
map1.insert(make_pair<int,string>(4,"V5")); //其中的type1, type2是可以不写的，让其自动推演出来；
map1.insert(pair<int,string>(1,"Siqinsini"));
map1.insert(map<int,string>::value_type(2,"Diyabi"));

//根据key取得value，key不能修改
string str = map1[3];

map1.erase(iter);    //删除迭代器数据
map1.erase(3);       //根据key删除value
map1.clear();        //清空所有元素

map1.size(); //元素个数
map1.empty(); //判断空

map<int,string>::iterator it = map1.find(0); //查找元素0
map1.count(0); //元素0的个数
```

遍历
```cpp
for(map<int,string>::iterator iter = map1.begin(); iter!=map1.end(); iter++) {
   int keyk = iter->first; //取得key
   string valuev = iter->second; //取得value
}

//c11用法
for (auto it : map) {
	it.first; //得到是key的类型，而不是一个迭代器
	it.second; //得到是value的类型，而不是一个迭代器
}
for (auto &it : map) {
	it.second = 3; //引用只能更改second.
}
```

## set集合
它是一个有序的容器，里面的元素都是排序好的支持插入、删除、查找等操作，就像一个集合一样，所有的操作都是严格在$\log_2 N$时间内完成，效率非常高。

### 增、删、改、查
定义
```cpp
//no data
```

增、删、改、查
```cpp
//no data
```

遍历
```cpp
//no data
```


## 经验
- 不要在循环中删除迭代器
	- 在循环中erase(it)会引发迭代器失效；

- 什么样的场景该用哪种容器？？

- 它们的效率对比是怎么样的？


## 扩展：std::string
### 增、删、改、查


### data(), c\_str()
定义、使用如下
```cpp
std::string str("abc");
const char* = str.c_str();
	生成一个const char*指针，指向以空字符终止的数组。
	是一个临时的，依赖str的生命周期、str的值；
char* = str.data();
	类似于c_str，只是返回的数组不以空字符终止；
```

Demo查看
```cpp
class Str
{
public:
    Str(std::string str) : mStr(str) {}
    std::string getStr() {
        return mStr;
    }
    void getStr(const char** str, size_t *len) {
        //*str = mStr.data();
        *str = mStr.c_str();
        *len = mStr.size();
    }
private:
    std::string mStr;
};
Str gS("Rabin");

void tst(const char** strPtr) //想要更改指针的指向（非内容），那么需要传二级指针；
{
#if 0
    std::string str;
    str = gS.getStr();
    *strPtr = str.c_str();
#else
    size_t len;
    gS.getStr(strPtr, &len);
#endif
    std::cout << "data in tst:" << *strPtr << std::endl;
}

int main(int argc, char* argv[])
{
    //const char *strPtr = NULL;
    const char *strPtr = NULL;
    tst(&strPtr);
    std::cout << "data in main:" << strPtr << std::endl;
    return 0;
}
```
