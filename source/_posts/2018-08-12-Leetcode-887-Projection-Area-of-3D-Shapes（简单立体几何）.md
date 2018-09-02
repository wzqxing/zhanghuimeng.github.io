---
title: Leetcode 887. Projection Area of 3D Shapes（简单立体几何）
urlname: leetcode-887-projection-area-of-3d-shapes
toc: true
date: 2018-08-12 01:37:41
updated: 2018-08-12 01:42:41
tags: [Leetcode, alg:Math]
---

题目来源：[https://leetcode.com/problems/projection-area-of-3d-shapes/description/](https://leetcode.com/problems/projection-area-of-3d-shapes/description/)

标记难度：Easy

提交次数：1/1

代码效率：99.84%

## 题意

计算一堆方块的三视图投影面积总和。

## 分析

大水题。

我的代码有点复杂了，考虑到题目说明了地板的大小是`N * N`，完全可以一遍统计出三视图的投影面积总和（因为此时前面和侧面是完全对称的）。（[11 line 1 pass Java code and  explanation of the problem, time O(N ^ 2) space O(1).](https://leetcode.com/problems/projection-area-of-3d-shapes/discuss/156771/11-line-1-pass-Java-code-and-explanation-of-the-problem-time-O%28N-2%29-space-O%281%29.)）

## 代码

```cpp
class Solution {
public:
    int projectionArea(vector<vector<int>>& grid) {
        // 顶面，直接统计有哪些方块不是0即可
        int up = 0;
        for (int i = 0; i < grid.size(); i++)
            for (int x: grid[i])
                if (x != 0)
                    up++;

        // 前面，统计每一列高度的最大值
        int front = 0;
        vector<int> maxHeight(grid.size(), 0);
        for (int i = 0; i < grid.size(); i++) {
            for (int x: grid[i])
                maxHeight[i] = max(maxHeight[i], x);
            front += maxHeight[i];
        }

        int left = 0;
        vector<int> maxHeight2(grid[0].size(), 0);
        for (int i = 0; i < grid.size(); i++) {
            for (int j = 0; j < grid[i].size(); j++)
                maxHeight2[j] = max(maxHeight2[j], grid[i][j]);
        }
        for (int h: maxHeight2)
            left += h;

        return up + front + left;
    }
};
```
