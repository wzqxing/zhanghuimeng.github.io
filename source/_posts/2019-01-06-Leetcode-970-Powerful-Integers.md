---
title: Leetcode 970. Powerful Integers
urlname: leetcode-970-powerful-integers
toc: true
date: 2019-01-06 19:37:46
updated: 2019-01-07 00:40:00
tags: [Leetcode, Leetcode Contest]
---

题目来源：[https://leetcode.com/problems/powerful-integers/description/](https://leetcode.com/problems/powerful-integers/description/)

标记难度：Medium

提交次数：1/1

代码效率：4ms

## 题意

给定正整数`x`和`y`（`1 <= x, y <= 100`），求所有在`[1, bound]`（`0 <= bound <= 10^6`）范围内的`x^i + y^j`，要求去重。

## 分析

去掉`x`或`y`为1的情况，由于`log(1000000) / log(2) = 19`，所以可以直接枚举所有`x`和`y`的幂和，然后去重。总的来说很简单。

## 代码

```cpp
class Solution {
public:
    vector<int> powerfulIntegers(int x, int y, int bound) {
        if (x > y) swap(x, y);
        
        if (x == 1 && y == 1) {
            if (bound >= 2) return {2};
            else return {};
        }
        
        if (x == 1) {
            vector<int> ans;
            int yj = 1;
            for (int j = 0; yj <= bound; j++) {
                int t = yj + 1;
                if (t <= bound) ans.push_back(t);
            }
            return ans;
        }
        
        unordered_set<int> numbers;
        vector<int> ans;
        int xi = 1;
        for (int i = 0; xi <= bound; i++) {
            int yj = 1;
            for (int j = 0; yj <= bound; j++) {
                int t = xi + yj;
                if (t <= bound && numbers.find(t) == numbers.end()) {
                    numbers.insert(t);
                    ans.push_back(t);
                }
                yj *= y;
            }
            xi *= x;
        }
        return ans;
    }
};
```