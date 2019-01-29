---
title: 'USACO 2.1.5: Hamming Codes（枚举？）'
urlname: usaco-2-1-5-hamming-codes
toc: true
date: 2019-01-29 22:49:29
updated: 2019-01-29 22:49:29
tags: [USACO, alg:Brute Force]
---

## 题意

见[P1461 海明码 Hamming Codes](https://www.luogu.org/problemnew/show/P1461)。

找出`N`个数字，每个数字的长度均为`B`位，且每两个数之间的海明距离都至少为`D`。如果有多个解，则输出排序后值最小的解。

## 分析

当然，我可以直接写个`O(2^(2^B))`的枚举加剪枝，而且的确能过。不过这是因为数据太弱了，而且刻意出的这么弱。

```
------- test 1 [length 7 bytes] ----
16 7 3
------- test 2 [length 6 bytes] ----
2 7 7
------- test 3 [length 6 bytes] ----
3 6 4
------- test 4 [length 6 bytes] ----
5 5 1
------- test 5 [length 7 bytes] ----
10 8 4
------- test 6 [length 7 bytes] ----
20 6 2
------- test 7 [length 7 bytes] ----
32 5 1
------- test 8 [length 7 bytes] ----
50 8 2
------- test 9 [length 7 bytes] ----
60 7 2
------- test 10 [length 7 bytes] ----
16 8 3
------- test 11 [length 7 bytes] ----
15 8 4
```

在上述测试数据下，暴力方法的运行时间为：

```
Compiling...
Compile: OK

Executing...
   Test 1: TEST OK [0.004 secs, 1388 KB]
   Test 2: TEST OK [0.004 secs, 1308 KB]
   Test 3: TEST OK [0.004 secs, 1328 KB]
   Test 4: TEST OK [0.004 secs, 1392 KB]
   Test 5: TEST OK [0.000 secs, 1312 KB]
   Test 6: TEST OK [0.000 secs, 1292 KB]
   Test 7: TEST OK [0.000 secs, 1304 KB]
   Test 8: TEST OK [0.000 secs, 1292 KB]
   Test 9: TEST OK [0.000 secs, 1304 KB]
   Test 10: TEST OK [0.000 secs, 1392 KB]
   Test 11: TEST OK [0.000 secs, 1308 KB]

All tests OK.
```

来个`N=64, B=8, D=4`这样的数据，暴力解法得跑到天荒地老。

我的一种想法是，把这`2^B`个数字当做图的结点，数字之间的海明距离当做边权来建图，然后这个问题就变成了类似于找最大团。但是问题是我们需要找的不只是一个最大团，而是一个排序后数字最小的最大团，好像就不符合一般的问题范畴了。[^max-clique]

[^max-clique]: [wikipedia - Clique problem](https://en.wikipedia.org/wiki/Clique_problem)

总之今天也是不想学习最大团算法的一天……

## 代码

如果把每次的判断`__builtin_popcount(code[i] ^ next)`都换成查表，常数应该能减少一点，但是并不能改变这个算法复杂度实在太高的事实……

```cpp
/*
ID: zhanghu15
TASK: hamming
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

int N, B, D;
int code[64];
int MAX;
bool found;

void dfs(int n) {
    if (n == N) {
        found = true;
        return;
    }
    int next;
    if (n == 0) next = 0;
    else next = code[n - 1] + 1;
    for (; next <= MAX; next++) {
        if (found) return;
        bool ok = true;
        for (int i = 0; i < n; i++) {
            if (__builtin_popcount(code[i] ^ next) < D) {
                ok = false;
                break;
            }
        }
        if (!ok) continue;
        code[n] = next;
        dfs(n + 1);
    }
}

int main() {
    ofstream fout("hamming.out");
    ifstream fin("hamming.in");
    fin >> N >> B >> D;
    MAX = (1 << B) - 1;
    dfs(0);
    for (int i = 0; i < N; i++) {
        fout << code[i];
        if (i % 10 == 9 || i == N - 1)
            fout << endl;
        else
            fout << ' ';
    }
    return 0;
}
```