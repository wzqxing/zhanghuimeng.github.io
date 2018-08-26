---
title: Leetcode 892. Surface Area of 3D Shapes（枚举），及周赛（99）总结
urlname: leetcode-892-surface-area-of-3d-shapes-and-weekly-contest-99
toc: true
date: 2018-08-26 17:41:41
updated: 2018-08-26 19:19:00
tags: [Leetcode, LeetcodeContest]
---

题目来源：[https://leetcode.com/problems/surface-area-of-3d-shapes/description/](https://leetcode.com/problems/surface-area-of-3d-shapes/description/)

标记难度：Easy

提交次数：3/4

代码效率：

* 按立方体枚举：8ms
* 按柱体枚举：4ms

## 题意

给定一个`N*N`的平面网格，以及每个网格上堆积了多少个立方体，求整体表面积。

## 分析

这是我参加的第三次周赛，排名稳步下降（683 / 3877），因为这次我只做出来两道题，然后就卡在第三道上了。事后发现，我的整个解法可能都是不可取的；以及我觉得第四题比第三题简单，比赛结束之后用STL随便拼了拼就做出来了，花了不到半个小时。

这也许说明不应该对着一道题死磕。不过我刷的题量确实还不够，连Medium刷得都不够。

这次比赛居然是两道Easy，一道Medium。

---

这道题的数据量只有`N <= 50`，`grid[i][j] <= 50`，所以很容易想到一种`O(N^2 * max(grid[i][j]))`的算法：枚举每个立方体，判断它的各面是否被周围的立方体遮住了，然后把没被遮住的部分加起来。[题解](https://leetcode.com/problems/surface-area-of-3d-shapes/solution/)的做法也是这样的；比赛的时候我就直接这么做了，花了大约8分钟。

事实上很容易想到一种优化方法。我们完全没有必要考虑每一个立方体有几个面被遮住了，只需要考虑对于每个柱子，有多少个面被遮住即可。[^tower]

[^tower]: [Minus Hidden Area](https://leetcode.com/problems/surface-area-of-3d-shapes/discuss/163414/C++Java1-line-Python-Minus-Hidden-Area)

## 代码

### 按立方体枚举

```cpp
class Solution {
public:
    int surfaceArea(vector<vector<int>>& grid) {
        int N = grid.size();
        int sum = 0;
        // 遍历每一个方块
        for (int i = 0; i < N; i++) {
            for (int j = 0; j < N; j++) {
                if (grid[i][j] > 0)
                    sum += 2; // 上下面
                for (int k = 0; k < grid[i][j]; k++) {
                    if (i == 0 || i > 0 && grid[i - 1][j] <= k)
                        sum++;
                    if (i == N - 1 || i < N - 1 && grid[i + 1][j] <= k)
                        sum++;
                    if (j == 0 || j > 0 && grid[i][j - 1] <= k)
                        sum++;
                    if (j == N - 1 || j < N - 1 && grid[i][j + 1] <= k)
                        sum++;
                }
            }
        }
        return sum;
    }
};
```

### 按柱体枚举

```cpp
class Solution {
public:
    int surfaceArea(vector<vector<int>>& grid) {
        int sum = 0, N = grid.size();
        for (int i = 0; i < N; i++) {
            for (int j = 0; j < N; j++) {
                if (grid[i][j] > 0)
                    sum += grid[i][j] * 4 + 2;
                /*
                // 这是一种更简洁的方法：每两个贴在一起的面只用考虑一次
                // 只考虑每个柱子左边和上边消掉的面
                // ref: https://leetcode.com/problems/surface-area-of-3d-shapes/discuss/163414/C++Java1-line-Python-Minus-Hidden-Area
                if (i > 0)
                    sum -= min(grid[i][j], grid[i-1][j]) * 2;
                if (j > 0)
                    sum -= min(grid[i][j], grid[i][j-1]) * 2;
                */
                if (i > 0) sum -= min(grid[i][j], grid[i-1][j]);
                if (i < N - 1) sum -= min(grid[i][j], grid[i+1][j]);
                if (j > 0) sum -= min(grid[i][j], grid[i][j-1]);
                if (j < N - 1) sum -= min(grid[i][j], grid[i][j+1]);
            }
        }

        return sum;
    }
};
```
