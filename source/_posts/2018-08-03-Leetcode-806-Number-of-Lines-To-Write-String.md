---
title: Leetcode 806. Number of Lines To Write String（模拟）
urlname: leetcode-806-number-of-lines-to-write-string
toc: true
date: 2018-08-03 23:07:12
updated: 2018-08-03 23:09:00
tags: [Leetcode]
---

题目来源：[https://leetcode.com/problems/number-of-lines-to-write-string/description/](https://leetcode.com/problems/number-of-lines-to-write-string/description/)

标记难度：Easy

提交次数：1/1

代码效率：100.00%

## 题意

有一个数组的元素，每个元素有一个宽度，需要把它们按顺序分成若干行，每一行的宽度最大为100，问至少需要多少行，最后一行的实际宽度是多少。

## 分析

超级大水题，直接模拟就行。

## 代码

```
class Solution {
public:
    vector<int> numberOfLines(vector<int>& widths, string S) {
        int lines = 1, width = 0;
        for (char s: S) {
            if (width + widths[s - 'a'] <= 100) {
                width += widths[s - 'a'];
            }
            else {
                lines++;
                width = widths[s - 'a'];
            }
        }
        vector<int> ans;
        ans.push_back(lines);
        ans.push_back(width);
        return ans;
    }
};
```
