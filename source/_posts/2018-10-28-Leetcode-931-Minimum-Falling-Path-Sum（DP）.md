---
title: Leetcode 931. Minimum Falling Path Sum（DP）
urlname: leetcode-931-minimum-falling-path-sum
toc: true
date: 2018-10-28 15:46:09
updated: 2018-10-28 15:55:00
tags: [Leetcode, Leetcode Contest, alg:Dynamic Programming]
---

题目来源：[https://leetcode.com/problems/minimum-falling-path-sum/description/](https://leetcode.com/problems/minimum-falling-path-sum/description/)

标记难度：Medium

提交次数：1/1

代码效率：8ms

## 题意

给定一个二维数组`A`，问经过`A`的falling path（每一层和上一层所处位置的横坐标绝对值最多相差1）经过的值的和的最小值是多少。

## 分析

直接DP之。一个细节是，其实可以不新开一个数组，直接在`A`上DP就行[^solution]……（当然我知道用滚动数组优化也可以）

[^solution]: [Official Solution for Leetcode 931. Minimum Falling Path Sum](https://leetcode.com/problems/minimum-falling-path-sum/solution/)

## 代码

```cpp
class Solution {
public:
    int minFallingPathSum(vector<vector<int>>& A) {
        int N = A.size(), M = A[0].size();
        int f[N][M];
        for (int i = 0; i < M; i++)
            f[0][i] = A[0][i];
        for (int i = 1; i < N; i++) {
            for (int j = 0; j < M; j++) {
                if (j == 0) f[i][j] = A[i][j] + min(f[i-1][j], f[i-1][j+1]);
                else if (j == M - 1) f[i][j] = A[i][j] + min(f[i-1][j], f[i-1][j-1]);
                else {
                    f[i][j] = min(f[i-1][j-1], f[i-1][j+1]);
                    f[i][j] = min(f[i-1][j], f[i][j]);
                    f[i][j] += A[i][j];
                }
            }
        }
        int ans = f[N-1][0];
        for (int i = 0; i < M; i++)
            ans = min(ans, f[N-1][i]);
        return ans;
    }
};
```
