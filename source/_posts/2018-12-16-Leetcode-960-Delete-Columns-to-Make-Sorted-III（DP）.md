---
title: Leetcode 960. Delete Columns to Make Sorted III（DP）
urlname: leetcode-960-delete-columns-to-make-sorted-iii
toc: true
date: 2018-12-16 22:26:13
updated: 2018-12-16 22:47:00
tags: [Leetcode, Leetcode Contest, alg:Dynamic Programming]
---

题目来源：[https://leetcode.com/problems/delete-columns-to-make-sorted-iii/description/](https://leetcode.com/problems/delete-columns-to-make-sorted-iii/description/)

标记难度：Hard

提交次数：1/1

代码效率：24ms

## 题意

有`N`个只包含小写字母的长度相同的字符串。把它们从上到下排成一个矩阵，从中删除一些列，使得剩下的列从左到右是非降序的。问最少删除几列。

## 分析

有人说这道题很难（从提交情况来看，确实很难）。但我完全没有觉得它难，因为在做这个系列的第一题（[Leetcode 944. Delete Columns to Make Sorted](/post/leetcode-944-delete-columns-to-make-sorted)）的时候，我就差点把题意当成了这一道，所以我早就知道这道题怎么解了。

解法很简单。当成一个最长不降子序列问题就可以。两列满足不降的条件是，对应的每对字符都是不降的。所以复杂度是`O(N*M^2)`。

随便讲两句废话。按Leetcode的记录，我已经参加了21场比赛了，从8月14日到12月15日。这期间很多事情在逐渐发生变化。比如我的rating不再变了，对活跃在题解区的几个人有了更多了解，[lee215](https://leetcode.com/lee215/)除了在题解区写题解之外还开始在简书上发中文题解和在[youtube](https://www.youtube.com/channel/UCUBt1TDQTl1atYsscVoUzoQ)上发视频，今天正好一共刷了233道题。我感觉，只要我在Leetcode上还有做起来费劲的题，刷Leetcode就不是没有价值的，因为掌握不止要看“会”，还要看“熟练度”；但是刷Leetcode还是远远不够的。

据说Leetcode用户觉得最难的题目类型是动态规划。

今天去考了CSP，感觉自己还是菜。翻翻《算法竞赛入门经典训练指南》，感觉自己不会的topic比会的多多了。等到寒假我打算尝试把USACO做完，并且开始打CF的div3之类的。

## 代码

```cpp
class Solution {
public:
    int minDeletionSize(vector<string>& A) {
        // 真·最长不降子序列
        int f[101];
        int ans = -1;
        int N = A.size(), M = A[0].length();
        f[0] = 1;
        for (int i = 1; i < M; i++) {
            f[i] = 1;
            for (int j = 0; j < i; j++) {
                bool isOk = true;
                for (int k = 0; k < N; k++) {
                    if (A[k][j] > A[k][i]) {
                        isOk = false;
                        break;
                    }
                }
                if (!isOk) continue;
                f[i] = max(f[i], f[j] + 1);
            }
            ans = max(f[i], ans);
        }
        return M - ans;
    }
};
```