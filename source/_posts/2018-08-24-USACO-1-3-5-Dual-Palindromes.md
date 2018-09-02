---
title: 'USACO 1.3.5: Dual Palindromes'
urlname: usaco-1-3-5-dual-palindromes
toc: true
date: 2018-08-24 00:22:43
updated: 2018-08-24 00:29:43
tags: [USACO, alg:Math]
---

## 题意

见[洛谷 P1207](https://www.luogu.org/problemnew/show/P1207)。

## 分析

这道题和上一道用到的代码很像，所以复用了一些。以及，题解里有个很有趣的说法：因为双重回文数很稠密，所以我们可以用暴力算法寻找这些数。但是如何说明双重回文数很普遍呢？这听起来好像循环论证了。写个暴力代码，按最大数据规模跑一遍（对这道题是比较简单的）就知道暴力到底是否可行了。输入`15 9999`时，程序输出：

```
10252
10308
10658
10794
10858
10922
10986
11253
11314
11757
11898
11950
12291
12355
12419
```

说明暴力算法确实是可行的。

（虽然我选择暴力只是因为这个Section的主题是暴力搜索，连想都没想。）

## 代码

```cpp
/*
ID: zhanghu15
TASK: dualpal
LANG: C++14
*/

#include <iostream>
#include <fstream>
#include <string>
#include <map>
#include <algorithm>

using namespace std;

string base10ToB(int B, int x) {
    string baseB;
    if (x == 0)
        return "0";
    while (x > 0) {
        baseB = (char)(x % B + '0') + baseB;
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
    ofstream fout("dualpal.out");
    ifstream fin("dualpal.in");

    int N, S;
    fin >> N >> S;

    for (int i = S + 1; ; i++) {
        int palNum = 0;
        for (int b = 2; b <= 10; b++) {
            if (checkIsPalindrome(base10ToB(b, i)))
                palNum++;
            if (palNum >= 2)
                break;
        }

        if (palNum >= 2) {
            fout << i << endl;
            if (--N == 0)
                break;
        }
    }

    return 0;
}
```
