---
title: USACO 1.4.3: Prime Cryptarithm
urlname: usaco-1-4-3-prime-cryptarithm
toc: true
date: 2018-12-03 11:03:04
updated: 2018-12-03 11:03:04
tags: [USACO, alg:Brute Force]
---

## 题意

见[洛谷 P1211](https://www.luogu.org/problemnew/show/P1211)。

## 分析

是个水题。开始时我还觉得会很难写，结果发现，与其正经地去按位搜索，不如直接枚举所有的两位数和三位数，判断它们和它们的乘积的中间结果是否满足题目要求。这样就非常好写了。

对于难（时间复杂度高的）题来说，当然还是得正经搜索，但对于简单题，改成这样简单的搜索反而更好。这是我从[Leetcode 401. Binary Watch](/post/leetcode-401-binary-watch)里学到的。题解也是这个做法。

## 代码

```cpp
/*
ID: zhanghu15
TASK: crypt1
LANG: C++14
*/

#include <iostream>
#include <fstream>
#include <vector>
#include <algorithm>

using namespace std;

bool digits[10];

bool isOk(int number, int length = -1) {
    int i = 0;
    while (number > 0) {
        if (!digits[number % 10]) return false;
        number /= 10;
        i++;
    }
    if (length != -1 && i != length) return false;
    return true;
}

int main() {
    ofstream fout("crypt1.out");
    ifstream fin("crypt1.in");
    int N;
    fin >> N;
    for (int i = 0; i < N; i++) {
        int x;
        fin >> x;
        digits[x] = true;
    }
    int ans = 0;
    for (int mul1 = 111; mul1 <= 999; mul1++) {
        if (!isOk(mul1)) continue;
        for (int mul2 = 11; mul2 <= 99; mul2++) {
            if (!isOk(mul2)) continue;
            int part1 = (mul2 % 10) * mul1;
            if (!isOk(part1, 3)) continue;
            int part2 = (mul2 / 10) * mul1;
            if (!isOk(part2, 3)) continue;
            int res = mul1 * mul2;
            if (!isOk(res, 4)) continue;
            ans++;
        }
    }
    fout << ans << endl;
    return 0;
}
```