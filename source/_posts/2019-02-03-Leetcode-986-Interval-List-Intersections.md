---
title: Leetcode 986. Interval List Intersections
urlname: leetcode-986-interval-list-intersections
toc: true
date: 2019-02-04 00:19:53
updated: 2019-02-04 21:55:00
tags: [Leetcode, Leetcode Contest, alg:Two Pointers]
---

题目来源：[https://leetcode.com/problems/interval-list-intersections/description/](https://leetcode.com/problems/interval-list-intersections/description/)

标记难度：Medium

提交次数：2/2

代码效率：

* 奇怪的做法：28ms
* 正常的合并：24ms

## 题意

有`A`和`B`两个闭区间数组，它们分别都是排过序的，且每个数组里的区间之间互不相交。请求出两个数组中闭区间的交。

## 分析

这道题的数据范围并不大（1000），所以我比赛的时候随便乱做了一下，也就过了。

不过正解是这样的：首先考虑最小的两个闭区间`A[0]`和`B[0]`，不失一般性，假设`A[0]`的右端点小于等于`B[0]`的右端点。由于`B`中的区间是互不相交的，因此`A[0]`只有可能与`B[0]`相交，不可能与`B`中其他的端点相交。因此我们可以在判断完后将`A[0]`丢掉，然后重新考虑新的`A`和`B`的相交情况。于是就得到了一个`O(N)`的算法。[^sln]

[^sln]: [Leetcode Official Solution for 986](https://leetcode.com/problems/interval-list-intersections/solution/)

## 代码

### 奇怪的做法

```cpp
class Solution {
private:
    bool intersect(Interval x1, Interval x2) {
        if (x1.end < x2.start || x2.end < x1.start) return false;
        return true;
    }
    
    pair<int, int> getIntsc(Interval x1, Interval x2) {
        return make_pair(max(x1.start, x2.start), min(x1.end, x2.end));
    }
    
public:
    vector<Interval> intervalIntersection(vector<Interval>& A, vector<Interval>& B) {
        int start = 0;
        int n = A.size(), m = B.size();
        vector<pair<int, int>> a;
        for (int i = 0; i < n; i++) {
            while (start < m && B[start].end < A[i].start && !intersect(A[i], B[start])) start++;
            if (start >= m) break;
            for (int j = start; j < m; j++)
                if (intersect(A[i], B[j]))
                    a.push_back(getIntsc(A[i], B[j]));
                else break;
        }
        sort(a.begin(), a.end());
        vector<Interval> ans;
        for (int i = 0; i < a.size(); i++)
            ans.emplace_back(a[i].first, a[i].second);
        return ans;
    }
};
```

### 正常的做法

```cpp
class Solution {
public:
    vector<Interval> intervalIntersection(vector<Interval>& A, vector<Interval>& B) {
        int i = 0, j = 0;
        vector<Interval> ans;
        while (i < A.size() && j < B.size()) {
            if (A[i].end < B[j].start) i++;
            else if (B[j].end < A[i].start) j++;
            else {
                ans.emplace_back(max(A[i].start, B[j].start), min(A[i].end, B[j].end));
                if (A[i].end < B[j].end) i++;
                else j++;
            }
        }
        return ans;
    }
};
```