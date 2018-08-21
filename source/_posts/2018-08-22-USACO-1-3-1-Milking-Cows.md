---
title: 'USACO 1.3.1: Milking Cows'
urlname: usaco-1-3-1-milking-cows
toc: true
date: 2018-08-22 00:20:00
updated: 2018-08-22 00:28:00
tags: [USACO]
---

## 题意

见[洛谷 P1204](https://www.luogu.org/problemnew/show/P1204)。

## 分析

看到题目的第一反应是用线段树，但是既然前面的文章说的是搜索，而且数据范围只有`[1, 1000000]`，那是不是直接暴力就能过呢？想了想，`N`的范围是`[1, 5000]`，直接过还是有点悬的。

实际上，完全可以把所有区间排序，然后直接扫描，获得每个有农夫在挤奶的区间，以及每个空闲的区间，然后算出最大区间长度。另一种更好想的做法是，先把所有区间排序，然后把所有能合并的区间合并起来，最后再扫描。

我第一次提交的时候WA了，因为我不知怎的就看错题了，把求最大值看成了求和……不过本质上是一样的。

这无疑是一道很有趣的题。我直觉上会觉得，这种题目应该在离散化之后用线段树来解决；事实证明，离散化的思想就够了，线段树反而是杀鸡用牛刀（因为只需要整个区间上的两种统计量，而且都可以通过`O(N)`的代价计算出来，不需要区间查询或者区间归并什么的）。

## 代码

```cpp
/*
ID: zhanghu15
TASK: milk2
LANG: C++14
*/

#include <iostream>
#include <fstream>
#include <vector>
#include <algorithm>

using namespace std;

int main() {
    ofstream fout("milk2.out");
    ifstream fin("milk2.in");
    int N;
    int startTime, endTime;
    vector<pair<int, int>> times;

    fin >> N;
    for (int i = 0; i < N; i++) {
        fin >> startTime >> endTime;
        times.push_back(make_pair(startTime, endTime));
    }
    sort(times.begin(), times.end());

    // 我傻了……不是要求和啊！是要求最大值啊！
    int milkTimeStart = -1, milkTimeEnd = -1;
    int maxMilkTimeInterval = 0, maxIdleTimeInterval = 0;
    for (pair<int, int> time: times) {
        startTime = time.first;
        endTime = time.second;
        if (milkTimeStart == -1) {
            milkTimeStart = startTime;
            milkTimeEnd = endTime;
        }
        else if (startTime <= milkTimeEnd) {
            // 连续挤奶
            if (endTime > milkTimeEnd)
                milkTimeEnd = endTime;
        }
        else {
            // 中间出现空闲
            maxIdleTimeInterval = max(maxIdleTimeInterval, startTime - milkTimeEnd);
            maxMilkTimeInterval = max(maxMilkTimeInterval, milkTimeEnd - milkTimeStart);
            milkTimeStart = startTime;
            milkTimeEnd = endTime;
        }
    }

    maxMilkTimeInterval = max(maxMilkTimeInterval, milkTimeEnd - milkTimeStart);

    fout << maxMilkTimeInterval << ' ' << maxIdleTimeInterval << endl;
    return 0;
}
```
