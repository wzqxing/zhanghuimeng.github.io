---
title: 'USACO 2.1.4: Healthy Holsteins（枚举）'
urlname: usaco-2-1-4-healthy-holsteins
toc: true
date: 2019-01-29 21:58:34
updated: 2019-01-29 21:58:34
tags: [USACO, alg:Brute Force]
---

## 题意

见[P1460 健康的荷斯坦奶牛 Healthy Holsteins](https://www.luogu.org/problemnew/show/P1460)。

给定一些稻草（种类数量<=15），每种稻草可以为奶牛提供若干数量的若干种维他命，问最少需要多少种稻草才能为奶牛提供足够的维他命？

## 分析

我最开始以为这是一道01背包题，然后发现维他命有25种，25维的01背包可还行？最后发现原来稻草最多只有15种，那就直接`2^G`枚举好了。

## 代码

这次的枚举是用循环写的（而不是DFS之类）。但仔细想想，这个写法（专指这道题的写法）不怎么样，外面还加了一层处理稻草个数的循环。虽然这道题里可以把这个循环去掉然后手动判断稻草个数，但在其他很多状态DP里不如写成递归的……

```cpp
/*
ID: zhanghu15
TASK: holstein
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

int V;
int need[25];
int sum[25];
int G;
int scoop[15][25];

int main() {
    ofstream fout("holstein.out");
    ifstream fin("holstein.in");

    fin >> V;
    for (int i = 0; i < V; i++)
        fin >> need[i];
    fin >> G;
    for (int i = 0; i < G; i++)
        for (int j = 0; j < V; j++)
            fin >> scoop[i][j];
    
    for (int num = 1; num <= G; num++) {
        for (int x = 0; x < (1 << G); x++) {
            if (__builtin_popcount(x) != num) continue;
            memset(sum, 0, sizeof(sum));
            for (int i = 0; i < G; i++) {
                if ((x & (1 << i)) == 0) continue;
                for (int j = 0; j < V; j++)
                    sum[j] += scoop[i][j];
            }
            bool ok = true;
            for (int i = 0; i < V; i++) {
                if (sum[i] < need[i]) {
                    ok = false;
                    break;
                }
            }
            if (ok) {
                fout << num;
                for (int i = 0; i < G; i++)
                    if (x & (1 << i))
                        fout << ' ' << i + 1;
                fout << endl;
                return 0;
            }
        }
    }
    return 0;
}
```