---
title: 'USACO 1.5.2: Mother''s Milk（BFS）'
urlname: usaco-1-5-2-mother-s-milk
toc: true
date: 2018-12-11 20:34:37
updated: 2019-01-07 20:20:00
tags: [USACO, alg:Breadth-first Search, alg:Depth-first Search]
---

## 题意

见[P1215 \[USACO1.4\]母亲的牛奶 Mother's Milk](https://www.luogu.org/problemnew/show/P1215)。

有三个容量分别是A，B，C的桶，A，B，C是1到20的整数。初始状态下A和B都是空的，C是满的。可以从一个桶向另一个桶内倒牛奶，直到倒满或者倒空为止。问A为空时C中所有可能的牛奶体积。

## 分析

非常基础的搜索题。用DFS和BFS都可以。搜就是了……

一种简化存储状态的方法是，只存两个桶内的牛奶数量，因为牛奶总数是不变的。（不过这个简化看起来没什么意思。）

我看了看题解里所谓的动态规划方法，发现它做的事情与其说是“动态规划”，还不如说是类似于Bellman-Ford的不断松弛，发现一次松弛就继续更新……

## 代码

```cpp
/*
ID: zhanghu15
TASK: milk3
LANG: C++14
*/

#include <iostream>
#include <fstream>
#include <vector>
#include <algorithm>
#include <cstring>

using namespace std;

int A, B, C;
int limit[3];
bool states[21][21][21];
bool ans[21];

void dfs(int s[3]) {
    if (s[0] == 0) ans[s[2]] = true;
    for (int i = 0; i < 3; i++) {
        for (int j = 0; j < 3; j++) {
            if (i == j) continue;
            // pour from i to j
            if (s[i] == 0 || s[j] == limit[j]) continue;
            int ns[3];
            if (s[i] <= limit[j] - s[j]) {
                ns[i] = 0;
                ns[j] = s[j] + s[i];
                ns[3-i-j] = s[3-i-j];
            }
            else {
                ns[i] = s[i] - (limit[j] - s[j]);
                ns[j] = limit[j];
                ns[3-i-j] = s[3-i-j];
            }
            if (states[ns[0]][ns[1]][ns[2]]) continue;
            states[ns[0]][ns[1]][ns[2]] = true;
            dfs(ns);
        }
    }
}

int main() {
    ofstream fout("milk3.out");
    ifstream fin("milk3.in");
    fin >> limit[0] >> limit[1] >> limit[2];
    int s[3] = {0, 0, limit[2]};
    dfs(s);
    for (int i = 0; i < limit[2]; i++)
        if (ans[i])
            fout << i << ' ';
    fout << limit[2] << endl;
    return 0;
}
```