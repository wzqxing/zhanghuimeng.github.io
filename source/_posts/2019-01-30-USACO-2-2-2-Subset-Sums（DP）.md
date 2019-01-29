---
title: 'USACO 2.2.2: Subset Sums（DP）'
urlname: usaco-2-2-2-subset-sums
toc: true
date: 2019-01-30 01:05:30
updated: 2019-01-30 01:05:30
tags: [USACO, alg:Dynamic Programming, alg:Meet in the Middle]
---

## 题意

见[洛谷 P1466 集合 Subset Sums](https://www.luogu.org/problemnew/show/P1466)。

将1到`N`的整数分成两个和相同的集合，问有多少种分法？

## 分析

这道题其实是一道标准的经典DP（只要正确建模）。结果我一看数据范围只有39，第一反应是写了一个Meet-in-the-Middle出来。当然也不是不行……

DP的做法是这样的：把原问题换成，用1到`N`的整数之和表示某个数字，最多有多少种表示方法？然后思路就很简单了，用`f[i][j]`表示用前`i`个整数表示`j`的方法，`f[i][j] = f[i-1][j] + f[i-1][j-i]`。

然后我才意识到对`N`大小的限制主要是为了动态规划的其中一维不要太大……

## 代码

### DP

```cpp
/*
ID: zhanghu15
TASK: subset
LANG: C++14
*/

#include <iostream>
#include <fstream>
#include <vector>
#include <algorithm>
#include <cstring>
#include <cmath>
#include <map>

using namespace std;
typedef long long int LL;

int N;
LL f[40][8000];

int main() {
    ofstream fout("subset.out");
    ifstream fin("subset.in");
    fin >> N;
    int sum = N * (N + 1) / 2;
    if (sum % 2 != 0) {
        fout << 0 << endl;
        return 0;
    }
    f[0][0] = 1;
    for (int i = 1; i <= N; i++) {
        for (int j = 0; j < 8000; j++) {
            f[i][j] = f[i-1][j];
            if (j >= i)
                f[i][j] += f[i-1][j-i];
        }
    }
    fout << f[N][sum / 2] / 2 << endl;
    return 0;
}
```

### Meet-in-the-Middle

```cpp
/*
ID: zhanghu15
TASK: subset
LANG: C++14
*/

#include <iostream>
#include <fstream>
#include <vector>
#include <algorithm>
#include <cstring>
#include <cmath>
#include <map>

using namespace std;
typedef long long int LL;

int main() {
    ofstream fout("subset.out");
    ifstream fin("subset.in");
    int N;
    fin >> N;
    int sum = N * (N + 1) / 2;
    if (sum % 2 != 0) {
        fout << 0 << endl;
        return 0;
    }
    int halfSum = sum / 2;
    int halfN = N / 2;
    // Meet-in-the-Middle
    map<int, int> cntMap;
    // gather [1, halfN] sums
    for (int i = 0; i < (1 << halfN); i++) {
        int sum = 0;
        for (int j = 0; j < halfN; j++)
            if (i & (1 << j))
                sum += j + 1;
        // cout << i << ' ' << sum << endl;
        cntMap[sum]++;
    }
    // gather [halfN+1, N] sums
    LL ans = 0;  // 注意数据范围……
    for (int i = 0; i < (1 << (N - halfN)); i++) {
        int sum = 0;
        for (int j = 0; j < N - halfN; j++)
            if (i & (1 << j))
                sum += j + halfN + 1;
        if (cntMap.find(halfSum - sum) != cntMap.end()) {
            ans += cntMap[halfSum - sum];
        }
    }
    fout << ans / 2 << endl;
    return 0;
}
```