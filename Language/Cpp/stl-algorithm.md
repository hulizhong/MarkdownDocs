[TOC]

## ReadMe
stl算法库，包含了如下：

```cpp
#include <algorithm>  //算法。
#include <numeric>    //数值算法。
#include <functional> //函数对象。
```

大致分为四大类：

- 非可变序列算法：不直接修改其所操作的容器内容的算法。
- 可变序列算法：可以修改它们所操作的容器内容的算法。
- 排序算法：对序列进行排序、合并的算法、搜索算法、有序序列上的集合操作。
- 数值算法：容器内容进行数值计算。





## in cppreference

| 不修改序列的操作                                             |                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 定义于头文件 `<algorithm>`                                   |                                                              |
| [all_ofany_ofnone_of](https://zh.cppreference.com/w/cpp/algorithm/all_any_none_of)(C++11)(C++11)(C++11) | 检查一定范围之内，是否全部、存在或不存在元素使得谓词为true  (函数模板) |
| [for_each](https://zh.cppreference.com/w/cpp/algorithm/for_each) | 将一个函数应用于某一范围的元素  (函数模板)                   |
| [for_each_n](https://zh.cppreference.com/w/cpp/algorithm/for_each_n)(C++17) | 应用函数对象到序列的首 n 个元素  (函数模板)                  |
| [countcount_if](https://zh.cppreference.com/w/cpp/algorithm/count) | 返回满足指定判别的元素数  (函数模板)                         |
| [mismatch](https://zh.cppreference.com/w/cpp/algorithm/mismatch) | 查找两个范围第一个不同元素的位置  (函数模板)                 |
| [findfind_iffind_if_not](https://zh.cppreference.com/w/cpp/algorithm/find)(C++11) | 查找满足特定条件的第一个元素  (函数模板)                     |
| [find_end](https://zh.cppreference.com/w/cpp/algorithm/find_end) | 查找一定范围内最后出现的元素序列  (函数模板)                 |
| [find_first_of](https://zh.cppreference.com/w/cpp/algorithm/find_first_of) | 查找元素集合中的任意元素  (函数模板)                         |
| [adjacent_find](https://zh.cppreference.com/w/cpp/algorithm/adjacent_find) | 查找彼此相邻的两个相同（或其它的关系）的元素  (函数模板)     |
| [search](https://zh.cppreference.com/w/cpp/algorithm/search) | 查找一个元素区间  (函数模板)                                 |
| [search_n](https://zh.cppreference.com/w/cpp/algorithm/search_n) | 在区间中搜索连续一定数目次出现的元素  (函数模板)             |
| 修改序列的操作                                               |                                                              |
| 定义于头文件 `<algorithm>`                                   |                                                              |
| [copycopy_if](https://zh.cppreference.com/w/cpp/algorithm/copy)(C++11) | 将某一范围的元素复制到一个新的位置  (函数模板)               |
| [copy_n](https://zh.cppreference.com/w/cpp/algorithm/copy_n)(C++11) | 复制一定数目的元素到新的位置  (函数模板)                     |
| [copy_backward](https://zh.cppreference.com/w/cpp/algorithm/copy_backward) | 按从后往前的顺序复制一个范围内的元素  (函数模板)             |
| [move](https://zh.cppreference.com/w/cpp/algorithm/move)(C++11) | 将某一范围的元素移动到一个新的位置  (函数模板)               |
| [move_backward](https://zh.cppreference.com/w/cpp/algorithm/move_backward)(C++11) | 按从后往前的顺序移动某一范围的元素到新的位置  (函数模板)     |
| [fill](https://zh.cppreference.com/w/cpp/algorithm/fill)     | 将一个值赋给一个范围内的元素  (函数模板)                     |
| [fill_n](https://zh.cppreference.com/w/cpp/algorithm/fill_n) | 将一个值赋给一定数目的元素  (函数模板)                       |
| [transform](https://zh.cppreference.com/w/cpp/algorithm/transform) | 将一个函数应用于某一范围的元素  (函数模板)                   |
| [generate](https://zh.cppreference.com/w/cpp/algorithm/generate) | 赋值相继的函数调用结果给范围中的每个元素  (函数模板)         |
| [generate_n](https://zh.cppreference.com/w/cpp/algorithm/generate_n) | 赋值相继的函数调用结果给范围中的 N 个元素  (函数模板)        |
| [removeremove_if](https://zh.cppreference.com/w/cpp/algorithm/remove) | 移除满足特定标准的元素  (函数模板)                           |
| [remove_copyremove_copy_if](https://zh.cppreference.com/w/cpp/algorithm/remove_copy) | 复制一个范围内不满足特定条件的元素  (函数模板)               |
| [replacereplace_if](https://zh.cppreference.com/w/cpp/algorithm/replace) | 将所有满足特定条件的元素替换为另一个值  (函数模板)           |
| [replace_copyreplace_copy_if](https://zh.cppreference.com/w/cpp/algorithm/replace_copy) | 复制一个范围内的元素，并将满足特定条件的元素替换为另一个值  (函数模板) |
| [swap](https://zh.cppreference.com/w/cpp/algorithm/swap)     | 交换两个对象的值  (函数模板)                                 |
| [swap_ranges](https://zh.cppreference.com/w/cpp/algorithm/swap_ranges) | 交换两个范围的元素  (函数模板)                               |
| [iter_swap](https://zh.cppreference.com/w/cpp/algorithm/iter_swap) | 交换两个迭代器所指向的元素  (函数模板)                       |
| [reverse](https://zh.cppreference.com/w/cpp/algorithm/reverse) | 将区间内的元素颠倒顺序  (函数模板)                           |
| [reverse_copy](https://zh.cppreference.com/w/cpp/algorithm/reverse_copy) | 将区间内的元素颠倒顺序并复制  (函数模板)                     |
| [shift_leftshift_right](https://zh.cppreference.com/w/cpp/algorithm/shift)(C++20) | 迁移范围中的元素  (函数模板)                                 |
| [rotate](https://zh.cppreference.com/w/cpp/algorithm/rotate) | 将区间内的元素旋转  (函数模板)                               |
| [rotate_copy](https://zh.cppreference.com/w/cpp/algorithm/rotate_copy) | 将区间内的元素旋转并复制  (函数模板)                         |
| [random_shuffle](https://zh.cppreference.com/w/cpp/algorithm/random_shuffle) | 将范围内的元素随机重新排序  (函数模板)                       |
| [sample](https://zh.cppreference.com/w/cpp/algorithm/sample)(C++17) | 从一个序列中随机选择 n 个元素  (函数模板)                    |
| [unique](https://zh.cppreference.com/w/cpp/algorithm/unique) | 删除区间内连续重复的元素  (函数模板)                         |
| [unique_copy](https://zh.cppreference.com/w/cpp/algorithm/unique_copy) | 删除区间内连续重复的元素并复制  (函数模板)                   |
| 划分操作                                                     |                                                              |
| 定义于头文件 `<algorithm>`                                   |                                                              |
| [is_partitioned](https://zh.cppreference.com/w/cpp/algorithm/is_partitioned)(C++11) | 判断区间是否被给定的谓词划分  (函数模板)                     |
| [partition](https://zh.cppreference.com/w/cpp/algorithm/partition) | 把一个区间的元素分为两组  (函数模板)                         |
| [partition_copy](https://zh.cppreference.com/w/cpp/algorithm/partition_copy)(C++11) | 将区间内的元素分为两组复制到不同位置  (函数模板)             |
| [stable_partition](https://zh.cppreference.com/w/cpp/algorithm/stable_partition) | 将元素分为两组，同时保留其相对顺序  (函数模板)               |
| [partition_point](https://zh.cppreference.com/w/cpp/algorithm/partition_point)(C++11) | 定位已划分的区域的划分点  (函数模板)                         |
| 排序操作                                                     |                                                              |
| 定义于头文件 `<algorithm>`                                   |                                                              |
| [is_sorted](https://zh.cppreference.com/w/cpp/algorithm/is_sorted)(C++11) | 检查区间元素是否按升序排列  (函数模板)                       |
| [is_sorted_until](https://zh.cppreference.com/w/cpp/algorithm/is_sorted_until)(C++11) | 找出最大的已排序子范围  (函数模板)                           |
| [sort](https://zh.cppreference.com/w/cpp/algorithm/sort)     | 将区间按升序排序  (函数模板)                                 |
| [partial_sort](https://zh.cppreference.com/w/cpp/algorithm/partial_sort) | 将区间内较小的N个元素排序  (函数模板)                        |
| [partial_sort_copy](https://zh.cppreference.com/w/cpp/algorithm/partial_sort_copy) | 对区间内的元素进行复制并部分排序  (函数模板)                 |
| [stable_sort](https://zh.cppreference.com/w/cpp/algorithm/stable_sort) | 将区间内的元素排序，同时保持相等的元素之间的顺序  (函数模板) |
| [nth_element](https://zh.cppreference.com/w/cpp/algorithm/nth_element) | 将给定的区间部分排序，确保区间被给定的元素划分  (函数模板)   |
| 二分搜索操作（在已排序范围上）                               |                                                              |
| 定义于头文件 `<algorithm>`                                   |                                                              |
| [lower_bound](https://zh.cppreference.com/w/cpp/algorithm/lower_bound) | 返回指向第一个不小于给定值的元素的迭代器  (函数模板)         |
| [upper_bound](https://zh.cppreference.com/w/cpp/algorithm/upper_bound) | 返回指向第一个大于给定值的元素的迭代器  (函数模板)           |
| [binary_search](https://zh.cppreference.com/w/cpp/algorithm/binary_search) | 判断一个元素是否在区间内  (函数模板)                         |
| [equal_range](https://zh.cppreference.com/w/cpp/algorithm/equal_range) | 返回匹配特定键值的元素区间  (函数模板)                       |
| 集合操作（在已排序范围上）                                   |                                                              |
| 定义于头文件 `<algorithm>`                                   |                                                              |
| [merge](https://zh.cppreference.com/w/cpp/algorithm/merge)   | 合并两个已排序的区间  (函数模板)                             |
| [inplace_merge](https://zh.cppreference.com/w/cpp/algorithm/inplace_merge) | 就地合并两个有序的区间  (函数模板)                           |
| [includes](https://zh.cppreference.com/w/cpp/algorithm/includes) | 如果一个集合是另外一个集合的子集则返回true  (函数模板)       |
| [set_difference](https://zh.cppreference.com/w/cpp/algorithm/set_difference) | 计算两个集合的差集  (函数模板)                               |
| [set_intersection](https://zh.cppreference.com/w/cpp/algorithm/set_intersection) | 计算两个集合的交集  (函数模板)                               |
| [set_symmetric_difference](https://zh.cppreference.com/w/cpp/algorithm/set_symmetric_difference) | 计算两个集合的对称差  (函数模板)                             |
| [set_union](https://zh.cppreference.com/w/cpp/algorithm/set_union) | 计算两个集合的并集  (函数模板)                               |
| 堆操作                                                       |                                                              |
| 定义于头文件 `<algorithm>`                                   |                                                              |
| [is_heap](https://zh.cppreference.com/w/cpp/algorithm/is_heap) | 检查给定的区间是否为一个堆  (函数模板)                       |
| [is_heap_until](https://zh.cppreference.com/w/cpp/algorithm/is_heap_until)(C++11) | 查找区间中为堆的最大子区间  (函数模板)                       |
| [make_heap](https://zh.cppreference.com/w/cpp/algorithm/make_heap) | 根据区间内的元素创建出一个堆  (函数模板)                     |
| [push_heap](https://zh.cppreference.com/w/cpp/algorithm/push_heap) | 将元素加入到堆  (函数模板)                                   |
| [pop_heap](https://zh.cppreference.com/w/cpp/algorithm/pop_heap) | 将堆中的最大元素删除  (函数模板)                             |
| [sort_heap](https://zh.cppreference.com/w/cpp/algorithm/sort_heap) | 将堆变成一个排好序的区间  (函数模板)                         |
| 最小/最大操作                                                |                                                              |
| 定义于头文件 `<algorithm>`                                   |                                                              |
| [max](https://zh.cppreference.com/w/cpp/algorithm/max)       | 返回两个元素中的较大者  (函数模板)                           |
| [max_element](https://zh.cppreference.com/w/cpp/algorithm/max_element) | 返回区间内的最大元素  (函数模板)                             |
| [min](https://zh.cppreference.com/w/cpp/algorithm/min)       | 返回两个元素中的较小者  (函数模板)                           |
| [min_element](https://zh.cppreference.com/w/cpp/algorithm/min_element) | 返回区间内的最小元素  (函数模板)                             |
| [minmax](https://zh.cppreference.com/w/cpp/algorithm/minmax)(C++11) | 返回两个元素中的的较大者和较小者  (函数模板)                 |
| [minmax_element](https://zh.cppreference.com/w/cpp/algorithm/minmax_element)(C++11) | 返回区间内的最小元素和最大元素  (函数模板)                   |
| [clamp](https://zh.cppreference.com/w/cpp/algorithm/clamp)(C++17) | 在一对边界值间夹住一个值  (函数模板)                         |
| 比较操作                                                     |                                                              |
| 定义于头文件 `<algorithm>`                                   |                                                              |
| [equal](https://zh.cppreference.com/w/cpp/algorithm/equal)   | 确定两个元素集合是否是相同的  (函数模板)                     |
| [lexicographical_compare](https://zh.cppreference.com/w/cpp/algorithm/lexicographical_compare) | 如果按字典顺序一个区间小于另一个区间，返回true  (函数模板)   |
| [compare_3way](https://zh.cppreference.com/w/cpp/algorithm/compare_3way)(C++20) | 用三路比较比较二个值  (函数模板)                             |
| [lexicographical_compare_3way](https://zh.cppreference.com/w/cpp/algorithm/lexicographical_compare_3way)(C++20) | 用三路比较比价二个范围  (函数模板)                           |
| 排列操作                                                     |                                                              |
| 定义于头文件 `<algorithm>`                                   |                                                              |
| [is_permutation](https://zh.cppreference.com/w/cpp/algorithm/is_permutation)(C++11) | 判断一个序列是否为另一个序列的排列组合  (函数模板)           |
| [next_permutation](https://zh.cppreference.com/w/cpp/algorithm/next_permutation) | 按字典顺序产生区间内元素下一个较大的排列组合  (函数模板)     |
| [prev_permutation](https://zh.cppreference.com/w/cpp/algorithm/prev_permutation) | 按字典顺序产生区间内元素下一个较小的排列组合  (函数模板)     |
| 数值运算                                                     |                                                              |
| 定义于头文件 `<numeric>`                                     |                                                              |
| [iota](https://zh.cppreference.com/w/cpp/algorithm/iota)(C++11) | 用从起始值开始连续递增的值填充区间  (函数模板)               |
| [accumulate](https://zh.cppreference.com/w/cpp/algorithm/accumulate) | 计算区间内元素的和  (函数模板)                               |
| [inner_product](https://zh.cppreference.com/w/cpp/algorithm/inner_product) | 计算两个区间元素的内积  (函数模板)                           |
| [adjacent_difference](https://zh.cppreference.com/w/cpp/algorithm/adjacent_difference) | 计算区间内相邻元素之间的差  (函数模板)                       |
| [partial_sum](https://zh.cppreference.com/w/cpp/algorithm/partial_sum) | 计算区间内元素的部分和  (函数模板)                           |
| [reduce](https://zh.cppreference.com/w/cpp/algorithm/reduce)(C++17) | 类似 [std::accumulate](https://zh.cppreference.com/w/cpp/algorithm/accumulate) ，除了以乱序  (函数模板) |
| [exclusive_scan](https://zh.cppreference.com/w/cpp/algorithm/exclusive_scan)(C++17) | 类似 [std::partial_sum](https://zh.cppreference.com/w/cpp/algorithm/partial_sum) ，第 i 个和中排除第 i 个输入  (函数模板) |
| [inclusive_scan](https://zh.cppreference.com/w/cpp/algorithm/inclusive_scan)(C++17) | 类似 [std::partial_sum](https://zh.cppreference.com/w/cpp/algorithm/partial_sum) ，第 i 个和中包含第 i 个输入  (函数模板) |
| [transform_reduce](https://zh.cppreference.com/w/cpp/algorithm/transform_reduce)(C++17) | 应用函数对象，然后以乱序规约  (函数模板)                     |
| [transform_exclusive_scan](https://zh.cppreference.com/w/cpp/algorithm/transform_exclusive_scan)(C++17) | 应用函数对象，然后进行排除扫描  (函数模板)                   |
| [transform_inclusive_scan](https://zh.cppreference.com/w/cpp/algorithm/transform_inclusive_scan)(C++17) | 应用函数对象，然后进行包含扫描  (函数模板)                   |
| 未初始化内存上的操作                                         |                                                              |

| 定义于头文件 `<memory>`                                      |                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [uninitialized_copy](https://zh.cppreference.com/w/cpp/memory/uninitialized_copy) | 将范围内的对象复制到未初始化的内存区域  (函数模板)           |
| [uninitialized_copy_n](https://zh.cppreference.com/w/cpp/memory/uninitialized_copy_n)(C++11) | 将指定数量的对象复制到未初始化的内存区域  (函数模板)         |
| [uninitialized_fill](https://zh.cppreference.com/w/cpp/memory/uninitialized_fill) | 复制一个对象到以范围定义的未初始化内存区域  (函数模板)       |
| [uninitialized_fill_n](https://zh.cppreference.com/w/cpp/memory/uninitialized_fill_n) | 复制一个对象到以起点和计数定义的未初始化内存区域  (函数模板) |
| [uninitialized_move](https://zh.cppreference.com/w/cpp/memory/uninitialized_move)(C++17) | 移动一个范围的对象到未初始化的内存区域  (函数模板)           |
| [uninitialized_move_n](https://zh.cppreference.com/w/cpp/memory/uninitialized_move_n)(C++17) | 移动一定数量对象到未初始化内存区域  (函数模板)               |
| [uninitialized_default_construct](https://zh.cppreference.com/w/cpp/memory/uninitialized_default_construct)(C++17) | 在范围所定义的未初始化的内存区域以[默认初始化](https://zh.cppreference.com/w/cpp/language/default_initialization)构造对象  (函数模板) |
| [uninitialized_default_construct_n](https://zh.cppreference.com/w/cpp/memory/uninitialized_default_construct_n)(C++17) | 在起始和计数所定义的未初始化内存区域用[默认初始化](https://zh.cppreference.com/w/cpp/language/default_initialization)构造对象  (函数模板) |
| [uninitialized_value_construct](https://zh.cppreference.com/w/cpp/memory/uninitialized_value_construct)(C++17) | 在范围所定义的未初始化内存中用[值初始化](https://zh.cppreference.com/w/cpp/language/value_initialization)构造对象  (函数模板) |
| [uninitialized_value_construct_n](https://zh.cppreference.com/w/cpp/memory/uninitialized_value_construct_n)(C++17) | 在起始和计数所定义的未初始化内存区域以[值初始化](https://zh.cppreference.com/w/cpp/language/value_initialization)构造对象  (函数模板) |
| [destroy_at](https://zh.cppreference.com/w/cpp/memory/destroy_at)(C++17) | 销毁在给定地址的对象  (函数模板)                             |
| [destroy](https://zh.cppreference.com/w/cpp/memory/destroy)(C++17) | 销毁一个范围中的对象  (函数模板)                             |
| [destroy_n](https://zh.cppreference.com/w/cpp/memory/destroy_n)(C++17) | 销毁范围中一定数量的对象  (函数模板)                         |
| C 库                                                         |                                                              |
| 定义于头文件 `<cstdlib>`                                     |                                                              |
| [qsort](https://zh.cppreference.com/w/cpp/algorithm/qsort)   | 排序类型未指定的元素的范围  (函数)                           |
| [bsearch](https://zh.cppreference.com/w/cpp/algorithm/bsearch) | 在未指定类型的数组中搜索元素  (函数)                         |

