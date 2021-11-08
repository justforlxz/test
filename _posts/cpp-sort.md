---
title: 使用标准库std::sort函数进行排序
s: cpp-sort
date: 2019-12-26 17:24:23
tags:
categories:
---

std的sort方法接受两个迭代器begin和end。通过迭代器来抽象元素的访问，隐藏内部实现。

<!-- more -->

这是一个简单的例子:

```
std::list<int> list {
    0,
    4,
    2,
    1,
    3,
};

std::sort(list.begin(), list.end());
```

结果就是list被排序了，至于使用了什么排序算法，我们并不需要关心。实际上标准库会通过元素的数量来决定使用什么算法，基于Introspective Sorting(内省式排序)。
它是一种混合式的排序算法：

- 在数据量很大时采用正常的快速排序，此时效率为O(logN)。

- 一旦分段后的数据量小于某个阈值，就改用插入排序，因为此时这个分段是基本有序的，这时效率可达O(N)。

- 在递归过程中，如果递归层次过深，分割行为有恶化倾向时，它能够自动侦测出来，使用堆排序来处理，在此情况下，使其效率维持在堆排序的O(N logN)，但这又比一开始使用堆排序好。

默认情况下排序是升序排序，既结果从小到大，我们可以通过使用std::equal_to<T>、std::not_equal_to<T>、std::greater<T>、std::less<T>、std::greater_equal<T>和std::less_equal<T>来控制排序。

以上是通过标准库内置的一些方式来控制排序，且适用于元素已实现了自定义比较(Compare)的要求。

比较 (Compare) 是一些标准库设施针对用户提供的函数对象类型所期待的一组要求。

对满足比较 (Compare) 的类型的对象运用函数调用操作的返回值，当按语境转换成 bool 时，若此类型所引入的严格弱序关系中，该调用的第一实参先于第二实参，则生成 true，否则生成 false。

与任何二元谓词 (BinaryPredicate) 相同，不允许该表达式的求值通过解引用的迭代器调用非 const 函数。

用人话来说就是，Compare必须提供出对比结果。

看一个例子:

```
struct Test {
    int i;
}

std::list<Test> list {
    new Test(2),
    new Test(1),
    new Test(4),
    new Test(3),
    new Test(5),
};

std::sort(list.begin(), list.end(), [=] (const Test& test1, const Test& test2) -> bool {
    return test1.i < test2.i;
});
```

这个例子提供了一个Compare，通过lambda来提供自定义的对比函数，返回值必须是bool，否则将不满足对比函数的要求。

通过以上三种方式可以看出，标准库的sort函数可以很方便的为使用者提供标准对比和自定义对比。如果元素自己已实现operator<，则只需要使用标准库内置的对比函数即可，但是大部分情况其实并不会涉及到元素的排序，仅在临时情况下需要列表有序，所以我个人倾向于通过lambda提供Compare函数来完成列表的排序。

[std::sort](https://zh.cppreference.com/w/cpp/algorithm/sort)
[知无涯之std::sort源码剖析](http://feihu.me/blog/2014/sgi-std-sort/)
