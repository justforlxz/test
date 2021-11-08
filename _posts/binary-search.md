---
title: leetcode 704. 二分查找
date: 2021-07-06 16:35:19
tags: ['数组', '算法', '简单']
categories: 'leetcode算法'
---

## [704. 二分查找](https://leetcode-cn.com/problems/binary-search/)

给定一个 n 个元素有序的（升序）整型数组 nums 和一个目标值 target  ，写一个函数搜索 nums 中的 target，如果目标值存在返回下标，否则返回 -1。

示例 1:

```text
输入: nums = [-1,0,3,5,9,12], target = 9
输出: 4
解释: 9 出现在 nums 中并且下标为 4
```

示例 2:

```text
输入: nums = [-1,0,3,5,9,12], target = 2
输出: -1
解释: 2 不存在 nums 中因此返回 -1
```

提示：

你可以假设 nums 中的所有元素是不重复的。
n 将在 [1, 10000]之间。
nums 的每个元素都将在 [-9999, 9999]之间。

## 解题思路

使用双指针实现二分查找。

初始化 `left` 为0, `right` 为 nums的长度减一，在每次遍历的时候，计算 `mid` 为中间值，计算方法如下：

```text
mid = left + (right - left) / 2;
```

当 `nums[mid] == target` 时，即查找到目标值，返回 `mid`。
当 `nums[mid] < target` 时，调整 `right` 为 `mid - 1`。
当 `nums[mid] > target` 时，调整 `left` 为 `mid + 1`。

循环条件为 `left <= right`，当超过中间值仍然没有查找到目标值，`left` 会超过 `right` 的位置，从而跳出循环，返回 `-1`。

详细实现：

```c++
int search(const std::vector<int>& nums, int target) {
    if (nums.size() < 2) {
        return nums[0] == target ? 0 : -1;
    }

    int mid = 0;
    int left = 0;
    int right = nums.size() -1;

    while (left <= right) {
        mid = left + (right - left) / 2;
        if (nums[mid] == target) {
            return mid;
        }
        if (target < nums[mid]) {
            right = mid - 1;
        }
        else {
            left = mid + 1;
        }
    }

    return -1;
}
```
