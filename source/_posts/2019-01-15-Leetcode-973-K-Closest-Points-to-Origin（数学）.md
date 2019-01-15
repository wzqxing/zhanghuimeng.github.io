---
title: Leetcode 973. K Closest Points to Origin（数学）
urlname: leetcode-973-k-closest-points-to-origin
toc: true
date: 2019-01-15 17:13:52
updated: 2019-01-15 21:24:00
tags: [Leetcode, Leetcode Contest, alg:Math]
---

题目来源：[https://leetcode.com/problems/k-closest-points-to-origin/description/](https://leetcode.com/problems/k-closest-points-to-origin/description/)

标记难度：Easy

提交次数：1/1

代码效率：80.0%（208ms）

## 题意

给定平面上的若干个（<=10000）点，求距离原点最近的`K`个点。保证结果对应的点集唯一，以任意顺序返回都可以。

## 分析

显然可以把距离算出来之后排序，然后取前`K`个点，时间复杂度为`O(N*log(N))`。

如果想做得更好一点的话，可以用一个大小为`K`的堆来维护距离前`K`小的点，时间复杂度为`O(N*log(K))`。

## 代码

```cpp
class Solution {
public:
    vector<vector<int>> kClosest(vector<vector<int>>& points, int K) {
        vector<pair<int, int>> toSort;
        for (int i = 0; i < points.size(); i++) {
            toSort.emplace_back(points[i][0] * points[i][0] + points[i][1] * points[i][1], i);
        }
        sort(toSort.begin(), toSort.end());
        vector<vector<int>> ans;
        for (int i = 0; i < K; i++)
            ans.push_back(points[toSort[i].second]);
        return ans;
    }
};
```