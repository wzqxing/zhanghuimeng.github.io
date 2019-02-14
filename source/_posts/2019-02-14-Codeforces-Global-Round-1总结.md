---
title: Codeforces Global Round 1总结
urlname: codeforces-global-round-1
toc: true
date: 2019-02-14 14:38:41
updated: 2019-02-14 14:38:41
tags: [Codeforces]
categories: Codeforces
---

因为写总结和做题实在太艰难了，我决定以后CF每场比赛只写一篇总结……

[^sln]: [The Editorial of the First Codeforces Global Round](https://codeforces.com/blog/entry/65079)

## 1110A

题目来源：[https://codeforces.com/contest/1110/problem/A](https://codeforces.com/contest/1110/problem/A)

提交次数：1/1

### 题意

### 分析

### 代码

## 1110D

题目来源：[https://codeforces.com/contest/1110/problem/D](https://codeforces.com/contest/1110/problem/D)

提交次数：1/1

### 题意

给定若干个1和m之间的整数，定义triplet为形如`(x, x, x)`或`(x, x+1, x+2)`的元组，问这些整数最多一共能组成多少个triplet？

### 分析

这道题有一个非常必要的观察：如果有大于等于三个的形如`[x, x+1, x+2]`的triplet，那么完全可以把其中的三个替换成`[x, x, x]`，`[x+1, x+1, x+1]`和`[x+2, x+2, x+2]`。这也就意味着对于每一个`x`，形如`[x-2, x-1, x]`的triplet最多只有2个。[^sln]

然后我就想了一种错漏百出的方法，调了一个下午，终于调出来了。

令`f[i][x][y]`表示考虑前`i-1`个数组成的triplet，`a[i-1]`还剩`x`个，`a[i-2]`还剩`y`个时的triplet总数。我心想：既然连续的triplet最多只有2个，那么`x`和`y`的上限就都是2。（后来事实证明这很离谱）于是列出向后的转移方程：

```cpp
// 不组成形如(i-2, i-1, i)的triplet
if (a[i] >= 0)
    f[i+1][(a[i] - 0) % 3][x - 0] =
        max(f[i+1][(a[i]- 0) % 3][x - 0], f[i][x][y] + 0 + (a[i] - 0) / 3);
// 组成一个形如(i-2, i-1, i)的triplet
if (a[i] >= 1 && x >= 1 && y >= 1)
    f[i+1][(a[i] - 1) % 3][x - 1] =
        max(f[i+1][(a[i] - 1) % 3][x - 1], f[i][x][y] + 1 + (a[i] - 1) / 3);
// 组成两个形如(i-2, i-1, i)的triplet
if (a[i] >= 2 && x >= 2 && y >= 2)
    f[i+1][(a[i] - 2) % 3][x - 2] =
        max(f[i+1][(a[i] - 2) % 3][x - 2], f[i][x][y] + 2 + (a[i] - 2) / 3);
```

跑一下看看，发现错得离谱。思考了一段时间之后，发觉这个转移方程“太准确了”。比如说，`a[4]=2`时，`f[4][2][2]`可以转移到`f[5][1][1]`，但也应该可以转移到`f[5][0][1]`和`f[5][1][0]`（丢掉一些4和3也是可以的）。于是把转移方程修改如下，加入了后面两维的更多可能性：

```cpp
if (a[i] >= 0) {
    for (int k = 0; k <= min(a[i], 2); k++) {
        for (int j = 0; j <= x - 0; j++) {
            f[i+1][k][j] = 
                max(f[i+1][k][j], f[i][x][y] + 0 + (a[i] - k) / 3);
        }
    }
}
if (a[i] >= 1 && x >= 1 && y >= 1) {
    for (int k = 0; k <= min(a[i] - 1, 2); k++) {
        for (int j = 0; j <= x - 1; j++) {
            f[i+1][k][j] = 
                max(f[i+1][k][j], f[i][x][y] + 1 + (a[i] - k - 1) / 3);
        }
    }
}
if (a[i] >= 2 && x >= 2 && y >= 2) {
    for (int k = 0; k <= min(a[i] - 2, 2); k++) {
        f[i+1][k][x - 2] = 
            max(f[i+1][k][x - 2], f[i][x][y] + 2 + (a[i] - k - 2) / 3);
    }
}
```

结果还是不对。经过更加漫长的debug，我意识到这种做法里的上限不能是2，因为它表示的是整体剩下的上限，而不是一共有多少个以它结尾的triplet。于是我直接把上限改成了6（既然有三种triplet的可能性），然后就过了（虽然耗时非常长）。

### 代码

```cpp
#include <iostream>
#include <cstring>
using namespace std;
int n, m;
int cnt[1000005];
int f[1000005][7][7];
int main() {
    cin >> n >> m;
    int *a = cnt + 1;  // 处理i-2的边缘情况
    for (int i = 0; i < n; i++) {
        int x;
        cin >> x;
        a[x]++;
    }
    int ans = 0;
    for (int i = 1; i <= m + 1; i++) {
        for (int x = 0; x <= min(6, a[i-1]); x++)
            for (int y = 0; y <= min(6, a[i-2]); y++) {
                if (a[i] >= 0) {
                    for (int k = 0; k <= min(a[i], 6); k++) {
                        for (int j = 0; j <= x - 0; j++) {
                            f[i+1][k][j] = 
                                max(f[i+1][k][j], f[i][x][y] + 0 + (a[i] - k) / 3);
                        }
                    }
                }
                if (a[i] >= 1 && x >= 1 && y >= 1) {
                    for (int k = 0; k <= min(a[i] - 1, 6); k++) {
                        for (int j = 0; j <= x - 1; j++) {
                            f[i+1][k][j] = 
                                max(f[i+1][k][j], f[i][x][y] + 1 + (a[i] - k - 1) / 3);
                        }
                    }
                }
                if (a[i] >= 2 && x >= 2 && y >= 2) {
                    for (int k = 0; k <= min(a[i] - 2, 6); k++) {
                        f[i+1][k][x - 2] = 
                            max(f[i+1][k][x - 2], f[i][x][y] + 2 + (a[i] - k - 2) / 3);
                    }
                }
            }
    }
    for (int x = 0; x <= 6; x++)
        for (int y = 0; y <= 6; y++)
            ans = max(ans, f[m + 2][x][y]);
    cout << ans << endl;
    return 0;
}
```