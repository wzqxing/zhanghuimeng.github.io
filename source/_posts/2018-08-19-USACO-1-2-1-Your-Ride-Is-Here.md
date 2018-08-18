---
title: 'USACO 1.2.1: Your Ride Is Here'
urlname: usaco-1-2-1-your-ride-is-here
toc: true
date: 2018-08-19 01:20:22
updated: 2018-08-19 01:27:22
tags: [USACO]
---

额，我竟然开始重刷USACO了……希望这次能刷完，不要像高中时那样半途而废吧。USACO的题目对现在的我说不上多有用，但这可能是种情结。

## 题意

见[洛谷 P1200](https://www.luogu.org/problemnew/show/P1200)。（网上这道题的相关翻译到处都是，但我不知道现在NOCOW的镜像站还能活多久，所以还是看洛谷好了。）

## 分析

很水，直接乘了之后取模就可以了。考虑到`26^5 = 11881376`，甚至都不需要考虑乘法溢出的问题（虽然我还是做了）。

## 代码

```cpp
/*
ID: zhanghu15
TASK: ride
LANG: C++14
*/

#include <iostream>
#include <fstream>
#include <string>

using namespace std;

int main() {
    ofstream fout("ride.out");
    ifstream fin("ride.in");
    string comet, group;
    fin >> comet >> group;
    int cometProd = 1, groupProd = 1;
    for (char c: comet)
        cometProd = (cometProd * (c - 'A' + 1)) % 47;
    for (char c: group)
        groupProd = (groupProd * (c - 'A' + 1)) % 47;

    if (cometProd == groupProd)
        fout << "GO" << endl;
    else
        fout << "STAY" << endl;

    return 0;
}
```
