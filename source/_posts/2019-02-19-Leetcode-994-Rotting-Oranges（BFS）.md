---
title: Leetcode 994. Rotting Oranges（BFS）
urlname: leetcode-994-rotting-oranges
toc: true
date: 2019-02-19 21:38:19
updated: 2019-02-20 23:55:00
tags: [Leetcode, Leetcode Contest, alg:Breadth-first Search]
categories: Leetcode
---

题目来源：[https://leetcode.com/problems/cousins-in-binary-tree/description/](https://leetcode.com/problems/cousins-in-binary-tree/description/)

标记难度：Easy

提交次数：1/1

代码效率：12ms

## 题意

## 分析

前几天在CF上做过一道有点类似的题（[1105D](https://codeforces.com/problemset/problem/1105/D)，也是BFS分次扩展，当时虽然过了pretest，却因为每次扩展的结点过多且`queue`初始化过慢超时了。所以我就学到了一个道理：在这种情况下注意到底应该扩展哪些结点。不过这道题其实没有这个必要……

考虑到这一点，不如直接用`vector`代替`queue`。

## 代码

```cpp
class Solution {
public:
    int orangesRotting(vector<vector<int>>& grid) {
        vector<pair<int, int>> q;
        int orangeCnt = 0;
        int n = grid.size(), m = grid[0].size();
        int mx[4] = {0, 0, 1, -1};
        int my[4] = {1, -1, 0, 0};
        
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                if (grid[i][j] != 0) orangeCnt++;
                if (grid[i][j] == 2) q.emplace_back(i, j);
            }
        }
        
        if (q.size() == orangeCnt) return 0;  // 全都烂了
        if (q.size() == 0) return -1;  // 根本没有烂的
        // [lastEnd, lastSize)这部分是本次需要扩展的
        // 之前的已经扩展过了，没有再扩展一次的必要
        int lastEnd = 0, lastSize = q.size();
        for (int i = 1; ; i++) {
            for (int j = lastEnd; j < lastSize; j++) {
                int x = q[j].first, y = q[j].second;
                for (int k = 0; k < 4; k++) {
                    int nx = x + mx[k], ny = y + my[k];
                    if (nx < 0 || nx >= n || ny < 0 || ny >= m) continue;
                    if (grid[nx][ny] == 1) {
                        grid[nx][ny] = 2;
                        q.emplace_back(nx, ny);
                    }
                }
            }
            // 本次没有扩展出新的烂橘子，且还有橘子没烂，说明扩展不到那边了
            if (lastSize == q.size()) return -1;
            // 所有橘子都烂了
            if (q.size() == orangeCnt) return i;
            lastEnd = lastSize;
            lastSize = q.size();
        }
        return 0;
    }
};
```
