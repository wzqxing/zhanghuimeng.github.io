---
title: 'USACO 2.1.2: Ordered Fractions（数学）'
urlname: usaco-2-1-2-ordered-fractions
toc: true
mathjax: true
date: 2019-01-29 20:52:21
updated: 2019-01-29 20:52:21
tags: [USACO, alg:Math]
---

## 题意

见[洛谷 P1458 顺序的分数 Ordered Fractions](https://www.luogu.org/problemnew/show/P1458)。

给定`N`，求`[0, 1]`范围内分母小于等于`N`的所有既约分数，按大小排序。（`1 <= N <= 160`）

## 分析

作为一道普通的题……可以说它是考察运算符重载之类的花样的（至少对C++选手是这样）。当然也可以写得更简单一些。总之，我就直接枚举了所有这个范围内的分数，把所有既约分数找出来，排了个序，然后输出。

### 法里数列（Farey sequence）

当然，这道题的题解里还有另一个生成解法：

* 首先生成列表`[0/1, 1/1]`
* 在列表的每两个分数之间插入一个新的分数，其分子和分母各为相邻两个分数分子和分母的和；于是得到：`[0/1, 1/2, 1/1]`
* 以此类推：`[0/1, 1/3, 1/2, 2/3, 1/1]`
* 继续以此类推：`[0/1, 1/4, 1/3, 2/5, 1/2, 3/5, 2/3, 3/4, 1/1]`
* 直到新生成的分数的分母都大于`N`为止

我觉得这个方法很神奇。通过非常简单的数学推导就可以说明，将两个分数的分子和分母分别相加，得到的新分数的值位于这两个分数之间：

已知$\frac{a_1}{b_1} < \frac{a_2}{b_2}$，即$a_1b_2 < a_2b_1$；

两侧都加上$a_1b_1$，化简后得到$\frac{a_1}{b_1} < \frac{a_1 + a_2}{b_1 + b_2}$；

另一侧同理。

当然，这并不能说明很多其他的问题，比如为什么这种生成方法生成的都是既约分数，而且是不重不漏的。这时候就需要使用[Farey sequence](https://en.wikipedia.org/wiki/Farey_sequence)的一些性质了。`n`阶的Farey sequence就是我们题目里所求的序列。它的一个性质是这样的：如果$a/b$和$c/d$在某阶的Farey sequence中是相邻的，那么当Farey sequence的阶增加，它们之间出现的第一项就是$(a+c)/(b+d)$，且这一项会第一次出现在$b + d$阶的数列中。这个生成算法就利用了这一性质：它每次生成的并不严格是某一阶的Farey sequence，而可能会多一些项出来；不过这并不重要，因为它们迟早会出现的。

还有一些详细的分析就不写了，总之这个想法很有趣。

## 代码

### 普通的代码

```cpp
/*
ID: zhanghu15
TASK: frac1
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

struct Frac {
    // 我着实不想去记分子和分母的英文了……
    int up;
    int down;

    Frac() {
        up = down = 0;
    }

    Frac(int x, int y) {
        up = x;
        down = y;
    }

    friend bool operator < (const Frac& f1, const Frac& f2) {
        return f1.up * f2.down < f1.down * f2.up;
    }
};

int gcd(int a, int b) {
    if (b == 0) return a;
    return gcd(b, a % b);
}

Frac f[160000];
int N;

int main() {
    ofstream fout("frac1.out");
    ifstream fin("frac1.in");
    fin >> N;
    int n = 0;
    for (int i = 1; i <= N; i++) {
        for (int j = 0; j <= i; j++) {
            if (gcd(i, j) > 1) continue;
            f[n++] = Frac(j, i);
        }
    }
    sort(f, f + n);
    for (int i = 0; i < n; i++)
        fout << f[i].up << '/' << f[i].down << endl;
    return 0;
}
```

### Farey数列生成器

```cpp
/*
ID: zhanghu15
TASK: frac1
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

ofstream fout("frac1.out");
ifstream fin("frac1.in");

int N;

void farey(int a1, int b1, int a2, int b2) {
    if (b1 + b2 > N) return;
    farey(a1, b1, a1 + a2, b1 + b2);
    fout << (a1 + a2) << '/' << (b1 + b2) << endl;
    farey(a1 + a2, b1 + b2, a2, b2);
}

int main() {
    fin >> N;
    fout << "0/1" << endl;
    farey(0, 1, 1, 1);
    fout << "1/1" << endl;
    return 0;
}
```