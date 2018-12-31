---
title: Leetcode 967. Numbers With Same Consecutive Differences
urlname: leetcode-967-numbers-with-same-consecutive-differences
toc: true
date: 2018-12-31 14:55:39
updated: 2018-12-31 15:09:00
tags: [Leetcode, Leetcode Contest]
---

题目来源：[https://leetcode.com/problems/numbers-with-same-consecutive-differences/description/](https://leetcode.com/problems/numbers-with-same-consecutive-differences/description/)

标记难度：Medium

提交次数：1/3

代码效率：8ms（100.00%）

## 题意

找出所有长度为`N`的数字，它们连续两位的差的绝对值都是`K`。数字不应含有前导零。

## 分析

这道题很显然很简单……首先是初始条件：`{1, 2, 3, ..., 9}`。然后枚举`N-1`次，每次都对集合中现有的元素尝试在后面加上新的一位，满足它和前一位的差的绝对值是`K`。

我中间写挂了一次是因为忘了去重了！如果`K=0`，当前数字的最后一位是`d`，则很显然可能存在`d+0=d-0`的情况。这是一个很好的边界条件……

以及我没看出来这道题有什么DP的影子……如果加长`N`并且只问总数还差不多。

## 代码

```cpp
class Solution {
public:
    vector<int> numsSameConsecDiff(int N, int K) {
        if (N == 1) return {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
        vector<int> a = {1, 2, 3, 4, 5, 6, 7, 8, 9};
        for (int i = 1; i < N; i++) {
            vector<int> b;
            int M = a.size();
            for (int j = 0; j < M; j++) {
                int x = a[j];
                int d = x % 10;
                if (d + K <= 9) b.push_back(x * 10 + d + K);
                // Note: if K == 0, then we need to deduplicate...
                if (d - K >= 0 && d + K != d - K) b.push_back(x * 10 + d - K);
            }
            a = b;
        }
        return a;
    }
};
```
