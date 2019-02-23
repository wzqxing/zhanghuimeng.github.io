---
title: Leetcode 996. Number of Squareful Arrays（DP）
urlname: leetcode-996-number-of-squareful-arrays
toc: true
date: 2019-02-21 00:05:00
updated: 2019-02-21 00:05:00
tags: [Leetcode, Leetcode Contest, alg:Backtracking, alg:Dynamic Porgramming]
categories: Leetcode
---

题目来源：[https://leetcode.com/problems/number-of-squareful-arrays/description/](https://leetcode.com/problems/number-of-squareful-arrays/description/)

标记难度：Hard

提交次数：1/1

代码效率：36.82%（12ms）

## 题意

给定数组`A`，问有多少种`A`的排列，使得任意两个相邻元素的和都是平方数。认为两种排列不等当且仅当存在某个位置的元素不等（而不是把不同index的相同元素认为是不同的）。

## 分析

这又是一道“虽然看起来好像应该用状态DP做但因为数据量太小了所以不如直接DFS搜索”的题。既然我都写了DP了，现在实在懒得去写DFS，就这样好了。

## 代码

```cpp
class Solution {
private:
    bool isSquare(int x) {
        int s = (int) sqrt(x);
        if (s*s == x) return true;
        return false;
    }
    
    int f[1 << 12][12];
    vector<int> A;
    bool ok[12][12];
    int n;
    
    int calc(int state, int end) {
        if (f[state][end] != -1) return f[state][end];
        if (!(state & (1 << end))) {
            f[state][end] = 0;
            return 0;
        }
        if (__builtin_popcount(state) == 1) {
            f[state][end] = 1;
            return 1;
        }
        f[state][end] = 0;
        for (int i = 0; i < n; i++) {
            if (i == end || !ok[i][end] || !(state & (1 << i))) continue;
            f[state][end] += calc(state ^ (1 << end), i);
        }
        return f[state][end];
    }
    
    int factorial(int x) {
        int ans = 1;
        while (x > 0) {
            ans *= x;
            x--;
        }
        return ans;
    }
    
public:
    int numSquarefulPerms(vector<int>& A) {
        sort(A.begin(), A.end());
        this->A = A;
        n = A.size();
        for (int i = 0; i < n; i++) {
            ok[i][i] = false;
            for (int j = i + 1; j < n; j++) {
                if (isSquare(A[i] + A[j]))
                    ok[i][j] = ok[j][i] = true;
                else
                    ok[i][j] = ok[j][i] = false;
            }
        }
        
        memset(f, -1, sizeof(f));
        int ans = 0;
        for (int i = 0; i < n; i++)
            ans += calc((1 << n) - 1, i);
        
        map<int, int> cnt;
        for (int x: A) cnt[x]++;
        for (auto p: cnt) ans /= factorial(p.second);
        
        return ans;
    }
};
```