---
title: Leetcode 357. Count Numbers with Unique Digits（组合数学）
urlname: leetcode-357-count-numbers-with-unique-digits
toc: true
date: 2018-07-31 23:23:46
updated: 2018-07-31 23:28:00
mathjax: true
tags: [Leetcode, alg:Math]
---

题目来源：[https://leetcode.com/problems/count-numbers-with-unique-digits/description/](https://leetcode.com/problems/count-numbers-with-unique-digits/description/)

标记难度：Easy

提交次数：2/5

代码效率：100.00%

## 题意

给定**非负整数**$n$，计算$[0, 10^n]$中没有重复数字的数的个数。

## 分析

这道题的数据量非常小（显然，n>10时的个数与n=10时是相同的），因此，可以直接打表计算。

## 代码

结果错了好多次……

两个边界情况：

* n=0时应该返回1
* n>10时应该返回8877691。虽然我开始时返回0也过了……

```
class Solution {
public:
    int countNumbersWithUniqueDigits(int n) {
        // 显然，n<=10
        // 所以干脆直接打表算了。
        // 下面的n指的是，这个数长度为n（因此0不能在第一位）
        // n = 1: 10
        // n = 2: 9 * 9 = 81
        // n = 3: 9 * 9 * 8 = 648
        // n = 4: 9 * 9 * 8 * 7 = 4536
        // n = 5: 9 * 9 * 8 * 7 * 6 = 27216
        // n = 6: 9 * 9 * 8 * 7 * 6 * 5 = 136080
        // n = 7: 9 * 9 * 8 * 7 * 6 * 5 * 4 = 544320
        // n = 8: 9 * 9 * 8 * 7 * 6 * 5 * 4 * 3 = 1632960
        // n = 9: 9 * 9 * 8 * 7 * 6 * 5 * 4 * 3 * 2 = 3265920
        // n = 10: 9 * 9 * 8 * 7 * 6 * 5 * 4 * 3 * 2 * 1 = 3265920
        if (n == 0) return 1;
        if (n == 1) return 10;
        if (n == 2) return 91;
        if (n == 3) return 739;
        if (n == 4) return 5275;
        if (n == 5) return 32491;
        if (n == 6) return 168571;
        if (n == 7) return 712891;
        if (n == 8) return 2345851;
        if (n == 9) return 3265920;
        if (n >= 10) return 8877691;
        return 0;
    }
};
```
