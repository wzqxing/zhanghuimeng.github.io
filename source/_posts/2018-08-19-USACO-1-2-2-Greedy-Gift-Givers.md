---
title: 'USACO 1.2.2: Greedy Gift Givers'
urlname: usaco-1-2-2-greedy-gift-givers
toc: true
date: 2018-08-19 02:13:02
updated: 2018-08-19 02:16:00
tags: [USACO]
---

## 题意

见[洛谷 P1201](https://www.luogu.org/problemnew/show/P1201)。

## 分析

一道简单的模拟题，特别是使用STL之后，变得更加简单。可能需要稍微判断一下，需要分配礼物的人数为0的情况，不要出现除零错之类的。

## 代码

```cpp
/*
ID: zhanghu15
TASK: gift1
LANG: C++14
*/

#include <iostream>
#include <fstream>
#include <string>
#include <map>

using namespace std;

int main() {
    ofstream fout("gift1.out");
    ifstream fin("gift1.in");
    int NP;
    string nameInOrder[10];
    map<string, int> bank;
    fin >> NP;
    for (int i = 0; i < NP; i++) {
        fin >> nameInOrder[i];
        bank[nameInOrder[i]] = 0;
    }

    for (int i = 0; i < NP; i++) {
        string giverName;
        fin >> giverName;
        int money, NG;
        fin >> money >> NG;
        if (NG > 0) {
            int moneyToGive = money / NG;
            int moneyLeft = money - NG * moneyToGive;
            bank[giverName] += moneyLeft - money;

            for (int j = 0; j < NG; j++) {
                string receiverName;
                fin >> receiverName;
                bank[receiverName] += moneyToGive;
            }
        }
    }

    for (int i = 0; i < NP; i++)
        fout << nameInOrder[i] << ' ' << bank[nameInOrder[i]] << endl;

    return 0;
}
```
