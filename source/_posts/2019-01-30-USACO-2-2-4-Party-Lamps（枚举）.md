---
title: 'USACO 2.2.4: Party Lamps（枚举）'
urlname: usaco-2-2-4-party-lamps
toc: true
date: 2019-01-30 20:07:50
updated: 2019-01-30 20:07:50
tags: [USACO]
---

## 题意

见[洛谷 P1468 派对灯 Party Lamps](https://www.luogu.org/problemnew/show/P1468)。

有`N`盏灯和4个按钮：

* 按钮1：按下后所有灯的状态都会改变
* 按钮2：按下后奇数编号的灯的状态会改变
* 按钮3：按下后偶数编号的灯的状态会改变
* 按钮4：按下后编号模3余1的灯的状态会改变

所有灯起初都是打开的，给定一些灯的终止状态和总共按下按钮的次数，问是否存在对应的终态？如果可能，输出所有可能的终态。

## 分析

显然一个直觉是，同一个按钮只能按0或1次，按多了就和按的次数模2次效果是一样的。所以一共只有`2^4=16`种可能，枚举所有的可能，判断次数模2的取值是否相符以及各个灯的终态是否相符即可。

看了题解，发现这个直觉还不够厉害。事实上每6盏灯的状态都是重复的（6是2和3的公倍数）：

```
press 1: oooooooooooo -> xxxxxxxxxxxx
press 2: oooooooooooo -> xoxoxoxoxoxo
press 3: oooooooooooo -> oxoxoxoxoxox
press 4: oooooooooooo -> xooxooxooxoo
```

所以直接枚举6盏灯的状态就行了。同时可以得到一个推论，编号模6相等的灯的终态必定相同。

## 代码

```cpp
/*
ID: zhanghu15
TASK: lamps
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

int N, C;
int finalState[101];
bool state[101];
vector<string> output;

int main() {
    ofstream fout("lamps.out");
    ifstream fin("lamps.in");
    fin >> N >> C;
    memset(finalState, -1, sizeof(finalState));
    int x;
    while (fin >> x) {
        if (x == -1) break;
        finalState[x] = 1;
    }
    while (fin >> x) {
        if (x == -1) break;
        finalState[x] = 0;
    }
    memset(state, 1, sizeof(state));
    // 这里直接写了一个四层循环……
    for (int b1 = 0; b1 < 2; b1++) {
        if (b1 == 1) {
            for (int i = 1; i <= N; i++)
                state[i] = !state[i];
        }
        for (int b2 = 0; b2 < 2; b2++) {
            if (b2 == 1) {
                for (int i = 1; i <= N; i += 2)
                    state[i] = !state[i];
            }
            for (int b3 = 0; b3 < 2; b3++) {
                if (b3 == 1) {
                    for (int i = 2; i <= N; i += 2)
                        state[i] = !state[i];
                }
                for (int b4 = 0; b4 < 2; b4++) {
                    if (b4 == 1) {
                        for (int i = 1; i <= N; i += 3)
                            state[i] = !state[i];
                    }
                    if (b1 + b2 + b3 + b4 <= C && (C - b1 - b2 - b3 - b4) % 2 == 0) {
                        bool ok = true;
                        for (int i = 1; i <= N; i++) {
                            if (finalState[i] != -1 && state[i] != finalState[i]) {
                                ok = false;
                                break;
                            }
                        }
                        if (ok) {
                            string s;
                            for (int i = 1; i <= N; i++)
                                s += (char)(state[i] + '0');
                            output.push_back(s);
                        }
                    }
                    if (b4 == 1) {
                        for (int i = 1; i <= N; i += 3)
                            state[i] = !state[i];
                    }
                }
                if (b3 == 1) {
                    for (int i = 2; i <= N; i += 2)
                        state[i] = !state[i];
                }
            }
            if (b2 == 1) {
                for (int i = 1; i <= N; i += 2)
                    state[i] = !state[i];
            }
        }
        if (b1 == 1) {
            for (int i = 1; i <= N; i++)
                state[i] = !state[i];
        }
    }
    // 忘了这回事，于是错了一次……
    if (output.size() == 0) {
        fout << "IMPOSSIBLE" << endl;
        return 0;
    }
    sort(output.begin(), output.end());
    for (int i = 0; i < output.size(); i++)
        fout << output[i] << endl;
    return 0;
}
```
