---
title: Codeforces 1037B. Reach Median（贪心）
urlname: codeforces-1037-b-reach-median
toc: true
date: 2018-09-03 16:36:25
updated: 2018-09-03 16:51:00
tags: [Codeforces, Codeforces Contest, alg:Greedy]
---

题目来源：[https://codeforces.com/contest/1037/problem/B](https://codeforces.com/contest/1037/problem/B)

提交次数：1/2

## 题意

给定`n`（保证`n`为奇数）个正数和`s`，要求修改这些数的数值，使得这`n`个数的中位数是`s`，问数值变化的绝对值总和最小是多少。

## 分析

比赛的时候我自以为想出了正确的思路，然后交上去了，结果连pretest都没过。一个小时之后，我突然灵光一闪：原题可没有保证输出的数据范围，也没要求取模。于是我把`int`改成`long long int`，重新交上去了，这回就过了。

---

首先把数组排个序，然后就可以得到现在的中位数了，记为`mid`。如果`mid == s`，那么不用做任何改变。但如果`mid != s`，通过一些修改，把当前的`mid`变成`s`，仍然是最好的方法。若`mid < s`，则把`mid`改成`s`，并把所有满足`mid <= x < s`的`x`也改成`s`，保证现在的`mid`仍为中位数；若`mid > s`，则把`mid`改成`s`，并把所有满足`s < x <= mid`的`x`也改成`s`，保证现在的`mid`仍为中位数。

## 代码

```cpp
#include <iostream>
#include <cstdio>
#include <algorithm>
using namespace std;
int a[200000], n, s;
int main() {
    scanf("%d %d", &n, &s);
    for (int i = 0; i < n; i++)
        scanf("%d", &a[i]);
    sort(a, a + n);
    // 实际上不需要显式地计算lower和upper，遍历一遍即可
    // 甚至只遍历半遍也可以，因为两侧是对称的
    // 参考标答：
    // https://codeforces.com/contest/1037/submission/42406891
    int lower = lower_bound(a, a + n, s) - a;
    int upper = upper_bound(a, a + n, s) - a;
    int mid = n / 2;
    // [lower, upper)
    long long int sum = 0;
    if (lower <= mid && mid < upper) {
        sum = 0;
    }
    else if (mid < lower) {
        for (int i = mid; i < lower; i++)
            sum += s - a[i];
    }
    else {
        for (int i = upper; i <= mid; i++)
            sum += a[i] - s;
    }
    cout << sum << endl;
    return 0;
}
```
