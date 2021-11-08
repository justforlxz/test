---
title: leetcode 1. 两数之和
date: 2021-07-06 16:50:11
tags: ['数组', '算法', '简单']
categories: 'leetcode算法'
---

## [1. 两数之和](https://leetcode-cn.com/problems/two-sum/)

给定一个整数数组 `nums` 和一个整数目标值 `target`，请你在该数组中找出 和为目标值 `target`  的那 两个 整数，并返回它们的数组下标。

你可以假设每种输入只会对应一个答案。但是，数组中同一个元素在答案里不能重复出现。

你可以按任意顺序返回答案。

**示例 1：**

```text
输入：nums = [2,7,11,15], target = 9
输出：[0,1]
解释：因为 nums[0] + nums[1] == 9 ，返回 [0, 1] 。
```

**示例 2：**

```text
输入：nums = [3,2,4], target = 6
输出：[1,2]
```

**示例 3：**

```text
输入：nums = [3,3], target = 6
输出：[0,1]
```

**提示：**

- 2 <= nums.length <= 104
- -109 <= nums[i] <= 109
- -109 <= target <= 109
- 只会存在一个有效答案

**进阶：** 你可以想出一个时间复杂度小于 O(n2) 的算法吗？

## 算法 1

**解题思路：**

这道题应该是大家的leetcode入门题了，做法非常的简单，核心在于使用 unordered map 保存 `target` 和 遍历值的差，没有在 map 中找到差就将结果保存到 map 中。

```c++
std::vector<int> twoSum(std::vector<int>& nums, int target) {
    std::unordered_map<int, int> hashtable;
    for (int i = 0; i < nums.size(); ++i) {
        auto it = hashtable.find(target - nums[i]);
        if (it != hashtable.end()) {
            return {it->second, i};
        }
        hashtable[nums[i]] = i;
    }
    return {};
}
```
