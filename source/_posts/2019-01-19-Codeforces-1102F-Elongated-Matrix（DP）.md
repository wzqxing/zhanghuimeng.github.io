---
title: Codeforces 1102F. Elongated Matrix（DP）
urlname: codeforces-1102f-elongated-matrix
toc: true
date: 2019-01-19 16:59:52
updated: 2019-01-21 23:57:00
tags: [Codeforces, Codeforces Contest, alg:Dynamic Programming, alg:Binary Search, alg:Bitmasks]
---

题目来源：[https://codeforces.com/contest/1102/problem/F](https://codeforces.com/contest/1102/problem/F)

提交次数：2/3

## 题意

给定一个`n`（`1<=n<=16`）行`m`列的矩阵，重新排列矩阵的各行，使得逐列遍历矩阵时相邻两个数之差绝对值的最小值最大。

## 分析

还是一道状态压缩DP。首先计算出每两行之间的距离（注意第一行和最后一行的距离）。

然后对于每种状态，枚举可能的起始结点，最大化路径上边权重的最小值。状态总数是`n * 2^n`（末尾结点\*状态总数），枚举起始结点的代价为`O(n)`，枚举上一个结点的代价是`O(n)`，总计算复杂度是`O(n^3 * 2^n)`。如果不用递归还要多一个`O(n)`，不知道有没有解决这个问题的方法。

另一种比较神奇的方法是二分答案，然后在图上寻找回路。题解里的具体方法是这样的：枚举起始点`i`，对于每个点`j`，寻找一条从`i`到`j`的经过了所有结点的路径，然后再判断`j`到`i`是否有路径。题解[^sln]里这种方法的复杂度是`O(n^2 * log(MAXN) * 2^n)`。但是我并不会题解里的写法，所以我写出来的还是平凡的`O(n^3 * 2^n)`的求哈密顿回路法……

[^sln]: [Codeforces Round #531 (Div. 3) Editorial](https://codeforces.com/blog/entry/64439)

## 代码

### 迭代写法的DP

相当慢……

```cpp
#include <iostream>
#include <cmath>
#include <cstdio>
using namespace std;
int a[16][10000];
int nextOk[16][16];
int firstLastOk[16][16];
int f[16][16][1 << 16];
int n, m, k;
int main() {
    cin >> n >> m;
    for (int i = 0; i < n; i++)
        for (int j = 0; j < m; j++)
            scanf("%d", &a[i][j]);
    for (int i = 0; i < n; i++) {
        for (int j = i + 1; j < n; j++) {
            int x = 1e9;
            for (int k = 0; k < m; k++) {
                x = min(x, (int) abs(a[i][k] - a[j][k]));
                if (x == 0) break;
            }
            nextOk[i][j] = nextOk[j][i] = x;
        }
    }
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            int x = 1e9;
            for (int k = 1; k < m; k++) {
                x = min(x, (int) abs(a[j][k-1] - a[i][k]));
                if (x == 0) break;
            }
            firstLastOk[i][j] = x;
        }
    }

    if (n == 1) {
        cout << firstLastOk[0][0] << endl;
        return 0;
    }

    int ans = 0;
    // 枚举路径长度
    for (int len = 1; len <= n; len++) {
        for (int status = 0; status < (1 << n); status++) {
            if (__builtin_popcount(status) != len) continue;
            for (int first = 0; first < n; first++) {
                if (!(status & (1 << first))) continue;
                if (len == 1) {
                    f[first][first][status] = 1e9;
                    continue;
                }
                for (int cur = 0; cur < n; cur++) {
                    if (!(status & (1 << cur)) || first == cur) continue;
                    f[first][cur][status] = 0;
                    for (int last = 0; last < n; last++) {
                        if (!(status & (1 << last)) || cur == last) continue;
                        int x = min(f[first][last][status ^ (1 << cur)], nextOk[last][cur]);
                        if (len == n) x = min(x, firstLastOk[first][cur]);
                        f[first][cur][status] = max(f[first][cur][status], x);
                    }
                    if (len == n) ans = max(f[first][cur][status], ans);
                }
            }
        }
    }
    cout << ans << endl;
    return 0;
}
```

### 递归写法的DP+二分答案

因为常数写多了所以甚至更慢了……

```cpp
#include <iostream>
#include <cmath>
#include <cstring>
#include <cstdio>
using namespace std;
int a[16][10000];
int nextOk[16][16];
int firstLastOk[16][16];
int f[16][1 << 16];
bool g[16][16];
int n, m, k;

int calc(int x, int state) {
    if (f[x][state] != -1) return f[x][state];
    f[x][state] = 0;
    for (int i = 0; i < n; i++) {
        if (i != x && (state & (1 << i)) && g[i][x] && calc(i, state ^ (1 << x))) {
            f[x][state] = 1;
            break;
        }
    }
    return f[x][state];
}

bool isOk(int thres) {
    memset(g, 0, sizeof(g));
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            if (i != j && nextOk[i][j] >= thres)
                g[i][j] = true;
        }
    }
    
    for (int i = 0; i < n; i++) {
        memset(f, -1, sizeof(f));  // 因为这句memset位置错了，调了一阵子
        // 控制只能从i开始
        for (int j = 0; j < n; j++)
            f[j][1 << j] = j == i ? 1 : 0;
        for (int j = 0; j < n; j++) {
            if (firstLastOk[i][j] >= thres) {
                if (calc(j, (1 << n) - 1)) return true;
            }
        }
    }
    return false;
}

int main() {
    cin >> n >> m;
    for (int i = 0; i < n; i++)
        for (int j = 0; j < m; j++)
            scanf("%d", &a[i][j]);
    for (int i = 0; i < n; i++) {
        for (int j = i + 1; j < n; j++) {
            int x = 1e9;
            for (int k = 0; k < m; k++) {
                x = min(x, (int) abs(a[i][k] - a[j][k]));
                if (x == 0) break;
            }
            nextOk[i][j] = nextOk[j][i] = x;
        }
    }
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            int x = 1e9;
            for (int k = 1; k < m; k++) {
                x = min(x, (int) abs(a[j][k-1] - a[i][k]));
                if (x == 0) break;
            }
            firstLastOk[i][j] = x;
        }
    }

    if (n == 1) {
        cout << firstLastOk[0][0] << endl;
        return 0;
    }

    // 二分答案
    // 这个写法是从题解里抄的（感觉也不是很优雅啊）
    // <del>谁叫你也写不出优雅的</del>
    int l = 0, r = 1e9;
    while (l < r - 1) {
        int m = (l + r) >> 1;
        if (isOk(m)) l = m;
        else r = m;
    }
    if (isOk(r)) cout << r << endl;
    else cout << l << endl;
    return 0;
}
```