---
title: 'USACO 1.3.4: Palindromic Squares'
urlname: usaco-1-3-4-palindromic-squares
toc: true
date: 2018-08-24 00:04:19
updated: 2018-08-24 00:10:19
tags: [USACO, alg:Math]
---

## 题意

见[洛谷 P1206](https://www.luogu.org/problemnew/show/P1206)。

## 分析

直接枚举就可以。进制转换不是很难，但是我遇到了一个小bug：在<=4.8.0的MinGW中，`stoi`和`to_string`等几个函数是没法直接用的。[^bug]这个问题有解决方案，但是我懒得去修了，总之我发现可以直接把`char`赋给`string`。

[^bug]: [Bug 52015 - std::to_string does not work under MinGW](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=52015)

## 代码

```cpp
/*
ID: zhanghu15
TASK: palsquare
LANG: C++14
*/

#include <iostream>
#include <fstream>
#include <string>
#include <map>
#include <algorithm>

using namespace std;

map<int, string> digitMap;

void init() {
    // 可以把char赋给string
    for (int i = 0; i < 10; i++)
        digitMap[i] = char(i + '0');  // to_string does not work, a known issue
    for (int i = 10; i <= 20; i++)
        digitMap[i] = (char) (i - 10 + 'A');
}

string base10ToB(int B, int x) {
    string baseB;
    if (x == 0)
        return "0";
    while (x > 0) {
        baseB = digitMap[x % B] + baseB;
        x /= B;
    }
    return baseB;
}

bool checkIsPalindrome(string x) {
    int n = x.length();
    for (int i = 0; i + i < n; i++)
        if (x[i] != x[n - i - 1])
            return false;
    return true;
}

int main() {
    ofstream fout("palsquare.out");
    ifstream fin("palsquare.in");
    int B;
    fin >> B;

    init();

    for (int i = 1; i <= 300; i++) {
        string square = base10ToB(B, i * i);
        if (checkIsPalindrome(square)) {
            fout << base10ToB(B, i) << ' ' << square << endl;
        }
    }

    return 0;
}
```
