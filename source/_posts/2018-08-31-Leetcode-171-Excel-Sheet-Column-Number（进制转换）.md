---
title: Leetcode 171. Excel Sheet Column Number（进制转换）
urlname: leetcode-171-excel-sheet-column-number
toc: true
date: 2018-08-31 10:27:42
updated: 2018-08-31 10:33:42
tags: [Leetcode, alg:Math]
---

题目来源：[https://leetcode.com/problems/excel-sheet-column-number/description/](https://leetcode.com/problems/excel-sheet-column-number/description/)

标记难度：Easy

提交次数：1/1

代码效率：7.82%

## 题意

把Excel的列标题（形如A，AB，AZ……）转换成数字。

## 分析

是个水题。就当26进制去转换就行了。但实际上这并不是26进制，只是看起来很像而已：因为没有0的位置，用`Z`而非`A0`表示26。

## 代码

```cpp
class Solution {
public:
    int titleToNumber(string s) {
        int ans = 0;
        for (char ch: s) {
            ans = ans * 26 + ch - 'A' + 1;
        }
        return ans;
    }
};
```
