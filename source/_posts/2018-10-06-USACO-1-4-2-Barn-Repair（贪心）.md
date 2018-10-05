---
title: 'USACO 1.4.2: Barn Repair（贪心）'
urlname: usaco-1-4-2-barn-repair
toc: true
date: 2018-10-06 01:19:53
updated: 2018-10-06 01:19:53
tags: [USACO, alg:Greedy]
---

## 题意

见[洛谷 P1209](https://www.luogu.org/problemnew/show/P1209)。

## 分析

这道题就是[上回的文章](/post/greedy-algorithm-usaco-translation)里的例题，所以具体的算法内容和正确性就不再讲了。总之我是这样实现的：

* 统计所有空隙（同时排除两侧的空牛棚，因为它们显然不需要覆盖）
* 排序
* 根据木板的数量，从牛棚总数中依次减去最大的空隙

其中我没有注意到这一点：木板的数量可能会超过所有连续牛棚的数量，结果数组越界了，WA了一次。

## 代码

```cpp
/*
ID: zhanghu15
TASK: barn1
LANG: C++14
*/

#include <iostream>
#include <fstream>
#include <vector>
#include <algorithm>

using namespace std;

bool occupied[205];

int main() {
    ofstream fout("barn1.out");
    ifstream fin("barn1.in");
    int M, S, C;
    fin >> M >> S >> C;
    for (int i = 0; i < C; i++) {
        int x;
        fin >> x;
        occupied[x] = true;
    }

    int start = -1;
    int ans = S;
    vector<int> gaps;
    for (int i = 1; i <= S; i++) {
        if (start == -1 && !occupied[i]) start = i;
        else if (start != -1 && occupied[i]) {
            if (start == 1) ans -= i - start;
            else gaps.push_back(i - start);
            start = -1;
        }
    }
    if (start != -1) ans -= S + 1 - start;

    // do the greedy (note that gaps can be exhausted...)
    sort(gaps.begin(), gaps.end());
    for (int i = 0; i < M - 1 && i < gaps.size(); i++)
        ans -= gaps[gaps.size() - i - 1];
    fout << ans << endl;
    return 0;
}
```
