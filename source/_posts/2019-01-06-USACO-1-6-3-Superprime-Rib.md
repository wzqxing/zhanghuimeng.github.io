---
title: 'USACO 1.6.3: Superprime Rib'
urlname: usaco-1-6-3-superprime-rib
toc: true
date: 2019-01-06 20:27:39
updated: 2019-01-06 20:51:39
tags: [USACO, alg:Math]
---

## 题意

见[洛谷 P1218 \[USACO1.5\]特殊的质数肋骨 Superprime Rib](https://www.luogu.org/problemnew/show/P1218)。

定义super prime为每个前缀都是质数的数。例：7，73，733，7331都是质数，所以7331是super prime。给定`N`，问在所有长度为`N`的数里，有哪些数是super prime？

## 分析

这其实是一道搜索题。先搜索出长度为`N-1`的所有super prime，然后在这些数后面分别加上1，3，7，9（去掉2和5的倍数），判断得到的数是否为质数，然后就得到了长度为`N`的super prime。super prime的总数并不多，所以直接这样搜就可以了，也没有更多需要剪枝的。

题解的方法比我还暴力。我先打了个1-10000的质数表；题解根本没管这个，直接DFS搜索树，然后对每个数暴力枚举2, 3, 5, 7, 9...是否是它的因子……

查了一下，这个数列是有正式定义的（只不过正式定义并不叫super prime，[super-prime](https://en.wikipedia.org/wiki/Super-prime)是别的东西），叫做Right-truncatable primes，在OEIS上的编号为[A024770](https://oeis.org/A024770)。好像没有什么特别有趣的性质。[^oeis]

[^oeis]: [OEIS - A024770 Right-truncatable primes: every prefix is prime.](https://oeis.org/A024770)

## 代码

```cpp
/*
ID: zhanghu15
TASK: sprime
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

int prime[10000];
int n;
bool isPrime[10000];
void init() {
    memset(isPrime, 1, sizeof(isPrime));
    isPrime[1] = false;
    for (int i = 2; i < 10000; i++) {
        if (isPrime[i]) {
            prime[n++] = i;
            for (int j = i * i; j < 10000; j += i)
                isPrime[j] = false;
        }
    }
}

bool checkIsPrime(int x) {
    if (x < 10000) return isPrime[x];
    for (int i = 0; i < n; i++)
        if (x % prime[i] == 0)
            return false;
    return true;
}

int main() {
    ofstream fout("sprime.out");
    ifstream fin("sprime.in");

    fin >> N;
    init();

    vector<int> sprimes = {2, 3, 5, 7};
    for (int i = 2; i <= N; i++) {
        vector<int> next;
        for (int x: sprimes) {
            for (int j = 1; j <= 9; j++) {
                int y = x * 10 + j;
                if (checkIsPrime(y))
                    next.push_back(y);
            }
        }
        sprimes = next;
    }

    for (int x: sprimes) fout << x << endl;

    return 0;
}
```