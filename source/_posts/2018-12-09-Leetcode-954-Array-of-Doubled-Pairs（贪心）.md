---
title: Leetcode 954. Array of Doubled Pairs（贪心）
urlname: leetcode-954-array-of-doubled-pairs
toc: true
date: 2018-12-09 16:30:27
updated: 2018-12-09 16:48:00
tags: [Leetcode, Leetcode Contest, alg:Greedy]
---

题目来源：[https://leetcode.com/problems/verifying-an-alien-dictionary/description/](https://leetcode.com/problems/verifying-an-alien-dictionary/description/)

标记难度：Easy

提交次数：1/2

代码效率：104ms

## 题意

给定一个长度为偶数的数组`A`，问能否将`A`排序，使得对于每个`0<=i<len(A)/2`，都有`A[2*i+1] = A[2*i]`。`-100000<=A[i]<=100000`。

事实上就是每个数都要和一个是它的二倍或者二分之一的数配对……

## 分析

我发现赛后这道题的函数签名也改了，从`bool canSortDoubled(vector<int>& A)`变成了`bool canReorderDoubled(vector<int>& A)`……

而且我比赛时的代码写错了……忘了判断大数是否被耗尽的情况……

这也能过？看来数据太弱了。

![出错！](wrong.png)

---

这道题用贪心就可以解决。对于当前还剩的绝对值最小的数，它必然需要和它的两倍的数配对；最后所有的数都需要配对完。以及0只能自己和自己配对，所以0的总数应该是偶数。

## 代码

```cpp
class Solution {
public:
    bool canReorderDoubled(vector<int>& A) {
        int positive[100001], negative[100001];
        memset(positive, 0, sizeof(positive));
        memset(negative, 0, sizeof(negative));
        for (int x: A) {
            if (x >= 0) positive[x]++;
            else negative[-x]++;
        }
        if (positive[0] % 2 != 0) return false;
        for (int i = 1; i <= 100000; i++) {
            if (positive[i] != 0) {
                if (i > 50000 || positive[i*2] < positive[i]) return false;
                positive[i*2] -= positive[i];
            }
        }
        for (int i = 1; i <= 100000; i++) {
            if (negative[i] != 0) {
                if (i > 50000 || negative[i*2] < negative[i]) return false;
                negative[i*2] -= negative[i];
            }
        }
        return true;
    }
};
```