---
title: 'USACO 1.2.4: Broken Necklace'
urlname: usaco-1-2-4-broken-necklace
toc: true
date: 2018-08-21 23:06:53
updated: 2018-08-21 23:27:00
tags: [USACO, alg:Dynamic Programming]
---

## 题意

见[洛谷 P1203](https://www.luogu.org/problemnew/show/P1203)。

## 分析

因为数据量只有350，所以完全可以直接暴力：枚举每一个打破项链的位置，然后分别向左和向右模拟取同色珠子的过程，复杂度是`O(N^2)`。有一些可以简化模拟过程的小技巧：

* 把珠子的字符串复制两遍。好吧，这一点其实有一点tricky（所以我在代码里写的时候实际上复制了三份），但是我觉得复制两遍还是足够的。
* 不是枚举每一个打破的位置，而是模拟从当前位置开始，允许打破一次能得到的最大长度。本质上没有差异，但据说会更好写一些。
* 不判断枚举两侧时是否相互覆盖，因为一旦发生相互覆盖，说明必然可以数完全部珠子，只要把最终结果和`N`取最小值即可。

当然用动态规划的方法也是可以的，对于每个点，计算向左和向右分别最多能收集多少红色和蓝色的珠子，从左开始扫一遍，再从右开始扫一遍即可。

关于“复制两遍为何足够”的一些想法：通过这一遍的复制，环上所有可能的圈都展开成了`2*N`数组中的一个子线段，因此最优解之一必然会被上述模拟过程覆盖到。动态规划法也是这样。这件事看起来是显然的，但我写代码的时候显然并没有想清楚。

## 代码

### 模拟

```cpp
/*
ID: zhanghu15
TASK: beads
LANG: C++14
*/

#include <iostream>
#include <fstream>
#include <string>
#include <cstring>

using namespace std;


int main() {
    ofstream fout("beads.out");
    ifstream fin("beads.in");
    int N;
    char tmp[355], beads[1200];
    fin >> N;
    fin >> tmp;
    for (int i = 0; i < N; i++)
        beads[i] = beads[i + N] = beads[i + 2 * N] = tmp[i];

    int maxSum = -1;
    // 遍历每一个可以break的位置
    for (int i = N; i < 2 * N; i++) {
        // break before i
        char color = 'w';
        int sum = 0;
        for (int j = 0; j < N; j++) {
            if (beads[i + j] != 'w') {
                if (color == 'w')
                    color = beads[i + j];
                else if (color != beads[i + j])
                    break;
            }
            sum++;
        }
        color = 'w';
        for (int j = 1; j <= N; j++) {
            if (beads[i - j] != 'w') {
                if (color == 'w')
                    color = beads[i - j];
                else if (color != beads[i - j])
                    break;
            }
            sum++;
        }
        if (sum > N)
            sum = N;

        maxSum = max(maxSum, sum);
    }

    fout << maxSum << endl;
    return 0;
}
```

### DP

```cpp
/*
ID: zhanghu15
TASK: beads
LANG: C++14
*/

#include <iostream>
#include <fstream>
#include <string>
#include <cstring>

using namespace std;


int main() {
    ofstream fout("beads.out");
    ifstream fin("beads.in");
    int N;
    char tmp[355], beads[800];  // 开两倍其实就足够计算了
    fin >> N;
    fin >> tmp;
    for (int i = 0; i < N; i++)
        beads[i] = beads[i + N] = tmp[i];

    // break before i
    // 注意语义！
    int left_red[800], left_blue[800], right_red[800], right_blue[800];
    left_red[0] = left_blue[0] = 0;
    for (int i = 1; i < 2 * N; i++) {
        if (beads[i - 1] == 'r') {
            left_red[i] = left_red[i - 1] + 1;
            left_blue[i] = 0;
        }
        else if (beads[i - 1] == 'b') {
            left_red[i] = 0;
            left_blue[i] = left_blue[i - 1] + 1;
        }
        else {
            left_red[i] = left_red[i - 1] + 1;
            left_blue[i] = left_blue[i - 1] + 1;
        }
    }

    // 因为是break before i，所以可以从右端不存在处开始初始化
    right_red[2 * N] = right_blue[2 * N] = 0;
    for (int i = 2 * N - 1; i >= 0; i--) {
        if (beads[i] == 'r') {
            right_red[i] = right_red[i + 1] + 1;
            right_blue[i] = 0;
        }
        else if (beads[i] == 'b') {
            right_red[i] = 0;
            right_blue[i] = right_blue[i + 1] + 1;
        }
        else {
            right_red[i] = right_red[i + 1] + 1;
            right_blue[i] = right_blue[i + 1] + 1;
        }
    }

    // 不需要特判超过N的情况，因为超过N则必然可以达到N了
    int maxSum = -1;
    for (int i = 0; i < 2 * N; i++)
        maxSum = max(maxSum, max(left_red[i], left_blue[i]) + max(right_red[i], right_blue[i]));

    fout << min(maxSum, N) << endl;

    return 0;
}
```
