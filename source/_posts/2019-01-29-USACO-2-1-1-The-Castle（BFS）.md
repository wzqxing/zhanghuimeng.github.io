---
title: 'USACO 2.1.1: The Castle（DFS）'
urlname: usaco-2-1-1-the-castle
toc: true
date: 2019-01-29 20:30:43
updated: 2019-01-29 20:30:43
tags: [USACO, alg:Depth-first Search]
---

## 题意

见[洛谷 P1457 城堡 The Castle](https://www.luogu.org/problemnew/show/P1457)。

有一个`M*N`的方格图（`1 <= M,N <= 50`），其中方格外侧有一整圈墙，有些方格线上有墙。问这些墙形成了多少个房间，以及去掉一堵墙能形成的最大房间的面积？

## 分析

应该可以说是标准的Floodfill算法了。算法流程如下：

* 对于每一个格子，如果它还没有被访问到，则房间数+1，并从该格子开始DFS
* 在DFS过程中，将访问到的每一个格子都标为当前房间数（于是立刻可以得到共有多少个房间）
* 对于每一堵墙，判断它两侧的格子是否属于不同房间，如果是，则拆掉这面墙可以将这两个房间连起来，且新房间的面积是原来两个房间的和

Floodfill的时间复杂度是`O(M*N)`，枚举墙的时间复杂度也是`O(M*N)`，因此总复杂度为`O(M*N)`。

## 代码

其他部分都没有什么难的，但是在答案去重这部分遇到了一点小问题。题目要求以某个格子北侧或东侧的形式打印需要的墙，且：

* 选择拆除后得到的房间面积最大的墙
* 如果面积相同，则选择最靠西的**格子**
* 如果同样靠西，则选择最靠北的**格子**
* 如果同样靠北，则优先选择格子东侧的墙

这可真是复杂的要求……

所以我也写了一堆复杂的判断条件……

顺带一提，我终于意识到现在USACO题目左边（可能会出现）的数字是你已经通过了多少个样例了……

```cpp
/*
ID: zhanghu15
TASK: castle
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

int M, N;

// west, north, east, south
int mx[4] = { 0, -1, 0, 1};
int my[4] = {-1,  0, 1, 0};
int module[50][50];
int visited[50][50];
int cnt[2501];
int n;

void dfs(int x, int y) {
    for (int i = 0; i < 4; i++) {
        if (module[x][y] & (1 << i)) continue;
        int nx = x + mx[i], ny = y + my[i];
        if (nx < 0 || nx >= N || ny < 0 || ny >= M) continue;
        if (visited[nx][ny]) continue;
        visited[nx][ny] = n;
        cnt[n]++;
        dfs(nx, ny);
    }
}

int main() {
    ofstream fout("castle.out");
    ifstream fin("castle.in");

    fin >> M >> N;
    for (int i = 0; i < N; i++)
        for (int j = 0; j < M; j++)
            fin >> module[i][j];
    
    for (int i = 0; i < N; i++)
        for (int j = 0; j < M; j++) {
            if (!visited[i][j]) {
                n++;
                visited[i][j] = n;
                cnt[n]++;
                dfs(i, j);
            }
        }
    fout << n << endl;
    int maxn = -1;
    for (int i = 1; i <= n; i++)
        maxn = max(maxn, cnt[i]);
    fout << maxn << endl;
    maxn = -1;
    char ch;
    int x, y;
    // 事实证明光靠顺序去重不太可行
    for (int i = 0; i < N; i++) {
        for (int j = 0; j < M; j++) {
            if (j < M - 1 && visited[i][j] != visited[i][j+1]) {
                int sum = cnt[visited[i][j]] + cnt[visited[i][j+1]];
                if (sum > maxn || (sum == maxn && (j < y || (j == y && i > x)))) {
                    maxn = sum;
                    ch = 'E';
                    x = i + 1;
                    y = j + 1;
                }
            }
            if (i > 0 && visited[i-1][j] != visited[i][j]) {
                int sum = cnt[visited[i][j]] + cnt[visited[i-1][j]];
                if (sum > maxn || (sum == maxn && (j < y || (j == y && i > x)))) {
                    maxn = sum;
                    ch = 'N';
                    x = i + 1;
                    y = j + 1;
                }
            }
        }
    }
    fout << maxn << endl;
    fout << x << ' ' << y << ' ' << ch << endl;
    return 0;
}
```
