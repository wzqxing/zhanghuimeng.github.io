---
title: 'USACO 1.4.6: Ski Course Design（暴力）'
urlname: usaco-1-4-6-ski-course-design
toc: true
date: 2018-12-03 20:36:31
updated: 2018-12-08 20:56:00
tags: [USACO, alg:Brute Force]
---

## 题意

见[洛谷 P3650](https://www.luogu.org/problemnew/show/P3650)。

有`N`座山，每座山有一个高度（`[0, 100]`范围内）。现在需要修改这些山的高度（每座山只修改一次），使得最高的山和最低的山之间的高度差不大于17；修改高度的代价是高度变化值的平方。问如何修改才能使总代价最小。`N<=1000`。

## 分析

还是暴力。这次题解里给的方法是，枚举所有的长度为17的区间，然后对于每个区间，计算把所有山的高度修改到这个区间内的代价。算法复杂度为`O(N*M)`，其中`M`是区间总数（也就是大约100个，实际84个），已经足够了。

当然事实上可以把复杂度减到更低，用计数排序的方法来统计山的高度，然后对于每个区间，计算把每种高度（可能对应几座山）修改到区间内的代价。复杂度是`O(M^2)`。我是这么写的。

这道题有非暴力的做法吗？我感觉是有的。（当然，现在用不上，但是如果数据范围变大就能用上了。）

## 代码

### 暴力

```cpp
/*
ID: zhanghu15
TASK: skidesign
LANG: C++14
*/

#include <iostream>
#include <fstream>
#include <vector>
#include <algorithm>

using namespace std;

int cnt[101];

int main() {
    ofstream fout("skidesign.out");
    ifstream fin("skidesign.in");
    int N;
    fin >> N;
    for (int i = 0; i < N; i++) {
        int x;
        fin >> x;
        cnt[x]++;
    }
    int ans = -1;
    // take i as the base
    for (int i = 0; i <= 83; i++) {
        int low = i, high = i + 17, cur = 0;
        for (int j = 0; j <= 100; j++) {
            if (cnt[j] == 0) continue;
            if (j < low) cur += (low - j) * (low - j) * cnt[j];
            else if (j > high) cur += (j - high) * (j - high) * cnt[j];
        }
        ans = ans == -1 ? cur : min(ans, cur);
    }
    fout << ans << endl;
    return 0;
}
```

### 非暴力

TBD
