---
title: Codeforces 1037A. Packets（数学），及比赛（Codefest 18）总结
urlname: codeforces-1037a-packets-and-contest-codefest-18
toc: true
mathjax: true
date: 2018-09-03 16:14:53
updated: 2018-09-03 16:33:00
tags: [Codeforces, Codeforces Contest, alg:Math]
---

题目来源：[https://codeforces.com/contest/1037/problem/A](https://codeforces.com/contest/1037/problem/A)

提交次数：1/1

## 题意

给定`n`枚一元硬币，要求把硬币分成若干包，使得任意`1 <= x <= n`的钱数都可以用这些硬币包表示出来。问硬币包的最小数量。

## 分析

这次比赛一共有8道题，其中大概前4题是比较简单的。比赛的时候我过了前4题的pretest，后来第4题没过全部的测试点，因为超时了。最后我的排名是2219 / 7396，rating增加了1，从1500变成了1501。（。。。）

这次暂时还没有时间和心情体验Hack机制。

---

从直觉上来说，显然用二进制的方式来构造硬币包比较好。所以不妨找到最大的的$k$，满足$1 + 2 + ... + 2^k \leq n$。这样，这$k$个硬币包就可以表示$[1, 2^{k+1} - 1]$范围内的所有数了。然后把剩下的所有硬币打包成大小为$n - 2^{k+1} + 1$的包。这样，$[2^{k+1}, n]$范围内的数就可以用刚才剩下的那个包+二进制硬币包来表示了。

不知道如何证明这个方法是最优的，虽然直觉上就是这样。

## 代码

```cpp
#include <iostream>
using namespace std;
int main() {
    int n;
    cin >> n;
    int i = 1, cnt = 0;
    while (n != 0) {
        if (i > n) break;
        n -= i;
        cnt++;
        i *= 2;
    }
    if (n != 0)
        cnt++;
    cout << cnt << endl;
    return 0;
}
```
