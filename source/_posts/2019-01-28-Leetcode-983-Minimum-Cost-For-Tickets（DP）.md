---
title: Leetcode 983. Minimum Cost For Tickets（DP）
urlname: leetcode-983-minimum-cost-for-tickets
toc: true
date: 2019-01-28 16:55:05
updated: 2019-01-28 16:55:05
tags: [Leetcode, Leetcode Contest, alg:Dynamic Programming]
---

题目来源：[https://leetcode.com/problems/minimum-cost-for-tickets/description/](https://leetcode.com/problems/minimum-cost-for-tickets/description/)

标记难度：Medium

提交次数：2/2

代码效率：

* 按连续日期DP：100.00%（0ms）
* 按离散日期DP：100.00%（0ms）

## 题意

你需要在一年里的某些日期旅行；你可以购买1天票、7天票和30天票，每种票的价格不同。问最少花多少钱才能使得需要旅行的每一天都有票？

## 分析

很简单的DP。如果写的是按连续日期DP的话，那`f[i]`就表示直到第`i`天都有合理的票（有些日期需要票，有些则不一定需要）的最少的花费；如果写的是按普通日期DP的话，那`f[i]`就表示直到第`day[i]`天都有合理的票的最少的花费。

## 代码

### 按连续日期DP

```cpp
class Solution {
public:
    int mincostTickets(vector<int>& days, vector<int>& costs) {
        int f[366];
        f[0] = 0;
        int i = 0;
        for (int j = 1; j <= 365; j++) {
            while (days[i] < j && i < days.size()) i++;
            if (i < days.size() && j == days[i]) {
                f[j] = 1e9;
                // 1 day
                f[j] = min(f[j], f[j-1] + costs[0]);
                // 7 days
                // buy on day f[j-k+1]
                for (int k = 1; k <= 7 && j - k >= 0; k++)
                    f[j] = min(f[j], f[j-k] + costs[1]);
                // 30 days
                for (int k = 1; k <= 30 && j - k >= 0; k++)
                    f[j] = min(f[j], f[j-k] + costs[2]);
            }
            else
                f[j] = f[j-1];
        }
        return f[365];
    }
};
```

### 按离散日期DP

```cpp
class Solution {
public:
    int mincostTickets(vector<int>& days, vector<int>& costs) {
        int f[365];
        int N = days.size();
        for (int i = 0; i < N; i++) {
            if (i == 0) {
                f[0] = min(costs[0], costs[1]);
                f[0] = min(costs[2], f[0]);
                continue;
            }
            f[i] = f[i-1] + costs[0];
            // 感觉这个转移条件也是有点奇怪……
            for (int j = 1; j <= 30 && j <= i; j++) {
                int x = j < i ? f[i-j-1] : 0;
                if (days[i] - days[i-j] + 1 <= 7)
                    f[i] = min(f[i], x + costs[1]);
                if (days[i] - days[i-j] + 1 <= 30)
                    f[i] = min(f[i], x + costs[2]);
            }
        }
        return f[N - 1];
    }
};
```