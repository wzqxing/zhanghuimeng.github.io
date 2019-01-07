---
title: 'USACO 1.6.1: Number Triangles'
urlname: usaco-1-6-1-number-triangles
toc: true
date: 2019-01-06 20:25:07
updated: 2019-01-07 20:28:00
tags: [USACO, alg:Dynamic Programming]
---

## 题意

见[P1216 \[IOI1999\]\[USACO1.5\]数字三角形 Number Triangles](https://www.luogu.org/problemnew/show/P1216)。

给定一个数字三角形，问从顶到底最大的路径和。

## 分析

非常简单的DP。对于每个点，判断从上左方走过来还是从上右方走过来和最大即可。上次从[Leetcode 931](/post/leetcode-931-minimum-falling-path-sum/)中学到一点，对于这样的题，可以不新开一个数组，直接在输入数组上DP。

自底向上做当然也可以，就是有点怪异……

## 代码

```cpp
/*
ID: zhanghu15
TASK: numtri
LANG: C++14
*/

#include <iostream>
#include <fstream>
#include <vector>
#include <algorithm>
#include <cstring>

using namespace std;

int tri[1001][1001];
int main() {
    ofstream fout("numtri.out");
    ifstream fin("numtri.in");
    int R;
    fin >> R;
    int ans = -1;
    for (int i = 1; i <= R; i++) {
        for (int j = 1; j <= i; j++) {
            fin >> tri[i][j];
            tri[i][j] += max(tri[i-1][j], tri[i-1][j-1]);
            if (i == R) ans = max(ans, tri[i][j]);
        }
    }
    fout << ans << endl;
    return 0;
}
```