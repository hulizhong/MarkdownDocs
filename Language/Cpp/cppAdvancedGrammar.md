

[TOC]



## ReadMe



## 模板相关

**Q, 用`typename`声明返回类型：**

```cpp
template<typename T, std::size_t N>
typename std::enable_if<N < 128 && std::is_integral<T>::value, void>::type 
insertion_sort(std::array<T, N>& array) {
  for (std::size_t i = 0; i < N; i++) {
    for (std::size_t j = i; j > 0 && array[j] < array[j-1]; j--) {
      std::swap(array[j], array[j - 1]);
    }
  }
}

typedef int haha;
//typename haha main() //编译不过
int main()
{
    std::array<int, 127> a1;
    insertion_sort(a1);
    //std::array<int, 129> a2;
    //insertion_sort(a2);
    return 0;
}
```



**Q, 探测模板参数是否为指针类型？**

```cpp
template <typename T>
struct is_pointer
{
    enum {
        value = false
    };
};

template <typename T>
struct is_pointer<T*>
{
    enum {value = true};
};

int main()
{
    //std::cout << std::is_pointer<int>::value << std::endl;
    //std::cout << std::is_pointer<int&>::value << std::endl;
    //std::cout << std::is_pointer<int*>::value << std::endl;
    //std::cout << std::is_pointer<int**>::value << std::endl;

    std::cout << is_pointer<int>::value << std::endl;
    std::cout << is_pointer<int&>::value << std::endl;
    std::cout << is_pointer<int*>::value << std::endl;
    std::cout << is_pointer<int**>::value << std::endl;
    return 0;
}
```

