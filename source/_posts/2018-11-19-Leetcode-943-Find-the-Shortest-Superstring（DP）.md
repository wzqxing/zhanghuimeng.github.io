---
title: Leetcode 943. Find the Shortest Superstring（DP）
urlname: leetcode-943-find-the-shortest-superstring
toc: true
date: 2018-11-19 01:36:16
updated: 2018-11-25 17:41:00
tags: [Leetcode, Leetcode Contest, alg:Dynamic Programming]
---

题目来源：[https://leetcode.com/problems/find-the-shortest-superstring/description/](https://leetcode.com/problems/find-the-shortest-superstring/description/)

标记难度：Hard

提交次数：3/4

代码效率：2.93% -> 79.31%

## 题意

有N个字符串，找到最小的字符串S，使得这N个字符串都是S的子串。其中N<=12，字符串的长度<=20。

## 分析

这道题比赛的时候我没有做出来，但我自认为自己已经找到了正确的解法（确实差不多是正确的），只要再调一小会就能调出来了！结果事实是又花了两天才弄出来。我犯了这些错误：

* 在搞错了状态变量的范围的同时没有设置好变量的初值
* 计算两个字符串的overlap的函数少考虑了一种情况

所以就这样了……

---

我觉得比较简单的方法还是状态压缩DP。[^solution]令`dp[mask][i]`表示总共包含`mask`这些字符串，且以`A[i]`作为结尾的字符串的最小长度（或者最大overlap长度；当字符串都是那么多时，这两者是一样的。然后就可以递推了：`dp[mask ^ 1<<j][j] = max(dp[mask][i] + overlap(i, j))`。显然，我们事实上可以不用保存具体的字符串（因为有最后一个字符串就够用了），而且可以事先计算出每两个字符串之间的overlap（这样就不需要重复计算）。不过这样就需要最后重建DP过程了……不过字符串处理过程太耗时了，也可以理解……

[^solution]: [Leetcode Official Solution for 943. Find the Shortest Superstring](https://leetcode.com/problems/find-the-shortest-superstring/solution/)

不过这样做了之后时间效率大大提高了（从1324ms提高到了28ms）

## 代码

特别慢的那个就不贴了……

```cpp
class Solution {
private:
    inline int calcOverlap(const string& s1, const string& s2) {
        int start = s1.length() - s2.length();
        start = max(0, start);
        for (int i = start; i < s1.length(); i++) {
            int len = s1.length() - i;
            if (s1.substr(i, len) == s2.substr(0, len))
                return len;
        }
        return 0;
    }
    
    inline bool contains(int cnt, int i) {
        return (cnt & (1 << i)) != 0;
    }
    
public:
    string shortestSuperstring(vector<string>& A) {
        int n = A.size();
        if (n == 1) return A[0];
        
        // pre-calculate overlap graph
        int overlap[n][n];
        for (int i = 0; i < n; i++)
            for (int j = 0; j < n; j++)
                overlap[i][j] = calcOverlap(A[i], A[j]);
        
        const int MAX_CNT = (1 << n);
        int f[MAX_CNT][n], parent[MAX_CNT][n];
        for (int i = 0; i < n; i++) {
            f[1 << i][i] = 0;
            parent[1 << i][i] = -1;
        }
        
        // start DP
        int ans = -1;
        int p = -1;
        for (int cnt = 2; cnt <= n; cnt++) {
            for (int i = 0; i < MAX_CNT; i++) {
                if (__builtin_popcount(i) != cnt) continue;
                // ends with j
                for (int j = 0; j < n; j++) {
                    if (!contains(i, j)) continue;
                    f[i][j] = -1;
                    
                    int nmask = i ^ (1 << j);
                    // last one ends with k
                    for (int k = 0; k < n; k++) {
                        if (!contains(nmask, k)) continue;
                        if (f[nmask][k] + overlap[k][j] > f[i][j]) {
                            f[i][j] = f[nmask][k] + overlap[k][j];
                            parent[i][j] = k;
                        }
                    }
                    
                    if (cnt == n && f[i][j] > ans) {
                        ans = f[i][j];
                        p = j;
                    }
                }
            }
        }
        
        // rebuild the path
        string str;
        int nmask = MAX_CNT - 1;
        while (true) {
            int par = parent[nmask][p];
            if (par == -1) {
                str = A[p] + str;
                break;
            }
            str = A[p].substr(overlap[par][p], A[p].length() - overlap[par][p]) + str;
            nmask ^= (1 << p);
            p = par;
        }
        
        return str;
    }
};
```