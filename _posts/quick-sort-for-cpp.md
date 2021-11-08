---
title: C++快速排序
p: quick sort for c++
date: 2018-11-11 16:57:44
tags: [
    'Program',
    'C++'
]
categories:
---

快速排序是基于分治思想的排序算法，通过这种策略把列表分为两个子列，重复该过程。是由东尼·霍尔提出，在平均状况下，排序N个数据要O(nlogn)次比较，在最坏情况下则需要O(n^2)，但退化成冒泡的情况比较少见，快速排序比其他排序算法通常情况是最佳的，因为内部使用的循环在很多平台都有优化。

<!-- more -->

快速排序的步骤很简单：

- 选择一个基准
- 遍历列表，将小于基准的放在列表左边，大于基准的放在列表右边
- 递归这个操作

在维基百科上的这张图可以很直观的展示快速排序的过程。

![quicksort](quick-sort-for-cpp/Sorting-quicksort-anim.gif)

代码实现:

首先需要一个返回基准的函数，该函数负责从指定的范围中挑选一个位置作为基准，并对范围内列表进行排序，并返回基准所在的位置。

```cpp
int Division(int a[], int left, int right) {
    int base = a[left]; // 取第一个数为基准
    while (left < right) {
        while (left < right && a[left] > base) {
            // 从右向左找第一个比基准小的元素
            --right;
        }
        a[left] = a[right]; // 交换位置，把小元素放在左侧

        while (left < right && a[left] < base) {
            // 从左向右找第一个比基准大的元素
            ++left;
        }
        a[right] = a[left]; // 交换位置，把大元素放在右侧
    }
    a[left] = base;
    return left;
}
```

Division函数只做了最简单的事，找一个基准，并交换左右的元素，使列表左侧均小于基准元素，使右侧均大于基准元素，接下来需要一个函数，使列表趋向最小，直至列表元素剩一(这里我感觉其实有点极限的思想)。

```
void quick_sort(int a[], int left, int right) {
    if (left < right) {
        int index = Division(a, left, right); //对列表进行分割
        quick_sort(a, left, i -1); //对左侧进行排序
        quick_sort(a, i + 1, right); //对右侧进行排序
    }
}
```

配合上方的gif，就可以很清楚的了解快速排序是如何使用分治法来排序的，通过将大任务拆分成小任务，最终达成完整的排序.
