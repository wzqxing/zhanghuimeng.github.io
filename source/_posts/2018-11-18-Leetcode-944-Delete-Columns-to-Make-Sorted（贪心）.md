---
title: Leetcode 944. Delete Columns to Make Sorted（贪心）
urlname: leetcode-944-delete-columns-to-make-sorted
toc: true
date: 2018-11-18 22:54:18
updated: 2018-11-18 23:00:00
tags: [Leetcode, Leetcode Contest, alg:Greedy]
---

题目来源：[https://leetcode.com/problems/delete-columns-to-make-sorted/description/](https://leetcode.com/problems/delete-columns-to-make-sorted/description/)

标记难度：Medium

提交次数：1/1

代码效率：68ms

## 题意

有`N`个只包含小写字母的长度相同的字符串。把它们从上到下排成一个矩阵，从中删除一些列，使得剩下的列都是从上到下非降序的。问最少删除几列。

## 分析

明白这道题的题意的时候我都惊了。我本来以为这是一个多字符串的最长不降子序列问题呢……那还有点意思。结果竟然是这样。那就直接把不是非降序的列全都删掉就好了！

（为什么这种题可以被标成Medium啊）

## 代码

```cpp
class Solution {
public:
    int minDeletionSize(vector<string>& A) {
        int n = A.size(), m = A[0].size();
        int ans = 0;
        for (int i = 0; i < m; i++) {
            bool inc = true;
            for (int j = 1; j < n; j++)
                if (A[j-1][i] > A[j][i]) {
                    inc = false;
                    break;
                }
            if (!inc) ans++;
        }
        return ans;
    }
};
```