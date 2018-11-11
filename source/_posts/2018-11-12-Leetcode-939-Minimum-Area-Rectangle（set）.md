---
title: Leetcode 939. Minimum Area Rectangle（set）
urlname: leetcode-939-minimum-area-rectangle
toc: true
date: 2018-11-12 00:51:44
updated: 2018-11-12 01:00:00
tags: [Leetcode, Leetcode Contest, alg:Set]
---

题目来源：[https://leetcode.com/problems/minimum-area-rectangle/description/](https://leetcode.com/problems/minimum-area-rectangle/description/)

标记难度：Medium

提交次数：1/1

代码效率：396ms

## 题意

给定若干个xy平面上的点，求这些点能构成的边和x/y轴平行的矩形的面积的最小值。

## 分析

这道题也很简单。直接把所有点放到一个`set`里面，然后对于每一对横纵坐标都不相等（也就是它们可以构成一条对角线）的点，构造出另一条对角线（交换横/纵坐标即可得到），然后检查这两个点是否存在即可。我最开始还排了个序，但事实是不需要排序。

题解提示了我一种hash方法，可以不用`set<pair<int, int>>`，而是改用`40001 * x + y`，也算是一种策略吧。[^solution]

[^solution]: [Official Solution for Leetcode 939](https://leetcode.com/problems/minimum-area-rectangle/solution/)

## 代码

```cpp
class Solution {
public:
    int minAreaRect(vector<vector<int>>& points) {
        set<pair<int, int>> pointSet;
        vector<pair<int, int>> pointPairs;
        for (int i = 0; i < points.size(); i++) {
            pointPairs.emplace_back(points[i][0], points[i][1]);
            pointSet.insert(pointPairs.back());
        }
        sort(pointPairs.begin(), pointPairs.end());
        
        int ans = 1600000001;
        bool found = false;
        for (int i = 0; i < pointPairs.size(); i++) {
            for (int j = i + 1; j < pointPairs.size(); j++) {
                if (pointPairs[i].first == pointPairs[j].first || pointPairs[i].second == pointPairs[j].second)
                    continue;
                pair<int, int> p1 = pointPairs[i], p2 = pointPairs[j]; 
                swap(p1.second, p2.second);
                if (pointSet.find(p1) != pointSet.end() && pointSet.find(p2) != pointSet.end()) {
                    ans = min(ans, abs(pointPairs[i].first - pointPairs[j].first) * abs(pointPairs[i].second - pointPairs[j].second));
                    found = true;
                }
            }
        }
        if (!found) ans = 0;
        return ans;
    }
};
```