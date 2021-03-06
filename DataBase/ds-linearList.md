[TOC]



## ReadMe

数据结构之线性表；



## 线性表

线性表

- 元素个数有限；
- 逻辑上元素有先后次序；（序列）
- 数据类型相同；
- 仅讨论元素间的逻辑关系；





### 顺序表

线性表的顺序存储结构（连续空间），可使用数组实现。

- 数组大小可静态、动态分配；
- 线性表从1开始，数组从0开始；

特点

- 优点
  - 随机访问，O(1)时间；
  - 存储密码高，逻辑上相邻，物理上也相邻；
- 缺点
  - 插入、删除需要移动大量元素，O(n)时间；
  - 难以确定顺序表的空间大小；



增加时移动元素（<font color=red>向右移动</font>），如下

```cpp
//移动pos后面的位置
int i = mSqLst.len;
for (i=mSqLst.len; i>=pos; i--) {
    mSqLst.data[i] = mSqLst.data[i-1];
}
//插入
mSqLst.data[i] = e;
mSqLst.len++;
```

删除时移动元素（<font color=red>向左移动</font>），如下

```cpp
for (int i=pos; i<=mSqLst.len; i++) {
    mSqLst.data[i-1] = mSqLst.data[i];
}
mSqLst.len--;
```



 

### 链表

线性表的链式存储结构

- 用连续或者不连续的地址进行数据元素的存储，元素间的序列关系依赖元素自己来指定； 

vs 顺序存储结构

- 允许存储空间不连接；
- 插入、删除时不需要移动大量的元素，只需要修改指针；找到位置后O(1)时间；
  - 插入、删除一般的操作步骤：<font color=red>先断、后链</font>。
- 查找需要遍历整个链表，O(n)时间；
- 顺序表需要预分配空间，容易上溢；链式在每个结点上多占空间，可以有无数多个结点只要有内存；



```cpp
struct Node {
    T item;  //数据域，存储数据元素信息；
    Node *next;  //指针域，存储直接后继元素位置；
};
```





#### 静态链表

定义：链表在顺序存储结构（数组）中的实现；

特点：

​	数组中的元素多个cur字段，用于存储本元素的直接后驱；

​	数组的第1个元素cur指向空链表、数组中的最后一个元素cur指向已用的链表；

 

优缺点：

​	Insert/delete无需移动元素；

​	表长难以确定，无随机读取的特性；



#### 单链表

由n个结点链结成一个链表，其中每个结点只包含1个指针域。

> **头结点**：非必须的属性，其里面是没有存储数据的，指针域指向链表的第一个元素；
> 头指针：必须的属性，一般冠以链表的名字；



分类（是否带头结点）

- 带头结点；
  - 头指针指向头结点；
  - 链表第1个位置节点上的操作和其它位置上的操作是一致的；
  - 无论链表是否为空，头指针都有指向；（空链表、非空链表处理一样）
- 无头结点；
  - 头指针指向第一个节点；

头结点的数据域可以不存储任何信息，头结点的[指针](https://baike.baidu.com/item/%E6%8C%87%E9%92%88)域存储指向第一个结点的指针（即第一个元素结点的存储位置）。头结点的作用是使所有[链表](https://baike.baidu.com/item/%E9%93%BE%E8%A1%A8)（包括空表）的[头指针](https://baike.baidu.com/item/%E5%A4%B4%E6%8C%87%E9%92%88/9794674)非空，并使对[单链表](https://baike.baidu.com/item/%E5%8D%95%E9%93%BE%E8%A1%A8)的插入、删除操作不需要区分是否为空表或是否在第一个位置进行，从而与其他位置的插入、删除操作一致。



----

**头插法**

```cpp
//Node *first为头指针； 
void insetHead(T data) {
    Node *newN = malloc();
    newN->next = first;
    first = newN;
}
```





---

**尾插法**

```cpp
//Node *rear为尾指针；
void insetTail(T data) {
    Node *newN = malloc();
    new->next = NULL;
    rear->next = newN;
    rear = newN;
}
```



---

**某一位置插入**

```cpp
void insertPos(T data, int pos) {
    Node *preItr, *postItr = first;
    while (preItr->data < data) {
        postItr->next = preItr;
        preItr->next = preItr->next;
    }
    
    Node *newN = malloc();
    newN->next = preItr;
    postItr->next = newN;
}
```



----

**删除结点**

```cpp
;
```





#### 循环链表

定义：将终端结点的指针域指向头结点，形成头尾相接的单链表。

- 出发点：为了解决如何从一个节点出发，访问到所有结点。
- 改进：不用头指针，而用尾指针rear指向终端结点。
  - 如此找头rear->next->next、找尾rear，都需要O(1)。 

Vs 单链表：

- 在操作上注意 单链表是`p->next != NULL`，而循环链表则是 `p->next != headNode`



#### 双（向）链表

定义：在结点中，再添加一个指针域用于指向直接前驱；

- 出发点：让查找更改容易（支持向前查找）。
  - 伴随着insert/delete更复杂。
  - 也是`空间`换`时间`的一种吧。





 

## 栈 LIFO

定义：仅在表尾（栈顶）进行Insert/delete的线性表；

 

### 顺序存储之共享空间型

特点：

​	两个栈顶分别在数组的两头向中间靠拢top1=-1, top2=n，；

​	判断栈满（top1++ == top2）；





### 链式存储之链栈

特点：

​	无头结点；

 

Vs 顺序栈：

​	顺序栈长度可控，链栈长度无限制；



### 栈与递归

Fibonacci

​           0   if n = 0

​	F(n) =  1  if n = 1 

​           F(n-1) + f(n-1)  if n>1

 



### 栈的应用

四则运行求值

​	计算机喜欢后缀表达式RPN  reverse polish notation

​	人喜欢中缀表达式；

​		中缀 -> 后缀： 数字直接输出，符号入栈后等有机会再出栈（入栈遇到后括号、优先级更低的）输出；

 

 

## 队列 FIFO

定义： 队尾Insert，队头delete的 线性表；

 



### 顺序存储之循环队列

特点：

​	Front指向头，rear指向尾；

​	问题：

​		队头 与 队尾 重合怎么算？（有可能是空，也有可能是有一个元素）  -------à rear指向队尾的下一个元素；

​		Front只向上偏移不移动数据会造成假上溢。    -----à 循环队列+留个空位：rear==front则空，(rear+1)%queuesize == front则队满

​	

### 链队列

特点：

​	Front指向头结点；

​	Rear指向终端结点；

 

Vs 循环队列：

​	能确定队列长度的最大值的情况下用循环队列；

​	无法预估队列长度时用链队列；

 







 