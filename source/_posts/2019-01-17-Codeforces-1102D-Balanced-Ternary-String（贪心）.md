---
title: Codeforces 1102D. Balanced Ternary String（贪心）
urlname: codeforces-1102d-balanced-ternary-string
toc: true
date: 2019-01-17 16:51:58
updated: 2019-01-17 17:05:00
tags: [Codeforces, Codeforces Contest, alg:Greedy]
---

题目来源：[https://codeforces.com/contest/1102/problem/D](https://codeforces.com/contest/1102/problem/D)

提交次数：1/1

## 题意

给定一个只包含0、1、2的长度是3的倍数的字符串，要求替换最少数量的字符，使得其中0、1、2的数量相等，且得到的字符串字典序最小。

## 分析

这道题让我想起之前的某道USACO题（[Sorting a Three-Valued Sequence](https://codeforces.com/blog/entry/19971)）。好吧，其实没啥太大的关系（除了都是三值序列以外）。

既然要求替换最少数量的字符，那么把什么字符换成什么字符，以及换多少个其实都是确定的。如果这一点听起来不是很直接的话……比如，0的数量和1的数量都比`n/3`多，那么多出来的0和1都要换成2。如果0的数量和1的数量都比`n/3`少，那么多出来的2要分别换成0和1。实际上只有这两种情况。

然后就可以开始考虑怎么换了。显然，如果要把小的字符换成大的字符，应该从后向前换，换完为止；如果把大的字符换成小的字符，应该从前往后换，也是换完为止。

光考虑这一点还不够。比如说我们需要把两个0换成1，两个0换成2，那么应该先（从后向前）换2，再（从后向前）换1。

然后就能过啦。

## 代码

```cpp
#include <iostream>
using namespace std;
int n;
int a[300000];
int cnt[3];
int main() {
    char c;
    cin >> n;
    for (int i = 0; i < n; i++) {
        cin >> c;
        a[i] = c - '0';
        cnt[a[i]]++;
    }
    int m = n / 3;
    // 从i改成j
    int order[3][3] = {
        {2, 1, 0},
        {0, 1, 2},
        {0, 1, 2}
    };
    for (int i = 0; i < 3; i++) {
        for (int j1 = 0; j1 < 3; j1++) {
            int j = order[i][j1];
            if (i == j) continue;
            if (cnt[i] == m || cnt[j] == m) continue;
            if (cnt[i] < m || cnt[j] > m) continue;
            // 从后向前改
            if (i < j) {
                for (int k = n - 1; k >= 0; k--) {
                    if (cnt[i] == m || cnt[j] == m) break;
                    if (a[k] == i) {
                        a[k] = j;
                        cnt[i]--;
                        cnt[j]++;
                    }
                }
            }
            // 从前向后改
            if (i > j) {
                for (int k = 0; k < n; k++) {
                    if (cnt[i] == m || cnt[j] == m) break;
                    if (a[k] == i) {
                        a[k] = j;
                        cnt[i]--;
                        cnt[j]++;
                    }
                }
            }
        }
    }
    for (int i = 0; i < n; i++)
        cout << a[i];
    cout << endl;
    return 0;
}
```