---
title: 'USACO 1.4.5: Wormholes（暴力）'
urlname: usaco-1-4-5-wormholes
toc: true
date: 2018-12-03 20:34:27
updated: 2018-12-08 20:55:00
tags: [USACO, alg:Brute Force]
---

## 题意

见[USACO 2013 December Contest, Bronze Problem 3. Wormholes](http://www.usaco.org/index.php?page=viewproblem2&cpid=360)。

原来USACO是会新加题的吗！！因为是新题所以好像洛谷没有……

在一个平面上有最多12个虫洞（偶数个），虫洞两两相连，可以互相传送；Bessie在平面上只会向+x方向移动。问有多少种虫洞连接方式会让Bessie困在循环中，

## 分析

这道题我做了好久，设计得相当不错，真是相当不错，我本来以为是暴力水题的来着……

需要注意的一点是，Bessie是向+x方向走，所以能够相互走到的是相邻且y坐标相等的虫洞，不是x坐标相等的虫洞……我最开始就写错了，后来就直接把x和y换了一下。

遍历虫洞两两相连的方法可以直接用回溯法解决。问题显然在于得到一个连接顺序之后，怎么判定有环。最开始我直接把这个问题建模成了“判断一张图上是否有环”，图上的边就是相连的虫洞和能相互走到的虫洞。然后我就挂了。问题在于，“连接虫洞的边”和“走路能到的边”不是一种边，在实际判定有环的时候，必须交替着走，而且判定有环的时候，不止要看虫洞是否重复，也要看上一条边是否重复，因为显然，走路到达一个虫洞（然后传送）和传送到达一个虫洞（然后开始走路）是两种不同的情况。

这些问题我搞了一晚上……然后错了三次，我注意到USACO左侧开始有一个类似于"3/10"的标签了，这是否意味着交10次就不能再交了（还是交满10次自动出题解）？

题解里又有一个[视频](https://www.youtube.com/watch?v=KR4iY-EfEs4)（大概因为是新题吧），里面的解法很巧妙：既然必须交替走两种边，那我一次就走两条边，传送1次+走路一次……这写法真简明扼要。

## 代码

```cpp
/*
ID: zhanghu15
TASK: wormhole
LANG: C++14
*/

#include <iostream>
#include <fstream>
#include <vector>
#include <algorithm>
#include <cstring>

using namespace std;

int N;
pair<int, int> wormholes[12];
int walkPairing[12];
int wormholePairing[12];
bool visited[12][2];  // 记录走/跳是必要的
int ans;

// 两类边交替走，找环
bool dfs(bool isWalking, int x) {
    if (visited[x][isWalking]) return true;
    visited[x][isWalking] = true;
    if (isWalking && walkPairing[x] != -1)
        return dfs(!isWalking, walkPairing[x]);
    else if (!isWalking)
        return dfs(!isWalking, wormholePairing[x]);
    return false;
}

void backtrack(int cur) {
    // 找到了一组合法的配对
    if (cur == N) {
        // 尝试DFS找环
        for (int i = 0; i < N; i++) {
            memset(visited, 0, sizeof(visited));
            if (dfs(true, i)) {
                ans++;
                break;
            }
        }
        return;
    }
    if (wormholePairing[cur] != -1)
        backtrack(cur + 1);
    else {
        for (int i = cur + 1; i < N; i++) {
            if (wormholePairing[i] != -1) continue;
            wormholePairing[cur] = i;
            wormholePairing[i] = cur;
            backtrack(cur + 1);
            wormholePairing[cur] = wormholePairing[i] = -1;
        }
    }
}

int main() {
    ofstream fout("wormhole.out");
    ifstream fin("wormhole.in");
    fin >> N;
    // x和y坐标搞反了……
    for (int i = 0; i < N; i++)
        fin >> wormholes[i].second >> wormholes[i].first;
    // 找能走到的虫洞
    sort(wormholes, wormholes+N);
    memset(walkPairing, -1, sizeof(walkPairing));
    for (int i = 1; i < N; i++)
        if (wormholes[i-1].first == wormholes[i].first)
            walkPairing[i-1] = i;
    // 寻找所有可能的虫洞配对方法
    memset(wormholePairing, -1, sizeof(wormholePairing));
    backtrack(0);

    fout << ans << endl;
    return 0;
}
```