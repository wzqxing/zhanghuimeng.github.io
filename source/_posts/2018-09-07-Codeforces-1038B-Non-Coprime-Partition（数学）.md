---
title: Codeforces 1038B. Non-Coprime Partition（数学）
urlname: codeforces-1038b-non-coprime-partition
toc: true
date: 2018-09-07 21:51:49
updated: 2018-09-08 10:15:00
tags: [Codeforces, Codeforces Contest, alg:Math]
---

题目来源：[http://codeforces.com/contest/1038/problem/B](http://codeforces.com/contest/1038/problem/B)

提交次数：1/2

## 题意

给定正整数`n`，要求将`1`到`n`的数分成两个各自非空的集合`S1`和`S2`，且`gcd(sum(S1), sum(S2)) > 1`。任意合法的解均可接受。

## 分析

这道题是个水题。但我一开始根本就没看清输出要求（要先输出集合的大小，再输出集合中的元素），所以WA了一次。

---

很显然有很多种分法，所以一个直觉是，对于一般的`n`，必然存在一组合法的解。由于`1 + 2 + ... + n = n*(n+1) / 2`，不妨这样构造：

* 当`n = 1, 2`时，显然没有合法的分法。
* 当`n`为大于1的奇数时，`n`必然是`n*(n+1) / 2`的约数，那么不妨把`n`分成一组（`S1`），其他数分成另一组，此时`sum(S2) = 1 + 2 + ... + (n-1) = (n-1)*n / 2`，`gcd(sum(S1), sum(S2)) = n`。
* 当`n`为大于2的偶数时，`n / 2`必然是`n*(n+1) / 2`的约数，那么不妨把`n / 2`分为一组，其他数分为另一组，同理可得此时`gcd(sum(S1), sum(S2)) = n / 2`。

## 代码

```cpp
#include <iostream>
using namespace std;
int main() {
    int n;
    cin >> n;
    if (n <= 2) {
        cout << "No" << endl;
        return 0;
    }
    if (n % 2 == 1) {
        cout << "Yes" << endl << 1 << ' ' << n << endl;
        cout << n - 1;
        for (int i = 1; i < n; i++)
            cout << ' ' << i;
        cout << endl;
    }
    else {
        cout << "Yes" << endl << 1 << ' ' << n / 2 << endl;
        cout << n - 1;
        for (int i = 1; i <= n; i++)
            if (i != n / 2)
                cout << ' ' << i;
        cout << endl;
    }
    return 0;
}
```
