---
title: Leetcode 915. Partition Array into Disjoint Intervals（数组）
urlname: leetcode-915-partition-array-into-disjoint-intervals
toc: true
date: 2018-09-30 20:15:48
updated: 2018-09-30 21:03:00
tags: [Leetcode, Leetcode Contest, alg:Array]
---

题目来源：[https://leetcode.com/problems/partition-array-into-disjoint-intervals/description/](https://leetcode.com/problems/partition-array-into-disjoint-intervals/description/)

标记难度：Medium

提交次数：2/3

代码效率：

* 3 Passes：28ms
* 1 Pass：28ms

## 题意

给定一个数组，要求将它分成`left`和`right`两半，满足：

* `len(left) > 0`，`len(right) > 0`
* `max(left) <= min(right)`
* `left`的长度最小

数据保证存在合法的分割方法，求`left`的最小长度。

## 分析

这道题还挺有意思的。

### 建立辅助数组

比赛的时候我用的就是这种方法，因为最好想。令`maxn[i] = max(A[0], ..., A[i])`，`minn[i] = min(A[i], ..., A[n-1])`，这两个数组可以分别通过一次遍历求得；然后再进行一次遍历，找出满足`maxn[i] <= minn[i + 1]`的最小的`i`即可。时间复杂度和额外空间复杂度都是`O(N)`。

我在考虑怎么优化这个算法的时候思考了一下这两个数组的性质。显然`maxn[i]`是递增的，而`minn[i]`也是递增的，但增长速度不同，所以会出现一些`maxn[i] <= minn[i+1]`的区间。我们要做的就是找到第一个这样的区间。但是这个东西似乎也没有很强的单调性……

### 不建立辅助数组

很显然，因为要求`len(left) > 0`，因此必然有`max(left) >= A[0]`。然后从左向右进行扫描，记`leftMax`为已经确定需要包括在`left`数组中的数的最大值，`j`为当前已经确定的`left`数组的右边界，`curMax`为当前已经发现的最大值。对于下一个数`A[i]`：

* 如果`A[i] < leftMax`，则它必须被包含在`left`中（否则`right`中将出现比`left`的最大值更小的数）。因此需要更新`j = i`，`leftMax = curMax`（因为`left`是连续的，所以之前已经发现过的最大值现在也必须包含于`left`中）。
* 如果`leftMax <= A[i] <= curMax`，则这个数有可能被包含在`left`中（如果右侧出现比`leftMax`更小的数），也有可能不需要（如果右侧没有出现这样的数）。所以暂时不作处理。
* 如果`A[i] > curMax`，则这个数也不一定被包含在`left`中（和前一种情况同理）。更新`curMax = A[i]`。

最后的结果为`j+1`。[^onepass]

以数组`[5, 7, 3, 8, 6, 9, 10]`为例。初始时，令`leftMax = 5`，`curMax = 5`，`j = 0`。

```
Initial State:
leftMax = 5, curMax = 5, j = 0
5  7  3  8  6  9  10
l  -  -  -  -  -  -

i = 1: Update curMax to 7
leftMax = 5, curMax = 7, j = 0
5  7  3  8  6  9  10
l  ^  -  -  -  -  -

i = 2: Update leftMax to 7 and j to 2
leftMax = 7, curMax = 7, j = 2
5  7  3  8  6  9  10
l  l  l  -  -  -  -

i = 3: Update curMax to 8
leftMax = 7, curMax = 8, j = 2
5  7  3  8  6  9  10
l  l  l  ^  -  -  -

i = 4: Update leftMax to 8, j to 4
leftMax = 8, curMax = 8, j = 4
5  7  3  8  6  9  10
l  l  l  l  l  -  -

i = 5: Update curMax to 9
leftMax = 8, curMax = 9, j = 4
5  7  3  8  6  9  10
l  l  l  l  l  ^  -

i = 6: Update curMax to 10
leftMax = 8, curMax = 10, j = 4
5  7  3  8  6  9  10
l  l  l  l  l  ^  ?
```

这一方法的时间复杂度仍然为`O(N)`，额外空间复杂度为`O(1)`。

[^onepass]: 这一想法的灵感来自[Different Thinking - Intuitive Explanation](https://leetcode.com/problems/partition-array-into-disjoint-intervals/discuss/175985/Different-Thinking-Intuitive-Explanation)

### 更多

我花了一些时间，用于思考这个问题的复杂度能否降得更低。但我并不能想出来：我既不能很好地利用`O(log(N))`的最大值选择算法，也不能证明复杂度的下界是`O(N)`。所以反正就是想不出来。

## 代码

### 3 Passes

```cpp
class Solution {
public:
    int partitionDisjoint(vector<int>& A) {
        int n = A.size();
        int maxn[n], minn[n];
        for (int i = 0; i < n; i++) {
            if (i == 0 || A[i] > maxn[i-1])
                maxn[i] = A[i];
            else
                maxn[i] = maxn[i - 1];
        }
        for (int i = n - 1; i >= 0; i--) {
            if (i == n - 1 || A[i] < minn[i+1])
                minn[i] = A[i];
            else
                minn[i] = minn[i + 1];
        }
        for (int i = 0; i < n - 1; i++) {
            if (maxn[i] <= minn[i+1])
                return i + 1;
        }
        return -1;
    }
};
```

### 1 Pass

```cpp
class Solution {
public:
    int partitionDisjoint(vector<int>& A) {
        int leftMax = A[0], curMax = A[0], j = 0;
        for (int i = 1; i < A.size(); i++) {
            if (A[i] < leftMax) {
                j = i;
                leftMax = curMax;
            }
            else if (A[i] > curMax) {
                curMax = A[i];
            }
        }
        return j + 1;
    }
};
```
