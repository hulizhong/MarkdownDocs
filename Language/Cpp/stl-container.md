[TOC]

## readme

容器类自动申请和释放内存，我们无需new和delete操作。

容器大比拼

|名称 |内部数据结构 |操作元素的方式 |插入删除操作迭代器是否失效 |
|----|-----------|------------|----------------------|
|vector |连续存储的数组形式（一端开口） |下标运算符、迭代器  |会|
|deque |连续或分段连续存储数组（两端开口） |下标运算符、迭代器 |插入迭代器失效；<br />删头尾指向被删除节点的迭代器失效，删除中间元素所有迭代器失效|
|list |双向环状链表 |迭代器 |删除指向被删除的迭代器失效|
|set |红黑树 |迭代器 |删除指向被删除的迭代器失效|
|multiset |红黑树 |迭代器 |删除指向被删除的迭代器失效|
|map |红黑树 |迭代器 |删除指向被删除的迭代器失效|
|multimap |红黑树 |迭代器 |删除指向被删除的迭代器失效|



### 几个坑

**T.方法**
`back(), front(), pop_back(), pop_front()`; 这几个接口在空容器上操作是未定义的。





## category

分类概要信息如下

- 顺序容器，<font color=red>提供能按顺序访问的数据结构</font>。
  - array，静态数组。（<font color=gree>变长一维数组，连续存放的内存块，有保留内存，堆中分配内存</font>）
  - vector，动态数组。
  - deque，双端队列。（<font color=gree>在堆上分配内存，一个堆保存几个元素，而堆之间使用指针连接</font>）
  - forward_list，单链表。
  - list，双链表。（<font color=gree>内存空间上可能是不连续的，无保留内存，堆中分配内存</font>）
- 关联容器，<font color=red>提供快速查找*$O(log_2N)$*数据结构</font>。
  - set，集合，key排序。（<font color=gree>使用平衡二叉树存储</font>）
  - map，key-value映射集，按key排序。（<font color=gree>使用平衡二叉树存储</font>）
  - multiset。
  - multimap。
- 无序关联容器（按key生成散列），<font color=red>提供快速查找（均摊*O(1)*，最坏*O(n)*）数据结构</font>。
  - unordered_set，就是以前的hash_set.
  - unordered_map `#include <unordered_map>`，就是以前的hash_map. `#include <ext/hash_map>`
  - unordered_multiset
  - unordered_multimap
- 容器适配器，<font color=red>提供顺序容器的不同接口</font>。
  - stack，栈LIFO。底层容器默认为deque（可指定成其它顺序容器也可）。
  - queue，队列FIFO。底层容器默认为deque（可指定成其它顺序容器也可）。
  - priority_queue，优先级队列。底层容器默认为vector（可指定成其它顺序容器也可）。



### 迭代器、元素引用失效

- 顺序容器
  - array，不能增加、删除。
  - vector，迭代器在内存重新分配时将失效（它所指向的元素在该操作的前后不再相同）。
    - 当把超过capacity()-size()个元素插入vector中时，内存会重新分配，所有的迭代器都将失效；否则，指向当前元素以后的任何元素的迭代器都将失效。
    - 当删除元素时，指向被删除元素以后的任何元素的迭代器都将失效。。
  - deque
    - 增加任何元素都将使deque的迭代器失效。
    - 在deque的中间删除元素将使迭代器失效。在deque的头或尾删除元素时，只有指向该元素的迭代器失效。
  - list, forward_list.
    - 增加任何元素都不会使迭代器失效。
    - 删除元素时，除了指向当前被删除元素的迭代器外，其它迭代器都不会失效。
- 关联容器
  - set、multiset、map、multimap
    - 如果迭代器所指向的元素被删除，则该迭代器失效。其它任何增加、删除元素的操作都不会使迭代器失效。
  - ...
- 无序关联容器
  - unordered_set
  - unordered_map
  - unordered_multiset
  - unordered_multimap
- 容器适配器
  - stack，不能遍历整个stack。
  - queue，不能遍历整个queue。
  - priority_queue，只能访问第一个元素，不能遍历整个priority_queue。



### 查找、增删效率

分类概要信息如下

- <font color=red>按顺序访问型</font>：顺序容器
  - array
  - vector，增加和获取元素效率很高，插入和删除的效率很低。
    - 随机访问——常数 *O(1)*
    - 在**末尾**插入、移除元素——均摊常数 *O(1)*
    - 插入、移除元素——与到 vector 结尾的距离成线性 *O(n)*
  - deque，增加和获取元素效率较高，插入和删除的效率较高？---跟vector差不多rabin?
    - 随机访问——常数 *O(1)*
    - 在**结尾、起始**插入或移除元素——常数 *O(1)*
    - 插入或移除元素——线性 *O(n)*
  - forward_list
  - list，增加和获取元素效率很低，插入和删除的效率很高 。
    - 查找*O(n)*
    - 插入、删除为*O(1)*
- <font color=red>查找型</font>：关联容器，红黑树实现（<font color=gree>插入、删除、查找都严格在$\log_2 N$时间内完成，效率非常高</font>）。
  - set
  - map
  - multiset
  - multimap
- <font color=red>查找型</font>：无序关联容器，hash实现（<font color=gree>相对RBT有更好的查找性能，但取决于哈希函数和哈希表的负载情况；但最好的情况是*O(1)*，最坏是*O(N)*</font>）。
  - unordered_set
  - unordered_map
  - unordered_multiset
  - unordered_multimap
- 容器适配器
  - stack
  - queue
  - priority_queue



**Q, 什么时候选顺序容器？什么时候选关联容器？**
顺序容器一般提供顺序访问功能，关联容器一般提供查找功能。

**Q, 那么从效率上来讲，如何选择容器呢？**

如果你<font color=red>需要高效的随机存取，而不关心插入和删除的效率</font>，使用vector.
如果你<font color=red>需要大量的插入和删除，而不关心随即存取</font>，则应使用list.
如果你<font color=red>需要随即存取，而且关心两端数据的插入和删除</font>，则应使用deque.







### thread safe



## Q&A

**Q, vector.remove() vs vector.erase() ?**
remove没有真正的删除元素（而是将这些元素移到了容器的尾部），而且size()没有变化。
erase()真正的删除了指定元素。
----------list.remove()同erase()一样释放了资源的？





## std::vector

**向量、动态数组**：有序集、支持随机访问。

```cpp
//-------------------------------------------------特征
push_back();
pop_back();
at();  //越界检查，报异常。
operator[];

//------------------------------------------------init
vector<int> vec1;    //默认初始化，vec1为空
vector<int> vec2(vec1);  //使用vec1初始化vec2
vector<int> vec3(vec1.begin(),vec1.end());//使用vec1初始化vec2
vector<int> vec4(10);    //10个值为0的元素
vector<int> vec5(10,4);  //10个值为4的元素

vector v1, v2;

insert()
v1.insert(v1.end(),5,3);    //从vec1.back位置插入5个值为3的元素
v2.assign(v1.begin(), v1.end())  //根据v1给v2赋值

v[0];  //取得第1个元素；
v.at(0);  //取得第1个元素，会检查跃界与否。
v.front();  //取得第1个元素；
v.push_back() //在Vector最后添加一个元素（参数为要插入的值）
v.back(); //取得最后1个元素；
v.pop_back(); //删除末尾元素

iterator erase( iterator loc );                   //要删除元素的迭代器
iterator erase( iterator start, iterator end);    //要删除的第一个元素的迭代器，要删除的第二个元素的迭代器
clear(); //清空所有元素
empty(); //判断Vector是否为空（返回true时为空）
size();  //返回元素个数；

//没有查找函数：
#include <algorithm>
if ( std::find(vec.begin(), vec.end(), item) != vec.end() ) {
    //can find it.
}
 
//---------------------------------------------------foreach
//c11
for (auto val : valList) {
	cout << val << endl; //这样迭代出来的var是vector中的元素，而非迭代器
}

auto iter = a.begin();
while (iter != a.end()) {
	if (*iter > 30) {
		iter = a.erase(iter);
	}
	else {
		++iter;
	}
}
```


### 问题点
vector使用超出了其空间
```bash
std::vector<int> va(-1);  初始化为-1个空间？
libc++abi.dylib: terminating with uncaught exception of type std::length_error: vector
critical handler: signal 6 is triggered.
```



## std::list

**双向链表**：。。。

```cpp
//--------------特征
insert();
erase();

//--------------------初始化.
list<int> lst1;          //创建空list
list<int> lst2(3);       //创建含有三个元素的list
list<int> lst3(3,2);     //创建含有三个元素为2的list
list<int> lst4(lst2);    //使用lst2初始化lst4
list<int> lst5(lst2.begin(),lst2.end());  //同lst4

//----------------------
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

//-----------------------遍历
for(list<int>::const_iterator iter = lst1.begin();iter != lst1.end();iter++) {
   cout<<*iter;
}
```



## std::deque

**双端队列**：。。。

```cpp
//-------------------特征
operator[];
at();
push_back();
pop_back();
push_front();
pop_front();


```



## std::array

```cpp
#include <array>
std::array<int, 4> a = {1, 2, 3, 4};
```







## std::map



```cpp
// --------------------定义
map<int,string> map1;  //空map

//------------------------- 添加
map1[3] = "Saniya";
map1.insert(std::make_pair<int,string>(4,"V5")); 
	//std::make_pair的模板参数type1, type2是可以不写的，让其自动推演出来；
map1.insert(std::pair<int,string>(1,"Siqinsini"));
	//std::pair的模板参数需要特化；
map1.insert(map<int,string>::value_type(2,"Diyabi"));

// ----------------------- 获取
try {
    string str = map1.at(3); //没有则会抛异常。
    string str = map1[3];    //没有则会构造一个默认的value回来（默认构造函数，如int为0）。
}
catch (exception &e) {...}
map1.count(0); //元素0的个数
map<int,string>::iterator it = map1.find(0); //查找元素0

// ---------------- 删除
map1.erase(iter);    //删除迭代器数据
map1.erase(3);       //根据key删除value
map1.clear();        //清空所有元素

map1.size(); //元素个数
map1.empty(); //判断空

//----------------遍历
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



### 问题点
```cpp
const map<std::string, std::string> mp;
std::string value = mp["key"];  //const map不能用数组法[]去获取value.
std::string value = mp.at("key");
```



## set集合



## std::stack

**栈，LIFO**：。。。



## std::queue

**队列，FIFO**：。。。。

the functions in queue template class, as follow.

```cpp
#include <queue>

//--------------------------------------------特征


//--------------------------------------------
queue<int> q;
q.push(4); //push to item at the end of queue.
q.front(); //return the first item, but not remove it.
q.back();  //return the lastest item, but not remove it.
q.pop();   //remove the first item, but not return.
```

<font color=red>小心：queue内没有元素，那么front()，back(), pop()的执行都会导致未定义的行为，所以在执行这三个操作是，可以通过size()和empty()判断容器是否为空</font>；



## std::priority_queue

**优先队列、二叉堆**：首元素最大或者最小，特征如下：
元素内有个<font color=gree>key，优先级排序就是靠这个key</font>判断；如果元素为自定义类型，那么需要重载比较符<font color=gree>< =</font>。
只能从访问头部，<font color=gree>不支持随机访问</font>。

https://blog.csdn.net/m0_37925202/article/details/81916600





## std::string

有三类string

```cpp
std::basic_string<T>;  //模板
std::string      std::basic_string<char>;  //实例化string.
std::wstring     std::basic_string<wchar_t>;  //实例化wstring.
std::u16string   std::basic_string<char16_t>; //实例化u16string.
std::u32string   std::basic_string<char32_t>; //实例化u32string.
```





增、删、改、查
https://www.cnblogs.com/yencain/articles/3110503.html

```cpp
//构造函数
string(const char *s); 

const char *data()const;  //返回一个非null终止的c字符数组
const char *c_str()const; //返回一个以null终止的c字符串

//string的赋值：
string &operator=(const string &s); //把字符串s赋给当前字符串
string &assign(const char *s); //用c类型字符串s赋值
string &assign(const char *s,i nt n); //用c字符串s开始的n个字符赋值
string &assign(const string &s);//把字符串s赋给当前字符串
string &assign(int n,char c);//用n个字符c赋值给当前字符串
string &assign(const string &s,int start,int n);//把字符串s中从start开始的n个字符赋给当前字符串
string &assign(const_iterator first,const_itertor last);//把first和last迭代器之间的部分赋给字符串
```

### data(), c\_str()
这两函数易出问题是，请看如下：
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

