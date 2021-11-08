---
title: C plus plus Iterator 笔记
date: 2018-07-09 13:05:12
tags: [
    C++
]
categories: C++
---

本文记录了咱对迭代器的一些理解

C++ 标准库提供了三种类型组件:

- 容器
- 迭代器
- 算法

容器是指存储某种类型的结构，容器有两种:

- 顺序容器 (vector、list和string等，是元素的有序集合。)
- 关联容器 (set、map等，是包含查找元素的键值。 )

遍历容器的方式之一就是迭代器，迭代器是一种泛型指针，普通指针指向一块内存，迭代器指向容器中的一个位置。STL的每个模板容器中，都定义了一组对应的迭代器类，使用迭代器和算法，就可以访问容器中特定位置的元素，而无需关心元素的类型。

每种容器都定义了一对begin和end的函数，用于返回迭代器。如果容器中有元素的话，begin返回的迭代器指向第一个元素。

<!-- more -->

```cpp
std::list<int>::iterator it = list.begin();
```

上述语句把it初始化为由list的begin返回的迭代器，如果list不为空，it将指向该元素list[0]。

由end操作返回的迭代器指向list的末端元素的下一个，通常指超出末端迭代器(off-the-end-iterator)，表明指向一个不存在的元素，如果容器为空，begin返回的迭代器将和end相同，在使用中，可以通过判断end来检查是否处理完容器种所有的元素。

迭代器类型定义了一些操作来获取迭代器所指向的元素，并允许程序员将迭代器从一个元素移动到另一个元素。

遍历列表：

```cpp
std::list<int> list

for (std::list<int>::const_iterator it = list.constBegin(); it != list.constEnd(); ++it) {
    // 通过迭代器访问元素需要解引用。
    std::cout << *it << std::endl;
}
```

```cpp
std::list<int> list;
std::sort(list.begin(), list.end(), [=] (int _i1, int _i2) {
    return _i1 < _i2;
});
```

上面的示例代码是对一个int类型的list进行排序，
