---
title: Leetcode 829. Consecutive Numbers Sum（数列）
urlname: leetcode-829-consecutive-numbers-sum
toc: true
date: 2018-08-05 18:43:29
updated: 2018-08-05 18:43:29
tags: [Leetcode]
---

题目来源：[https://leetcode.com/problems/consecutive-numbers-sum/description/](https://leetcode.com/problems/consecutive-numbers-sum/description/)

标记难度：Medium

提交次数：1/2

代码效率：98.33%

## 题意

给定一个正整数`N`，问有多少种把`N`表示成若干个连续正整数之和的方法。

## 分析

我感觉这道题还挺好想的。既然要把`N`表示成一个等差数列（公差为1）的和，我们不妨设这个数列的首项是`x`，项数为`m`，则这个数列的和就是`[x + (x + (m-1))]m / 2 = mx + m(m-1)/2 = N`。接下来，一个很自然的想法就是，枚举`m`，通过上式判断对于相应的`m`是否存在合法的`x`。显然枚举的复杂度是`O(sqrt(N))`。

## 代码

错了一次是因为忘记把计数器初始化了。

```cpp
class Solution {
public:
    int consecutiveNumbersSum(int N) {
        int ans = 0;
        for (int m = 1; ; m++) {
            int mx = N - m * (m-1) / 2;
            if (mx <= 0)
                break;
            if (mx % m == 0)
                ans++;
        }
        return ans;
    }
};
```
