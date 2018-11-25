---
title: Leetcode 946. Validate Stack Sequences（栈）
urlname: leetcode-946-validate-stack-sequences
toc: true
date: 2018-11-25 15:09:53
updated: 2018-11-25 15:17:00
tags: [Leetcode, Leetcode Contest, alg:Stack]
---

题目来源：[https://leetcode.com/problems/validate-stack-sequences/description/](https://leetcode.com/problems/validate-stack-sequences/description/)

标记难度：Medium

提交次数：1/1

代码效率：4ms

## 题意

给定两个值两两不同的数组（`pushed`和`popped`），问它们能否是对一个初始为空的栈进行一系列操作的结果。

## 分析

一个水题，直接跟着这两个数组模拟就好了，模拟得出来就是可以，模拟不出来就是不行。（简直梦回NOIP普及组笔试题啊……）

## 代码

```cpp
class Solution {
public:
    bool validateStackSequences(vector<int>& pushed, vector<int>& popped) {
        int n = pushed.size();
        stack<int> s;
        int i = 0, j = 0;
        while (i < n && j < n) {
            while ((s.empty() || s.top() != popped[j]) && i < n)
                s.push(pushed[i++]);
            if (s.empty()) break;
            while (!s.empty() && j < n && s.top() == popped[j]) {
                s.pop();
                j++;
            }
        }
        return i == n && j == n;
    }
};
```