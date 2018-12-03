---
title: 'USACO 1.4.4: Combination Lock（暴力）'
urlname: usaco-1-4-4-combination-lock
toc: true
date: 2018-12-03 15:00:44
updated: 2018-12-03 15:00:44
tags: [USACO, alg:Brute Force]
---

## 题意

见[洛谷 P2963](https://www.luogu.org/problemnew/show/P2693)。

有一些密码锁组合（形如`[1, 2, 3]`，每个数的范围是`[1, N]`），定义两个组合每个位置上的数相差两个位置以内时是**接近**的（密码锁是一个圈，所以1和N是相邻的）。给定两个已知组合，问所有组合里和这两个组合中的至少一个**接近**的有多少。

## 分析

暴力方法和[上一道题](/post/usaco-1-4-3-prime-cryptarithm)如出一辙。可能需要注意的两点：

* 要求是和至少一个组合接近，而不是每个位置上的数和至少一个组合在这个位置上的数相近。不过这一点仍然可以用来剪枝。
* 如何判断**接近**。题解里的方法是`abs(a-b) <= 2 || abs(a-b) >= N-2`；我写的是`(a-b+N) % N <= 2 || (b-a+N) % N <= 2`。这两者应该都是对的。嗯，总之就是需要注意一下。

这道题题解里竟然有个[录像](https://www.youtube.com/watch?v=nPs1fvOBG9Y)，讲得很不错。

## 代码

```cpp
/*
ID: zhanghu15
TASK: combo
LANG: C++14
*/

#include <iostream>
#include <fstream>
#include <vector>
#include <algorithm>

using namespace std;

bool isNear(int a, int b, int N) {
    return (a-b+N) % N <= 2 || (b-a+N) % N <= 2;
}

int main() {
    ofstream fout("combo.out");
    ifstream fin("combo.in");
    int N;
    int combo1[3];
    int combo2[3];
    fin >> N;
    fin >> combo1[0] >> combo1[1] >> combo1[2];
    fin >> combo2[0] >> combo2[1] >> combo2[2];
    int ans = 0;
    for (int c1 = 1; c1 <= N; c1++) {
        if (!isNear(combo1[0], c1, N) && !isNear(combo2[0], c1, N)) continue;
        for (int c2 = 1; c2 <= N; c2++) {
            if (!isNear(combo1[1], c2, N) && !isNear(combo2[1], c2, N)) continue;
            for (int c3 = 1; c3 <= N; c3++) {
                if (isNear(combo1[0], c1, N) && isNear(combo1[1], c2, N) && isNear(combo1[2], c3, N) || 
                    isNear(combo2[0], c1, N) && isNear(combo2[1], c2, N) && isNear(combo2[2], c3, N))
                    ans++;
            }
        }
    }
    fout << ans << endl;
    return 0;
}
```