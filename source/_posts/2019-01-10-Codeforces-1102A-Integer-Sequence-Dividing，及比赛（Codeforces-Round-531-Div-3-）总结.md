---
title: Codeforces 1102A. Integer Sequence Dividing，及比赛（Codeforces Round #531 (Div. 3)）总结
urlname: codeforces-1102a-integer-sequence-dividing-and-contest-531-div-3-summary
toc: true
date: 2019-01-10 00:56:36
updated: 2019-01-17 16:03:00
tags: [Codeforces, Codeforces Contest, alg:Math]
---

题目来源：[https://codeforces.com/contest/1102/problem/A](https://codeforces.com/contest/1102/problem/A)

提交次数：1/1

## 题意

将`1, 2, ..., n`分成两个集合，使得两个集合中的数之和的差的绝对值最小。

## 分析

这次做的531是div 3，题目相当之简单，我竟然都会做……

---

比赛的时候的第一直觉是，`n(n+1) / 2`模2余0时返回0（可以分成两个相等的集合），模2余1时返回1（大概只能凑成两个差最小为1的集合）。然后就这样交上去了。

题解里给出了证明。不妨考虑`n, n-1, n-2, n-3`这四个数：我们总可以把`n`和`n-3`放到集合A中，把`n-1`和`n-2`放到集合B中，两边的和是一样的；以此类推，这样我们就只需考虑`n mod 4`的情形。`n mod 4`等于0或3时，可以得到相同的和；等于1或2时只能得到相差为1。[^sln]

[^sln]: [Codeforces Round #531 (Div. 3) Editorial](https://codeforces.com/blog/entry/64439)

另一种方法是证明，对于`1, 2, ..., n`，我们总能得到任意从0到`n(n+1) / 2`的子集和。证明方法大概很简单，可以用数学归纳法，0到`n`的子集和易证，再多的话，减去一些就可以了。

我总觉得之前在CF上做过这道题，而且还认真考虑过分法，但是找不到题目了，真是魔幻。。

## 代码

```cpp
#include <iostream>
using namespace std;
typedef long long int LL;
int main() {
    LL n;
    cin >> n;
    LL prod = n * (n + 1) / 2;
    if (prod % 2 == 0) cout << 0 << endl;
    else cout << 1 << endl;
    return 0;
}
```