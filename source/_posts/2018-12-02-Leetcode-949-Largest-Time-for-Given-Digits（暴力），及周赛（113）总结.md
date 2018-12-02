---
title: Leetcode 949. Largest Time for Given Digits（暴力），及周赛（113）总结
urlname: leetcode-949-largest-time-for-given-digits-and-weekly-contest-113
toc: true
date: 2018-12-02 12:30:04
updated: 2018-12-02 12:45:00
tags: [Leetcode, Leetcode Contest, alg:Math, alg:Brute Force]
---

题目来源：[https://leetcode.com/problems/largest-time-for-given-digits/description/](https://leetcode.com/problems/largest-time-for-given-digits/description/)

标记难度：Easy

提交次数：1/2

代码效率：4ms

## 题意

给定4个数字，求出能用这些数字排列成的最大时间（24小时制）。

## 分析

这次我没参加正式比赛，因为我参加internal contest去了。很有趣，而且有很多感受，我可以再写一篇文章出来了……internal contest虽然人少，也是有计时和排名的，我看了一下，我在正式比赛中大概能排在50-100名左右吧。

---

这道题看起来很简单，因为“时间”这个属性限制了解的数量，这使得暴力方法非常适用；这一点让我想起之前做的[Leetcode 401. Binary Watch](/post/leetcode-401-binary-watch)，也是暴力方法优于搜索方法的例子。反正就是枚举这4个数字的排列，然后从中选出合法且最大的时间。

但是这道题的通过率相当低（虽然通过的人数显然不低）。（我猜）这是因为很多人都没有处理时间数字前面带0的情况，于是在`[0, 0, 0, 0]`这样的样例中就挂了。我很难想到什么保证自己能想到这样的corner case的例子，也许只能多练吧。

## 代码

```cpp
class Solution {
public:
    string largestTimeFromDigits(vector<int>& A) {
        sort(A.begin(), A.end());
        int maxtime = -1;
        string time;
        do {
            int hour = A[0] * 10 + A[1];
            if (hour >= 24) continue;
            int minute = A[2] * 10 + A[3];
            if (minute >= 60) continue;
            if (hour * 100 + minute > maxtime) {
                maxtime = hour * 100 + minute;
                time = to_string(A[0]) + to_string(A[1]) + ":" + to_string(A[2]) + to_string(A[3]);
            }
        } while (next_permutation(A.begin(), A.end()));
        return time;
    }
};
```