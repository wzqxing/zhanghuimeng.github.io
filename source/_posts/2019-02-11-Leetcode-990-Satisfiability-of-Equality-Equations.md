---
title: Leetcode 990. Satisfiability of Equality Equations
urlname: leetcode-990-satisfiability-of-equality-equations
toc: true
date: 2019-02-11 14:52:31
updated: 2019-02-11 14:52:31
tags: [Leetcode, Leetcode Contest, alg:Depth-first Search]
categories: Leetcode
---

题目来源：[https://leetcode.com/problems/satisfiability-of-equality-equations/description/](https://leetcode.com/problems/satisfiability-of-equality-equations/description/)

标记难度：Medium

提交次数：1/1

代码效率：16ms

## 题意

给定一系列等式，每个等式形如`a==b`或`a!=b`，变量名为单个英文小写字母，问这些等式组能否成立？

## 分析

这道题的思路很简单：首先将所有`a==b`等式转化成`a`和`b`之间的连边，然后做并查集或DFS，然后再判断形如`a!=b`的等式中的两个变量是否在同一个连通集中。总之用并查集和DFS都差不多……

## 代码

所以我就直接写了并查集……

```cpp
class Solution {
private:
    int _fa[26];
    
    void init() {
        for (int i = 0; i < 26; i++)
            _fa[i] = i;
    }
    
    int fa(int x) {
        if (_fa[x] == x) return x;
        return _fa[x] = fa(_fa[x]);
    }
    
    void merge(int x, int y) {
        x = fa(x);
        y = fa(y);
        _fa[x] = y;
    }
    
public:
    bool equationsPossible(vector<string>& equations) {
        init();
        for (string s: equations) {
            if (s[1] == '=')
                merge(s[0] - 'a', s[3] - 'a');
        }
        for (string s: equations) {
            if (s[1] == '!')
                if (fa(s[0] - 'a') == fa(s[3] - 'a'))
                    return false;
        }
        return true;
    }
};
```