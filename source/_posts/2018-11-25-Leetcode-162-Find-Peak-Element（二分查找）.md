---
title: 2018-11-25-Leetcode 162. Find Peak Element（二分查找）
urlname: leetcode-162-find-peak-element
toc: true
date: 2018-11-25 10:20:02
updated: 2018-11-25 13:31:02
tags: [Leetcode, alg:Array, alg:Binary Search]
---

题目来源：[https://leetcode.com/problems/find-peak-element/description/](https://leetcode.com/problems/find-peak-element/description/)

标记难度：Medium

提交次数：2/2

代码效率：

* 线性扫描：8ms
* 二分查找：4ms

## 题意

给定一个数组，其中任意两个相邻元素不等，请返回其中一个peak元素（即比两边元素都大；可以认为`nums[-1]=nums[n]=+inf`）。

要求代码时间复杂度为`O(log(n))`。

## 分析

线性扫描的方法太简单了，不值得细说。不过在写的时候很容易想到一个很有趣的问题：在题设条件下是否保证有这样的peak元素出现呢？我觉得显然是有的。不妨利用反证法：假设这样的peak元素不存在，则对于`nums[0]`和`nums[n-1]`，由于它们旁边各有一个比它们小的元素（`nums[-1]`和`nums[n]`），为了保证peak元素不存在，必有`nums[0] < nums[1]`和`nums[n-2] > nums[n-1]`。然后以此类推，即可推出矛盾。

通过上述方法大概可以想到一种二分的思路。选择`i = n/2`，判断是否满足`nums[i-1] < nums[i] > nums[i+1]`：

* 如果满足，则显然`i`就是满足要求的peak元素
* 否则，如果`nums[i-1] < nums[i] < nums[i+1]`，则必存在一个peak元素位于`[i+1, n-1]`范围中
* 如果`nums[i-1] > nums[i] > nums[i+1]`，则必存在一个peak元素位于`[0, i-1]`范围中
* 如果`nums[i-1] > nums[i] < nums[i+1]`，则i两侧都可能存在peak元素

## 代码

### 线性扫描

```cpp
class Solution {
public:
    int findPeakElement(vector<int>& nums) {
        for (int i = 0; i < nums.size(); i++)
            if ((i == 0 || nums[i] > nums[i-1]) && (i == nums.size() - 1 || nums[i] > nums[i+1]))
                return i;
        return -1;
    }
};
```

### 二分查找

参考了题解[^solution]的写法……题解的细节写得还是挺不错的。

[^solution]: [Leetcode Offcial Solution for 162. Find Peak Element](https://leetcode.com/articles/find-peak-element/)

```cpp
class Solution {
public:
    int findPeakElement(vector<int>& nums) {
        if (nums.size() == 1) return 0;
        int l = 0, r = nums.size() - 1;
        while (l < r) {
            int mid = (l + r) / 2;
            if (nums[mid] > nums[mid + 1]) r = mid;
            else l = mid + 1;
        }
        return l;
    }
};
```