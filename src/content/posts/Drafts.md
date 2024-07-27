---
title: Drafts
published: 2024-07-16
description: '杂项笔记'
image: ''
tags: []
category: ''
draft: false 
---

# C++杂项小知识
- C++中std::string小于15个字节，小字符串优化(SVO)，存入string本身的union中(见basic_string.h),存入union的_M_local_buf(15字节)里
- 访问移动后的原对象语言层面可行，但是标准库不保证移动后原对象是空，不同编译器对于移动的实现有差异，如MSVC对string小字符串的情况拷贝和移动一致，原对象未被清空.
- std::transform
- linux wchar_t32位
- C++继承的C语言一些函数调用全局locale(调用时检索locale全局是否包含字符，很慢，而且字符集不稳定，不如直接用strcchr),需要用到如STL的is系列函数最好还是自己实现
- memcpy的src和dst不能为空指针->给size加个判断,size为0时跳过不用memcpy
- memcpy不能接受带有重叠的src和dst(void *\__restrict dest, const void *\__restrict src),高度矢量化
    - memmove没有restrict修饰,guaranteeing correct behavior for overlapping strings
- T类型指针必须对齐到alignof(T),malloc只保证对齐到max_align_t(GCC 16字节),new T保证对齐到alignof(T),否则为未定义行为
    - 与处理器无关，C++不允许不对齐访问
```cpp
alignas(alignof(int)) char buf[sizeof(int)]; // 使用alignas进行对齐
int *p = (int *)buf; // 还是UB，见下面
//待检验?： 在C++20前，还需要new(buf) int初始化，c++17及后还需要std::launder洗一下。另外，我不太清楚标准什么时候开始修复的char可以用来提供存储，一开始只有unsigned char和std::byte可以，用char的话，new(buf) int就会导致buf的生命周期结束。
```

```cpp
char buf[sizeof(int)*2];
int *p = (int *)(((uintptr_t)buf + sizeof(int) - 1) & ~(alignof(int) - 1)); // 手动取与对齐
```

- Debug配置的MSVC STL中，'&*p'会产生断言异常而'p.get()'不会
- objdump -O build/main -j.text
- std::lock,std::unlock,内部根据地址排序,地址低的先上锁。
- recursive_mutex,unique_lock
- subspan

# 容器迭代器失效

```cpp
std::vector<int> v = { 1, 2, 3 };
auto it = v.begin();
v.push_back(4); // push_back 可能导致扩容，会使之前保存的 v.begin() 迭代器失效
*it = 0;        // 错！
```

如果不需要连续内存，可以改用分段内存的 deque 容器，其可以保证元素不被移动，迭代器不失效。

```cpp
std::deque<int> v = { 1, 2, 3 };
auto it = v.begin();
v.push_back(4); // deque 的 push_back 不会导致迭代器失效
*it = 0;        // 可以
```

- https://www.geeksforgeeks.org/iterator-invalidation-cpp
- https://en.cppreference.com/w/cpp/container

