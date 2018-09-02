---
title: Leetcode 88. Merge Sorted Array（归并）
urlname: leetcode-88-merge-sorted-array
toc: true
date: 2018-08-08 20:16:07
updated: 2018-08-08 20:24:00
tags: [Leetcode, alg:Array, alg:Two Pointers]
---

题目来源：[https://leetcode.com/problems/merge-sorted-array/description/](https://leetcode.com/problems/merge-sorted-array/description/)

标记难度：Easy

提交次数：1/1

代码效率：

* 非原地版本：98.65%
* 原地版本：98.65%

## 题意

合并两个已排序的数组，把排序结果存放到第一个数组中（保证空间足够）。

## 分析

这是一道水题。不过其实需要一点点的思考。非原址的合并方法完全不需要思考。但是如果要求原址合并，那么从前向后合并会导致合并结果会覆盖第一个数组的数据。那么从后向前合并就可以了。被覆盖的数据必然是已经移动过的。这种操作数组的思路倒是有点类似于快排。

## 代码

### 非原地版本

```cpp
class Solution {
public:
    void merge(vector<int>& nums1, int m, vector<int>& nums2, int n) {
        // 不知道怎么进行原址merge，干脆新开一个vector
        // 从后向前应该可以保证不冲突？
        vector<int> nums3;
        int i = 0, j = 0;
        while (i < m && j < n) {
            if (nums1[i] < nums2[j])
                nums3.push_back(nums1[i++]);
            else
                nums3.push_back(nums2[j++]);
        }
        while (i < m)
            nums3.push_back(nums1[i++]);
        while (j < n)
            nums3.push_back(nums2[j++]);
        for (int k = 0; k < nums3.size(); k++)
            nums1[k] = nums3[k];
    }
};
```

### 原地版本

```cpp
class Solution {
public:
    void merge(vector<int>& nums1, int m, vector<int>& nums2, int n) {
        vector<int> nums3;
        int i = m - 1, j = n - 1, end = m + n - 1;
        while (i >= 0 && j >= 0) {
            if (nums1[i] > nums2[j])
                nums1[end--] = nums1[i--];
            else
                nums1[end--] = nums2[j--];
        }
        while (j >= 0)
            nums1[end--] = nums2[j--];
    }
};
```
