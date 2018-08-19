---
title: 'USACO 1.2.3: Friday the Thirteenth'
urlname: usaco-1-2-3-friday-the-thirteenth
toc: true
date: 2018-08-20 02:11:22
updated: 2018-08-20 02:15:00
tags: [USACO]
---

## 题意

见[洛谷 P1202](https://www.luogu.org/problemnew/show/P1202)。

## 分析

比较简单的模拟题（或者说是暴力），但是思路有点绕。在做题的时候，我好好考虑了一下，这个月的13号和下个月的13号之间到底应该差多少天，结论是天数取决于这个月的天数，而和下个月的天数无关：因为我们可以把下个月的13天挪到这个月前面来，这样就凑齐了这个月的所有天数。这好像是一个很简单的结论，但是在日常生活中我从来没有仔细思考过这个问题，总有种“应该是个平均值吧”的错觉。

## 代码

```cpp
/*
ID: zhanghu15
TASK: friday
LANG: C++14
*/

#include <iostream>
#include <fstream>
#include <string>

using namespace std;

int daysInMonth[] = {31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31};

int thirteens[7];

int main() {
    ofstream fout("friday.out");
    ifstream fin("friday.in");
    int N;
    fin >> N;

    int dayOfWeek = 5;  // 1900.1.13是周六
    int year = 1900;
    for (int i = 0; i < N; i++) {
        for (int month = 0; month < 12; month++) {
            thirteens[dayOfWeek]++;
            dayOfWeek += daysInMonth[month];
            if (month == 1 && ((year % 4 == 0 && year % 100 != 0) || year % 400 == 0))
                dayOfWeek++;
            dayOfWeek %= 7;
        }
        year++;
    }

    // 输出顺序是周六，周日，周一，周二……有趣
    // 好吧！USACO不接受行尾空格！
    fout << thirteens[5];
    for (int i = 6; i < 12; i++)
        fout << ' ' << thirteens[i % 7];
    fout << endl;

    return 0;
}
```
