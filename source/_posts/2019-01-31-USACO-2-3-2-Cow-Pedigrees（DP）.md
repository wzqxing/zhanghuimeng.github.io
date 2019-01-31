---
title: 'USACO 2.3.2: Cow Pedigrees（DP）'
urlname: usaco-2-3-2-cow-pedigrees
toc: true
date: 2019-01-31 01:32:06
updated: 2019-01-31 01:32:06
tags: [USACO, alg:Dynamic Programming]
---

## 题意

见[洛谷 P1472 奶牛家谱 Cow Pedigrees](https://www.luogu.org/problemnew/show/P1472)。

求所有高度为`K`，结点数量为`N`的二叉树个数。

## 分析

我在这道题上花了好久！之前Leetcode上有道类似的题（[Leetcode 894. All Possible Full Binary Trees](/post/leetcode-894-all-possible-full-binary-trees)），不过那道题是要生成所有完全二叉树，而不是计数。

一个很简单的思路是这样的：

* 用`f[n][k]`记录高度为`k`，结点数量为`n`的二叉树数量
* 固定左子树高度为`k-1`，枚举左子树结点个数和右子树高度（显然左子树结点数量确定后，右子树结点数量也确定了）
* （适当地）乘2（因为只固定了左子树的高度，差不多只考虑了一半情况；但是去重还需要仔细考虑……）

但是去重就出了一点问题。最开始我直接把所有结果都乘2，结果发现显然乘多了。比如说，左右两侧高度和结点数都相同的情况就不需要乘2。但是这样仍然不对。经过了比较漫长的手算和debug过程，我发现，左右两侧高度相同的情况都不需要乘2——因为它们都会出现在枚举中。举个例子：

![四个二叉树](btree.jpg)

`f[5][3]`和`f[7][3]`都会作为左边的二叉树出现在枚举中。

---

题解里的思路就稍微好（不容易出错）一点：分别枚举左边比右边深，右边比左边深和左右一样深的三种情况。这个时候如果还需要明确地按子树的不同高度进行枚举未免太麻烦，所以就用一个辅助数组`g[n][k]`存所有高度`<= k`且结点数量为`n`的二叉树的数量。

不过整体复杂度仍然都是`O(N^3)`的。

吐槽1：我之前的确很少（如果不是没有）见到要求结果模9901的。

吐槽2：这道题的文件名是`nocows`，听起来和NOCOW很像啊……

## 代码

### 我的DP

```cpp
/*
ID: zhanghu15
TASK: nocows
LANG: C++14
*/

#include <iostream>
#include <fstream>
#include <sstream>
#include <vector>
#include <algorithm>
#include <cstring>
#include <cmath>

using namespace std;
typedef long long int LL;

ofstream fout("nocows.out");
ifstream fin("nocows.in");

const LL P = 9901;
LL f[201][101];  // f[i][j]：i个结点的高度为j的二叉树的数量
int N, K;

LL calc(int num, int height) {
    if (f[num][height] != -1) return f[num][height];
    if (num == 1 && height == 1) {
        f[num][height] = 1;
        return 1;
    }
    if (num < 2*height - 1) {
        f[num][height] = 0;
        return 0;
    }
    if (height <= 0) {
        f[num][height] = 0;
        return 0;
    }
    f[num][height] = 0;
    // 一侧结点数为s，高度为height-1
    // 另一侧结点数为num-s-1，高度<=height-1
    for (int s = 1; s <= num - 2; s += 2) {
        if (calc(s, height-1) == 0) continue;
        for (int h = 1; h <= height - 1; h++) {
            f[num][height] += calc(s, height-1) * calc(num-s-1, h);
            // 去重的正确姿势好难……
            if (h != height-1)
                 f[num][height] += calc(s, height-1) * calc(num-s-1, h);
            f[num][height] %= P;
        }
    }
    return f[num][height];
}

int main() {
    fin >> N >> K;
    memset(f, -1, sizeof(f));
    fout << calc(N, K) << endl;
    return 0;
}
```

### 题解的DP

```cpp
/*
ID: zhanghu15
TASK: nocows
LANG: C++14
*/

#include <iostream>
#include <fstream>
#include <sstream>
#include <vector>
#include <algorithm>
#include <cstring>
#include <cmath>

using namespace std;
typedef long long int LL;

ofstream fout("nocows.out");
ifstream fin("nocows.in");

const LL P = 9901;
LL f[201][101];  // f[i][j]：i个结点的高度为j的二叉树的数量
LL g[201][101];  // g[i][j]：i个结点的高度<=j的二叉树的数量
int N, K;

int main() {
    fin >> N >> K;
    f[1][1] = 1;
    // 像这样要用到辅助数组的时候，就不如写bottom-up的迭代形式了
    for (int i = 1; i <= N; i += 2) {
        for (int j = 1; j <= K; j++) {
            for (int k = 1; k <= i - 2; k++) {
                // 左边深度为j-1，右边深度<=j-2
                f[i][j] += f[k][j - 1] * g[i - k - 1][j - 2];
                // 左边深度<=j-2，右边深度为j-1
                f[i][j] += g[k][j - 2] * f[i - k - 1][j - 1];
                // 两边深度都是j-1
                f[i][j] += f[k][j - 1] * f[i - k - 1][j - 1];
                f[i][j] %= P;
            }
            for (int h = j; h <= K; h++)
                g[i][h] = (g[i][h] + f[i][j]) % P;
        }
    }
    fout << f[N][K] << endl;
    return 0;
}
```