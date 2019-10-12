[TOC]



## ReadMe

```cpp
#include <boost/algorithm/xx.hpp>
```



## String Algorithm

https://www.boost.org/doc/libs/1_55_0/doc/html/string_algo/quickref.html

`boost/algorithm/string.hpp`文件包含如下：

```cpp
#include <boost/algorithm/string/std_containers_traits.hpp>
#include <boost/algorithm/string/trim.hpp>
#include <boost/algorithm/string/case_conv.hpp>
#include <boost/algorithm/string/predicate.hpp>
#include <boost/algorithm/string/find.hpp>
#include <boost/algorithm/string/split.hpp>
#include <boost/algorithm/string/join.hpp>
#include <boost/algorithm/string/replace.hpp>
#include <boost/algorithm/string/erase.hpp>
#include <boost/algorithm/string/classification.hpp>
#include <boost/algorithm/string/find_iterator.hpp>

// 注意：不需要链接boost的任何库，即可使用！！！
boost::algorithm::fun_name();
```



### case conversion

大小写转换函数如下：

```cpp
boost::algorithm::to_lower(str);       //str会被改变
std::string newstr = boost::algorithm::to_lower_copy(str);  //str不会被改变
to_upper();
to_upper_copy();
```



### predicates

判断/谓词函数如下：

```cpp
boost::algorithm::starts_with();
istarts_with();
ends_with();  //stringA是否以stringB结尾。
iends_with(); //同上，但不区分大小。
contains();
icontains();
equals();
iequals(); //字符串比较，忽略大小写。相等返回true.
all();
lexicographical_compare();
ilexicographical_compare();
```



### classification

分类函数如下：

```cpp
boost::algorithm::is_classified();
is_space();
is_alnum();
is_alpha();
is_cntrl();
is_digit();
is_graph();
is_lower();
is_upper();
is_print();
is_punct();
is_xdigit();
is_any_of();
is_from_range();
```



### trimming

修剪函数如下：

```cpp
boost::trim(str);  //str两端的空白符会被取消。str改变。
str2 = boost::trim_copy(str);   //删除字符串首部尾部空格，并返回新的字符串，str不改变。
trim_if();
trim_copy_if(); 
trim_left();   //去掉字符串左边的空格；
trim_left_if()();
trim_left_copy()();
trim_left_copy_if()();
trim_right();  //去掉字符串右边的空格；
trim_right_if();
trim_right_copy();
trim_right_copy_if();

//--------------------trim_all.
trim_all();  //删除str首部空格，如果中间有多个空格，只保留第一个，str改变。
trim_all_if();
trim_all_copy();
trim_all_copy_if();
trim_fill();
trim_fill_if();
trim_fill_copy();
trim_fill_copy_if();
```



### find algorithms

查找函数如下：

```cpp
find();
find_first();
ifind_first();
find_last();
ifind_last();
find_nth();
ifind_nth();
find_head();
find_tail();
find_token();

//--------------------------find_format.
find_format_copy();
find_format();
find_format_all_copy();
find_format_all();

//--------------------------find_iterator.
find_iterator();
make_find_iterator();
split_iterator();
make_split_iterator();

//------------------------finder.
first_finder();
last_finder();
nth_finder();
head_finder();
tail_finder();
token_finder();
range_finder();
```



### replace, erase

替换函数如下：

```cpp
replace_range_copy();
replace_range();
replace_first_copy();
void replace_first(str, search, replace);  //从头找到第1个词组，并进行替换；
ireplace_first_copy();
ireplace_first();
replace_last_copy();
replace_last();
ireplace_last_copy();
ireplace_last();
replace_nth_copy();
replace_nth();
ireplace_nth_copy();
ireplace_nth();
replace_all_copy();
boost::algorithm::replace_all(str, "src", "dst"); //替换str中的所有src为dst。
ireplace_all_copy();
ireplace_all();
replace_head_copy();
replace_head();
replace_tail_copy();
replace_tail();

void erase_first(str, search);
```



### find iterator

查找迭代如下：

```cpp
iter_find();
iter_split();
```



### split, join

拆分、合并如下：

```cpp
#include <boost/algorithm/string.hpp> //已包含了各子目录如，string/join.hpp

find_all();
ifind_all();

string str1("hello abc-*-ABC-*-aBc goodbye");
vector<string> vec; 
boost::split(vec, str1, boost::is_any_of("-*"), boost::token_compress_on);
	//is_any_of，字符串中的任意一个字符。
	//token_compress_on，把连续多个分隔符当一个，默认没有打开。
	//注意：str1为空的情况，会使vec.size()==1，但vec[0]为空。

#include <boost/algorithm/string/join.hpp>
boost::algorithm::join(list, ",");
join_if();
```



### boost.string.algorithm VS std ？

boost string算法 与 std库string算法有哪些区别？

std::string algorithm. https://zh.cppreference.com/w/cpp/string/basic_string
可以看出**std库只以成员函数来实现了一些基础的算法**，如查找、操作、。。

而boost的string算法则强大的多！





## Others

### string format

```cpp
#include <boost/format.hpp>
std::string str = boost::str(boost::format("%1%.%2%.%3%") %var1 %var2 %var3);
	//只能通过boost::str来转成std::string.
```



