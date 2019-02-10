---
title: Leetcode 989. Add to Array-Form of Integer
urlname: leetcode-989-add-to-array-form-of-integer
toc: true
date: 2019-02-10 20:49:07
updated: 2019-02-10 20:49:07
tags: [Leetcode, Leetcode Contest, alg:Array]
categories: Leetcode
---

题目来源：[https://leetcode.com/problems/add-to-array-form-of-integer/description/](https://leetcode.com/problems/add-to-array-form-of-integer/description/)

标记难度：Easy

提交次数：1/1

代码效率：144ms

## 题意

给定一个自然数的各个数位从左到右的数组表示和另一个自然数，求这两个数的和的数组表示。

## 分析

很简单的加法题的一个小变形。题解的做法跟我差不多：开一个新的数组（因为原来的表示方法不符合一般从右往左表示的规律），从最后一位开始加，最后再把数组倒过来。

[^sln]: [Leetcode Official Solution for 989. Add to Array-Form of Integer](https://leetcode.com/problems/add-to-array-form-of-integer/solution/)

## 代码

```cpp
class Solution {
public:
    vector<int> addToArrayForm(vector<int>& A, int K) {
        vector<int> a;
        a.push_back(0);
        for (int i = A.size() - 1; i >= 0; i--) {
            a.back() += A[i];
            a.back() += K % 10;
            K /= 10;
            int x = a.back() / 10;
            a.back() %= 10;
            a.push_back(x);
        }
        while (K > 0) {
            a.back() += K % 10;
            K /= 10;
            int x = a.back() / 10;
            a.back() %= 10;
            a.push_back(x);
        }
        while (a.size() > 1 && a.back() == 0) a.pop_back();
        reverse(a.begin(), a.end());
        return a;
    }
};
```
