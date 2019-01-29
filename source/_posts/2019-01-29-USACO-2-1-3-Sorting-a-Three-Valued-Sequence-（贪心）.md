---
title: 'USACO 2.1.3: Sorting a Three-Valued Sequence（贪心）'
urlname: usaco-2-1-3-sorting-a-three-valued-sequence
toc: true
date: 2019-01-29 21:36:00
updated: 2019-01-29 21:36:00
tags: [USACO, alg:Greedy]
---

## 题意

见[洛谷 P1459 三值的排序 Sorting a Three-Valued Sequence](https://www.luogu.org/problemnew/show/P1459)。

给定一个只包含1，2，3的数组，问将这个数组排序最少需要多少次交换。

## 分析

这个好像是之前的[翻译：贪心算法（USACO）](/post/greedy-algorithm-usaco-translation)的原题……所以做法也没什么好说的了。首先找出一次交换可以解决的“错位”二元组的数量，然后再找出需要两次交换才能解决的“错位”三元组的数量即可。只要仔细一想就可以发现，每种二元组和三元组的数量是固定的，只跟每个“区域”（排序之后某种元素应该占的位置）内的其他元素的数量相关。

以及我觉得这道题和[Codeforces 1102D. Balanced Ternary String](/post/codeforces-1102d-balanced-ternary-string)有一点点像。（虽然像的程度很有限就是了……）

## 代码

```cpp
/*
ID: zhanghu15
TASK: sort3
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

int N;
int a[1000];
int num[4];
int cnt[4][4];

int main() {
    ofstream fout("sort3.out");
    ifstream fin("sort3.in");
    fin >> N;
    for (int i = 0; i < N; i++) {
        fin >> a[i];
        num[a[i]]++;
    }
    int start = 0;
    // cnt[i][j]：在i“区域”内j的数量
    for (int i = 1; i <= 3; i++) {
        for (int j = 0; j < num[i]; j++) {
            cnt[i][a[start + j]]++;
        }
        start += num[i];
    }
    int swaps = 0;
    // 一次交换能解决的数所需交换次数
    for (int i = 1; i <= 3; i++) {
        for (int j = 1; j <= 3; j++) {
            if (i == j) continue;
            // 尽可能地将i区域内的j和j区域内的i相交换
            int x = min(cnt[i][j], cnt[j][i]);
            swaps += x;
            cnt[i][j] -= x;
            cnt[j][i] -= x;
        }
    }
    // 两次交换能解决的数所需交换次数
    for (int i = 1; i <= 3; i++) {
        for (int j = 1; j <= 3; j++) {
            if (i == j) continue;
            int k = 6 - i - j;
            // 尽可能地将i区域内的j和j区域内的k交换，然后再和k区域内的i交换
            int x = min(min(cnt[i][j], cnt[j][k]), cnt[k][i]);
            swaps += x * 2;
            cnt[i][j] -= x;
            cnt[j][k] -= x;
            cnt[k][i] -= x;
        }
    }
    fout << swaps << endl;
    return 0;
}
```