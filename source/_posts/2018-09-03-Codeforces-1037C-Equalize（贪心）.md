---
title: Codeforces 1037C. Equalize（贪心）
urlname: codeforces-1037c-equalize
toc: true
date: 2018-09-03 18:34:42
updated: 2018-09-03 18:45:00
tags: [Codeforces, Codeforces Contest, alg:Greedy]
---

目来源：[https://codeforces.com/contest/1037/problem/B](https://codeforces.com/contest/1037/problem/B)

提交次数：1/2

## 题意

给定两个长度相同的只含有`0`和`1`的串`a`和串`b`，可以对`a`进行下列两种操作之一：

* 交换`i`和`j`处的位，代价为`|i - j|`
* 将一位翻转，代价为1

问对`a`执行操作使它变成`b`的最小代价是多少。

## 分析

这道题足够简单，所以很快一遍过了。

---

很显然交换位的代价太大了，只有当两个需要交换的位相邻的时候，代价才比分别翻转这两个位小。所以我就先扫描了一遍`a`，将所有适合交换的相邻（这两位不同，且与`b`中对应位都不同）交换，然后再扫描一遍，翻转所有不同的位。

当然事实上扫描一遍就够了。[^solution]顺序考虑每一位：

* 如果这一位和下一位可以交换，`sum++`，直接跳过下一位
* 如果不能交换，翻转这一位，`sum++`

以及，从题解里发现了加速`ostream`的方法，即取消和`stdio`的同步：

```cpp
ios_base::sync_with_stdio(false); cin.tie(nullptr); cout.tie(nullptr);
```

[^solution]: [https://codeforces.com/contest/1037/submission/42406848](https://codeforces.com/contest/1037/submission/42406848)

## 代码

```cpp
#include <iostream>
#include <cstdio>
using namespace std;
char a[1000005];
char b[1000005];
int main() {
    int n;
    scanf("%d", &n);
    scanf("%s", a);
    scanf("%s", b);
    int sum = 0;
    for (int i = 1; i < n; i++) {
        if (a[i-1] != b[i-1] && a[i] != b[i] && a[i-1] != a[i]) {
            swap(a[i], a[i-1]);
            sum++;
        }
    }
    for (int i = 0; i < n; i++)
        if (a[i] != b[i])
            sum++;

    cout << sum << endl;
    return 0;
}
```
