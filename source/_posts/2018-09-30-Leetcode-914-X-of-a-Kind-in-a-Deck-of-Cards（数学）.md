---
title: Leetcode 914. X of a Kind in a Deck of Cards（数学），及周赛（104）总结
urlname: leetcode-914-x-of-a-kind-in-a-deck-of-dards-and-contest-104
toc: true
date: 2018-09-30 19:45:13
updated: 2018-09-30 19:45:13
tags: [Leetcode, Leetcode Contest, alg:Math, alg:Hash Table]
---

题目来源：[https://leetcode.com/problems/x-of-a-kind-in-a-deck-of-cards/description/](https://leetcode.com/problems/x-of-a-kind-in-a-deck-of-cards/description/)

标记难度：Easy

提交次数：1/1

代码效率：12ms

## 题意

给定若干个元素，问每种元素的数量的最大公约数。

## 分析

这次比赛时我正在上《信号处理原理》。所以就随便写了一下。前三题过于水了，一共花了不到20分钟……然后我觉得第四题应该要用对抗搜索，就直接上课了，没去管它。后来我发现我居然是51 / 3579名，4道题都做出来的一共41个人，而做出来三道题的超过了1000个人。所以这是一次拼手速的比赛。

---

直接用hash表统计元素数量加gcd算法就好了，这道题真的水……

## 代码

```cpp
class Solution {
private:
    int gcd(int x, int y) {
        if (x % y == 0) return y;
        return gcd(y, x % y);
    }

public:
    bool hasGroupsSizeX(vector<int>& deck) {
        int h[10005];
        memset(h, 0, sizeof(h));
        for (int d: deck)
            h[d]++;
        int g = -1;
        for (int i = 0; i < 10000; i++) {
            if (g == 1) break;
            if (h[i] > 0) {
                if (g == -1)
                    g = h[i];
                else
                    g = gcd(h[i], g);
            }
        }
        return g > 1;
    }
};
```
