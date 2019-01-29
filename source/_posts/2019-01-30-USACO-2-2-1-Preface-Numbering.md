---
title: 'USACO 2.2.1: Preface Numbering'
urlname: usaco-2-2-1-preface-numbering
toc: true
date: 2019-01-30 00:44:35
updated: 2019-01-30 00:44:35
tags: [USACO]
---

## 题意

见[洛谷 P1465 序言页码 Preface Numbering](https://www.luogu.org/problemnew/show/P1465)。

求1到`N`的罗马数字表示中每个字母出现的次数。

## 分析

这道题出得有点令人无言以对（的简单）的感觉……总之做就是了。

罗马数字表示法和普通的十进制其实很类似，都是用单个数字来表示某个位，而且各个位的规律是差不多的。比如说，我们可以把一个四位的十进制数字这样分解：

* 个位：`1 -> I, 2 -> II, 3 -> III, 4 -> IV, 5 -> V, 6 -> VI, 7 -> VII, 8 -> VIII, 9 -> IX`
* 十位：`1 -> X, 2 -> XX, 3 -> XXX, 4 -> XL, 5 -> L, 6 -> LX, 7 -> LXX, 8 -> LXXX, 9 -> XC`
* 百位：`1 -> C, 2 -> CC, 3 -> CCC, 4 -> CD, 5 -> D, 6 -> DC, 7 -> DCC, 8 -> DCCC, 9 -> CM`
* 千位：`1 -> M, 2 -> MM, 3 -> MMM`（题目里没有更大的数字了）

所以直接按照这个方法去分解就可以啦！

我写得非常暴力，考虑到各个位之间的规律性，可以写得稍微更简单一点。不过因为数组清零时写成了`sizeof(0)`WA了一次……

题解里有各种花式简化方法。

## 代码

```cpp
/*
ID: zhanghu15
TASK: preface
LANG: C++14
*/

#include <iostream>
#include <fstream>
#include <vector>
#include <algorithm>
#include <cstring>
#include <cmath>

using namespace std;
typedef long long int LL;

int main() {
    ofstream fout("preface.out");
    ifstream fin("preface.in");
    int N;
    fin >> N;
    int d[4], n;
    int ci = 0, cv = 0, cx = 0, cl = 0, cc = 0, cd = 0, cm = 0;
    for (int i = 1; i <= N; i++) {
        int x = i;
        n = 0;
        memset(d, 0, sizeof(d));
        while (x > 0) {
            d[n++] = x % 10;
            x /= 10;
        }

        cm += d[3];

        if (d[2] == 9)  // 900 = CM
            cc++, cm++;
        else if (5 < d[2] && d[2] < 9)  // 800 = DCCC
            cd++, cc += d[2] - 5;
        else if (d[2] == 5)  // 500 = D
            cd++;
        else if (d[2] == 4)  // 400 = CD
            cc++, cd++;
        else // 300 = CCC
            cc += d[2];
        
        if (d[1] == 9)  // 90 = XC
            cx++, cc++;
        else if (5 < d[1] && d[1] < 9)  // 80 = LXXX
            cl++, cx += d[1] - 5;
        else if (d[1] == 5)  // 50 = L
            cl++;
        else if (d[1] == 4)  // 40 = XL
            cx++, cl++;
        else  // 30 = XXX
            cx += d[1];
        
        if (d[0] == 9)  // 9 = IX
            ci++, cx++;
        else if (5 < d[0] && d[0] < 9)  // 8 = VIII
            cv++, ci += d[0] - 5;
        else if (d[0] == 5)  // 5 = V
            cv++;
        else if (d[0] == 4)  // 4 = IV
            ci++, cv++;
        else  // 3 = III
            ci += d[0];
    }

    if (ci > 0) fout << "I " << ci << endl;
    if (cv > 0) fout << "V " << cv << endl;
    if (cx > 0) fout << "X " << cx << endl;
    if (cl > 0) fout << "L " << cl << endl;
    if (cc > 0) fout << "C " << cc << endl;
    if (cd > 0) fout << "D " << cd << endl;
    if (cm > 0) fout << "M " << cm << endl;
    return 0;
}
```