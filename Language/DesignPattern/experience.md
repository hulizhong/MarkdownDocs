[TOC]



## ReadMe

小的设计技巧、经验、或者可以套用的模式；





## Big Data

大数据相关。



----------

**指定时间对大量数据进行检测**

每次检查或者操作，只针对特定步长内的元素。
skill, check the big data.(Every time check step elements.)

```cpp
std::vector<ConnectionObj> vec; //There are many elements in vec.
for (int i=0; i<100; i++, checkpoint++) {
    //reset the checkpoint when checkpoint reach the max value.
    if (vec[checkpoint].isTimeout()) {
        ;//...
    }
}
```




## Object Lifecycle

对象生命周期相关。



----

**对象在使用的过程中被其它线程析构**

使对象具有Refcount，只有当count=0时才会真的触发对象的析构。

```cpp
class ReferCount : public boost::noncopyable
{
public:
    ~ReferCount() {}
    friend void increase(const ReferCount* rc);
    friend void decrease(const ReferCount* rc);

private:
    mutable atomic_int count = 0;
};
inline void increase(const ReferCount* rc)
{
    ++rc->count;
}
inline void decrease(const ReferCount* rc)
{
    --rc->count;
    if (rc->cout == 0) {
        delete rc;
    }
}

class Obj : public ReferCount {
};

//increase refercount.
    //when create obj.
    //when use obj.
//decrease refercount.
    //when destory obj.
    //when unuse obj.
```


