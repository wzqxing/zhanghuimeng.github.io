---
title: Leetcode 910. Smallest Range II（数学）
urlname: leetcode-910-smallest-range-ii
toc: true
date: 2018-09-23 14:58:14
updated: 2018-09-23 15:23:00
tags: [Leetcode, Leetcode Contest, alg:Math, alg:Greedy]
---


题目来源：[https://leetcode.com/problems/smallest-range-ii/description/](https://leetcode.com/problems/smallest-range-ii/description/)

标记难度：Medium

提交次数：1/1

代码效率：36ms

## 题意

给定整数数组`A`，对于每个`A[i]`，令`B[i] = A[i] + K or - K`。问数组`B`中最大值和最小值之差的最小可能值。

## 分析

在比赛的时候我居然完全没有想到可以把`A`先排序这回事……

---

不妨先把所有元素排序并减去`K`。这样，原问题就转化为，需要选出一部分元素加上`2*K`，另一部分元素不变。显然，为了缩小最大值和最小值的差值，应该选择较小的元素加上`2*K`。

可以证明，需要加上`2*K`的元素是数组中从左侧（最小的元素）开始的连续元素。这一点可以这样说明：

* 假如最优解中有至少一个元素增加了`2*K`（`B[i] = A[i] + 2*K`），且它的左侧有一个不变的元素（`B[j] = A[j]`），那么由于`A[j] < A[i]`，令`B[j] = A[j] + 2*K`不会使`B`数组的最大值变化，但却有可能使`B`数组的最小值增大，情况不会变的更糟；
* 假如最优解中有至少一个元素不变（`B[i] = A[i]`），且它的右侧有一个增加了`2*K`的元素（`B[j] = A[j] + 2*K`），那么由于`A[j] > A[i]`，令`B[j] = A[j]`不会使`B`数组的最小值变化，但却有可能使`B`数组的最大值减小，情况也不会变的更糟

综上，总存在一个只有从左侧开始的连续元素才加了`2*K`的最优解。我觉得这个证明的思路很像贪心；这大概也可以算是一种贪心算法？[^comment]

[^comment]: [Nice proof by veihbisd](https://leetcode.com/problems/smallest-range-ii/discuss/173377/C++JavaPython-Add-0-or-2-*-K/178922)

所以我们可以把元素扫描一遍，对于每个元素，计算“当它是增加的最末一个元素”时的差值，并在这些差值里求最小值。对于`A[i]`，这个差值为`max(A[N-1], A[i] + 2*K) - min(A[i+1], A[0] + 2*K)`。

## 代码

```cpp
class Solution {
public:
    int smallestRangeII(vector<int>& A, int K) {
        sort(A.begin(), A.end());
        int N = A.size();
        int res = A[N-1] - A[0];
        for (int i = 0; i < N - 1; i++) {
            res = min(res, max(A[N-1], A[i] + 2*K) - min(A[i+1], A[0] + 2*K));
        }
        return res;
    }
};
```
