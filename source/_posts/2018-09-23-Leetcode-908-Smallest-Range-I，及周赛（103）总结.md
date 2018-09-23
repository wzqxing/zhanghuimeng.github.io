---
title: Leetcode 908. Smallest Range I，及周赛（103）总结
urlname: leetcode-908-smallest-range-i-and-weekly-contest-103
toc: true
date: 2018-09-23 13:48:27
updated: 2018-09-23 14:03:00
tags: [Leetcode, Leetcode Contest, alg:Math]
---

题目来源：[https://leetcode.com/problems/smallest-range-i/description/](https://leetcode.com/problems/smallest-range-i/description/)

标记难度：Easy

提交次数：2/4

代码效率：24ms

## 题意

给定整数数组`A`，对于每个`A[i]`，可以选择任意`-K <= x <= K`，令`B[i] = A[i] + x`。问数组`B`中最大值和最小值之差的最小可能值。

## 分析

这次比赛我获得了180 / 4160的名次，是最近一段时间以来最高的一次了。我猜想这是因为这次的题目整体难度非常简单，居然有三道Medium的缘故。而且，很神奇的是，我是先做完第2题，第4题，最后几分钟才做完第1题的，因为我开始时觉得第1题和第3题都特别难……当然事实证明它们还是比较简单的。

---

思路非常简单：如果`max(A) - min(A) <= 2*K`，则可以通过上述方式将所有`A`中的数都变成相等的，如`min(A) + (max(A) - min(A)) / 2`；而如果`max(A) - min(A) > 2*K`，则显然差值的最小值是`max(A) - min(A) - 2*K`。对于数组中的各个值，可以分别进行以下操作：

* 如果`A[i] <= min(A) + K`，则`B[i] = A[i] + K`
* 如果`A[i] >= max(A) - K`，则`B[i] = A[i] - K`
* 如果`min(A) + K < A[i] < max(A) - K`，则`B[i] = A[i]`

此时所有`B[i]`都位于`[min(A) + K, max(A) - K]`区间内了。[^solution]

[^solution]: [Leetcode 908 Solution](https://leetcode.com/problems/smallest-range-i/solution/)

## 代码

```cpp
class Solution {
public:
    int smallestRangeI(vector<int>& A, int K) {
        int maxn = A[0], minn = A[0];
        for (int i = 0; i < A.size(); i++) {
            maxn = max(maxn, A[i]);
            minn = min(minn, A[i]);
        }
        if (maxn - minn > 2 * K) return maxn - minn - 2 * K;
        else return 0;
    }
};
```
