---
title: Leetcode 462. Minimum Moves to Equal Array Elements II（L1范数性质）
urlname: leetcode-462-minimum-moves-to-equal-array-elements-2
toc: true
date: 2018-08-03 21:16:11
updated: 2018-08-03 21:45:00
mathjax: true
tags: [Leetcode, alg:Math]
---

题目来源：[https://leetcode.com/problems/minimum-moves-to-equal-array-elements-ii/description/](https://leetcode.com/problems/minimum-moves-to-equal-array-elements-ii/description/)

标记难度：Medium

提交次数：1/2

代码效率：79.95%

## 题意

$\min \sum_{i=1}^{N} |s_i - x|$

## 分析

我最开始做这道题的时候走了很多弯路，比如想用平均值和二分法。但实际上正确的解法是求中位数。详细解释可以看[The Median Minimizes the Sum of Absolute Deviations (The L1 Norm)](https://math.stackexchange.com/questions/113270/the-median-minimizes-the-sum-of-absolute-deviations-the-l-1-norm)。

从直观上来说，可以想象$x$的值在数轴上移动，如果它左边有$m$个数，右边有$n$个数，此时如果$x$向左移动$\epsilon$（没有越过其他的数值），它到左边的数的总距离就会减少$m \epsilon$，到右边的数的总距离就会增加$n \epsilon$，对总和的贡献就是$(n-m) \epsilon$。可以看出，在$n < m$的情况下，我们总是应该向左移动。向右同理；总之大概可以推断出，x是中位数时这个结果是最小的。

另一种推导方法是直接取导数。

## 代码

```
class Solution {
private:
    int calcMoves(vector<int>& nums, int x) {
        int sum = 0;
        for (int num: nums)
            sum += abs(num - x);
        return sum;
    }

public:
    int minMoves2(vector<int>& nums) {
        if (nums.size() == 0)
            return 0;

        sort(nums.begin(), nums.end());

        return calcMoves(nums, nums[nums.size() / 2]);
    }
};
```
