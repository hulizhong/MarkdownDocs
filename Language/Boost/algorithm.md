[TOC]



## ReadMe

```cpp
#include <boost/algorithm/xx.hpp>
```



## String Algorithm

```cpp
#include <boost/algorithm/string.hpp>
```



### case conversion

```cpp
boost::algorithm::to_upper();
to_lower_copy();
to_upper();
to_upper_copy();
```



### predicates

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

```cpp
trim_left();
trim_left_if()();
trim_left_copy()();
trim_left_copy_if()();
trim_right();
trim_right_if();
trim_right_copy();
trim_right_copy_if();
trim();
trim_if();
trim_copy();
trim_copy_if();

//--------------------trim_all.
trim_all();
trim_all_if();
trim_all_copy();
trim_all_copy_if();
trim_fill();
trim_fill_if();
trim_fill_copy();
trim_fill_copy_if();
```



### find algorithms

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



### replace

```cpp
replace_range_copy();
replace_range();
replace_first_copy();
replace_first();
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
replace_all();
ireplace_all_copy();
ireplace_all();
replace_head_copy();
replace_head();
replace_tail_copy();
replace_tail();
```



### find iterator

```cpp
iter_find();
iter_split();
```



### split

```cpp
find_all();
ifind_all();
split();
```



### join

```cpp
join();
join_if();
```

