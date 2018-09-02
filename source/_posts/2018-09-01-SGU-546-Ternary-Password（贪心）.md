---
title: SGU 546. Ternary Password（贪心）
urlname: sgu-546-ternary-password
toc: true
date: 2018-09-01 23:58:38
updated: 2018-09-02 00:20:00
tags: [SGU, alg:Greedy]
---

题目来源：[http://codeforces.com/problemsets/acmsguru/problem/99999/546](http://codeforces.com/problemsets/acmsguru/problem/99999/546)

提交次数：2/3

## 题意

给定一个只包含`0`，`1`，`2`三种字符的字符串，长度为`n`（`1 <= n <= 200`）；要求通过替换操作修改字符串，使得`0`的个数为`a`，`1`的个数为`b`（`0 <= a,b <= 200`）。输出最少的替换次数和任意替换后的合法字符串，如无法完成要求，输出`-1`。

## 分析

这道题还比较简单。

通过替换操作修改字符串实际上相当于可以把字符串完全换掉一遍，所以无法完成要求，只可能是`a + b > n`的情况。

然后很显然可以贪心。只要给定了`a`和`b`，事实上替换的方向和数量就都确定了：如果`0`当前的数量比`a`多，那么其中必然有一部分`0`需要被替换掉，而被替换的这些`0`可以是当前字符串中所有`0`的任意子集，所以不妨替换最靠前的那些。进行一次操作之后，得到的必然仍是一个最优子问题。

进行实际替换的操作过程大概是一个贪心，不过这道题的确定性显然比一般所说的“贪心”问题更强。比如说，替换次数就可以直接计算出来：`changeTimes * 2 = abs(a-count('0')) + abs(b-count('1')) + abs(n-a-b-count('2'))`。

## 代码

```cpp
#include <iostream>
#include <cstring>
#include <cmath>
using namespace std;

char p[205];

int main() {
    int n, a, b;
    int actNum[3], expNum[3];
    cin >> n >> a >> b;
    cin >> p;
    memset(actNum, 0, sizeof(actNum));
    expNum[0] = a;
    expNum[1] = b;
    expNum[2] = n - a - b;

    if (a < 0 || b < 0 || a + b > n) {
        cout << -1 << endl;
        return 0;
    }

    for (int i = 0; i < n; i++)
        actNum[p[i] - '0']++;

    // int changed = 0;
    int changed = (abs(actNum[0]-expNum[0]) + abs(actNum[1]-expNum[1]) + abs(actNum[2]-expNum[2])) / 2;
    for (int i = 0; i < n; i++) {
        int x = p[i] - '0';
        if (actNum[x] > expNum[x]) {
            for (int j = 0; j < 3; j++)
                if (j != x && actNum[j] < expNum[j]) {
                    p[i] = j + '0';
                    actNum[j]++;
                    actNum[x]--;
                    // changed++;
                    break;
                }
        }
    }

    cout << changed << endl << p << endl;
    return 0;
}
```
