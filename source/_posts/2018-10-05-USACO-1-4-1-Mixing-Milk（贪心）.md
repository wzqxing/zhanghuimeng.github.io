---
title: 'USACO 1.4.1: Mixing Milk（贪心）'
urlname: usaco-1-4-1-mixing-milk
toc: true
date: 2018-10-05 03:12:25
updated: 2018-10-05 03:19:25
tags: [USACO, alg:Greedy]
---

## 题意

见[洛谷 P1208](https://www.luogu.org/problemnew/show/P1208)。

## 分析

典型的贪心问题，可以通过对最优解的exchange方法简单地证明，必然是按顺序选择价格从低到高的牛奶是最好的，否则总能找出更优的解。

这道题的Analysis中有很多人的讨论。因为题目对牛奶的价格做了限制（1000），所以可以利用这个来做一些优化。最开始好像有人打算用计数排序来进行优化，然后还写了个链表。很快就有人指出链表根本没有意义，因为价格相同的牛奶完全可以看做是一样的，直接存进一个大小为1000的数组中就行。总之这些优化还是挺有意思的。

## 代码

### 普通排序

```cpp
/*
ID: zhanghu15
TASK: milk
LANG: C++14
*/

#include <iostream>
#include <fstream>
#include <vector>
#include <algorithm>

using namespace std;

int main() {
    ofstream fout("milk.out");
    ifstream fin("milk.in");

    long long int N, M;
    fin >> N >> M;

    vector<pair<int, long long int>> milks;
    for (int i = 0; i < M; i++) {
        int P, A;
        fin >> P >> A;
        milks.emplace_back(P, A);
    }
    sort(milks.begin(), milks.end());

    long long int ans = 0;
    for (int i = 0; i < milks.size(); i++) {
        long long int c = min(N, milks[i].second);
        ans += c * milks[i].first;
        N -= c;
        if (N <= 0) break;
    }

    fout << ans << endl;

    return 0;
}
```

### 计数排序

```cpp
/*
ID: zhanghu15
TASK: milk
LANG: C++14
*/

#include <iostream>
#include <fstream>
#include <vector>
#include <algorithm>

using namespace std;

long long int milkPrice[1005];

int main() {
    ofstream fout("milk.out");
    ifstream fin("milk.in");

    long long int N, M;
    fin >> N >> M;
    for (int i = 0; i < M; i++) {
        int P, A;
        fin >> P >> A;
        milkPrice[P] += A;
    }

    // 1. 用计数排序来代替快排
    // 2. 合并所有价格相同的牛奶
    // 3. 直接进行遍历
    long long int ans = 0;
    for (int i = 0; i <= 1000; i++) {
        if (N >= milkPrice[i]) {
            ans += milkPrice[i] * i;
            N -= milkPrice[i];
        }
        else {
            ans += N * i;
            break;
        }
    }

    fout << ans << endl;
    return 0;
}
```
