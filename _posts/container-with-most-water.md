---
title: leetcode 11. 盛最多水的容器
date: 2021-07-06 17:40:18
tags: ['数组', '算法', '简单']
categories: 'leetcode算法'
---

## [盛最多水的容器](https://leetcode-cn.com/problems/container-with-most-water/)

给你 `n` 个非负整数 `a1，a2，...，an`，每个数代表坐标中的一个点 `(i, ai)` 。在坐标内画 `n` 条垂直线，垂直线 `i` 的两个端点分别为 `(i, ai)` 和 `(i, 0)` 。找出其中的两条线，使得它们与 `x` 轴共同构成的容器可以容纳最多的水。

**说明：你不能倾斜容器。**

**示例 1：**

![示例1](https://aliyun-lc-upload.oss-cn-hangzhou.aliyuncs.com/aliyun-lc-upload/uploads/2018/07/25/question_11.jpg)

```text
输入：[1,8,6,2,5,4,8,3,7]
输出：49
解释：图中垂直线代表输入数组 [1,8,6,2,5,4,8,3,7]。在此情况下，容器能够容纳水（表示为蓝色部分）的最大值为 49。
```

**示例 2：**

```text
输入：height = [1,1]
输出：1
```

**示例 3：**

```text
输入：height = [4,3,2,1,4]
输出：16
```

**示例 4：**

```text
输入：height = [1,2,1]
输出：2
```

**提示：**

- n = height.length
- 2 <= n <= 3 * 104
- 0 <= height[i] <= 3 * 104

## 算法1

**解题思路：**

使用双指针进行解题，本题的核心点在于，容纳的水量为

`两个指针指向的数字中较小值 ∗ 指针之间的距离`

所以，使用双指针标记区间范围，缓存面积的值，每次计算都求出当前区间范围的面积，并求出缓存的面积值和当前的面积值的最大值，更新缓存。之后再从最小值开始移动指针，求下一次的面积。

```c++
int maxArea(const std::vector<int> &height) {
    size_t lp = 0;
    size_t rp = height.size() - 1;
    size_t ans = 0;
    while (lp < rp) {
        size_t area = std::min(height[lp], height[rp]) * (rp - lp);
        ans = std::max(ans, area);
        if (height[lp] < height[rp]) {
            ++lp;
        }
        else {
            --rp;
        }
    }

    return ans;
}
```
