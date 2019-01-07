---
title: 'USACO 1.5.1: Arithmetic Progressions（枚举）'
urlname: usaco-1-5-1-arithmetic-progressions
toc: true
date: 2018-12-11 20:31:07
updated: 2019-01-07 20:00:00
tags: [USACO, alg:Brute Force]
---

## 题意

见[洛谷 P1214 \[USACO1.4\]等差数列 Arithmetic Progressions](https://www.luogu.org/problemnew/show/P1214)。

在双平方数集合（`p^2 + q^2, 0 <= p, q <= M`）中寻找长度为`N`的等差数列。`3 <= N <= 25`，`1 <= M <= 250`。

## 分析

我的做法是这样的。

首先枚举得到所有双平方数并去重和排序，复杂度为`O(M^2)`；并且用数组`ind`记录每个数是否是双平方数，以及如果是，它排序后的index。

枚举公差长度`b`（`1 <= b <= 2*M^2 / (N-1)`）：对每个双平方数，用数组`maxlen`维护以它为结尾的公差为`b`的等差数列的最大长度。从小到大枚举所有双平方数`x`：

* 如果`x - b`不存在，则记`maxlen[x] = 1`（即数列中只有它一个数）
* 如果`maxlen[x] >= N`，则输出首项为`x - (N-1)*b`的等差数列（也即输出末项为`x`的等差数列，这样可以做到不重不漏）
* 如果`x + b`也是双平方数，则将`maxlen[ind[x + b]]`更新为`maxlen[x] + 1`

这种做法的时间复杂度约为`O(M^4 / (N-1))`，时间最长的测试点需要2.558s才能通过。

题解里的第一种算法剪枝的方法有些不同。首先也是预处理出所有的双平方数并排序；然后对于任意两个双平方数，判断它们是否有可能成为一个长度为`N`的等差数列的前两项（判断方法是期望的末项和倒数第二项是否存在），如果有可能，则将这两个数的差记录下来，作为有可能的公差；最后对于每个有可能的公差，列举每个双平方数，判断将这个数作为首项是否存在长度为`N`的等差数列，如果有则输出。

第二种算法也没剪枝，就直接枚举公差和首项，然后判断是否存在长度为`N`的等差数列。

## 代码

```cpp
/*
ID: zhanghu15
TASK: ariprog
LANG: C++14
*/

#include <iostream>
#include <fstream>
#include <vector>
#include <algorithm>
#include <cstring>

using namespace std;

int a[62500];
int maxlen[62500];
bool isBisquare[125001];
int ind[125001];

int main() {
    ofstream fout("ariprog.out");
    ifstream fin("ariprog.in");
    int N, M;
    fin >> N >> M;
    int n = 0;
    for (int i = 0; i <= M; i++)
        for (int j = i; j <= M; j++) {
            isBisquare[i*i + j*j] = true;
        }
    int m = 2*M*M;
    memset(ind, -1, sizeof(ind));
    for (int i = 0; i <= m; i++) {
        if (isBisquare[i]) {
            a[n] = i;
            ind[i] = n;
            n++;
        }
    }
    bool found = false;
    for (int b = 1; b <= m/(N-1); b++) {
        memset(maxlen, 0, sizeof(maxlen));
        for (int i = 0; i < n; i++) {
            if (maxlen[i] == 0) maxlen[i] = 1;
            if (maxlen[i] >= N) {
                fout << a[i] - (N-1)*b << ' ' << b << endl;
                found = true;
            }
            if (a[i]+b <= m && ind[a[i]+b] >= 0) {
                maxlen[ind[a[i]+b]] = maxlen[i] + 1;
            }
        }
    }
    if (!found) fout << "NONE" << endl;
    fin.close();
    fout.close();
    return 0;
}
```