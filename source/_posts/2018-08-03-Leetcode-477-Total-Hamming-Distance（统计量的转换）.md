---
title: Leetcode 477. Total Hamming Distance（统计量的转换）
urlname: leetcode-477-total-hamming-distance
toc: true
date: 2018-08-03 23:43:19
updated: 2018-08-03 23:52:00
tags: [Leetcode]
---

题目来源：[https://leetcode.com/problems/total-hamming-distance/description/](https://leetcode.com/problems/total-hamming-distance/description/)

标记难度：Easy

提交次数：1/1

代码效率：39.56%

## 题意

给定若干个数，计算其中所有数对的海明（Hamming）距离之和。

## 分析

如果直接计算的话，`O(N^2)`的代价是不可忍受的。然后就很自然地想到，我们把这个问题横向转化一下：既然海明距离计算的只是每个对应二进制位的差异，那么，我们可以直接对所有数的这一位进行统计，得到`0`和`1`的个数，然后计算这一位对整体距离的贡献：`count(0) * count(1)`。

我感觉这种思路的转换方法很常见，但是我一时想不起来其他的例子了，大概最近刷题太少了吧。

## 代码

```
class Solution {
public:
    int totalHammingDistance(vector<int>& nums) {
        int zeroBits[31];
        memset(zeroBits, 0, sizeof(zeroBits));
        for (int num: nums) {
            for (int i = 0; i < 31; i++) {
                if ((num & (1 << i)) == 0)
                    zeroBits[i]++;
            }
        }
        int ans = 0, n = nums.size();
        for (int i = 0; i < 31; i++)
            ans += zeroBits[i] * (n - zeroBits[i]);

        return ans;
    }
};
```
