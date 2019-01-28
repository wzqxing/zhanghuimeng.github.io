---
title: Codeforces 1107C. Brutality（贪心），及比赛（Educational Codeforces Round 59）总结
urlname: codeforces-1107c-brutality-and-educational-codeforces-round-59-summary
toc: true
date: 2019-01-28 18:06:49
updated: 2019-01-28 18:06:49
tags: [Codeforces, Codeforces Contest, alg:Greedy]
---

题目来源：[https://codeforces.com/contest/1107/problem/C](https://codeforces.com/contest/1107/problem/C)

提交次数：1/1

## 题意

给定`n`个英文小写字符和对应的`n`个值，按某个字符就可以得到对应的值，不能连续按某个字符超过`k`次，问得到的值的总和最大是多少。

## 分析

这次比赛我做得乱七八糟。从提交数量来看，C应该是道很简单的题，但是我却做不出来。事后我发现我看错题了。

PS. 我决定以后CF比赛中我能做出来的题就不写题解了。不然题目实在太多了。。。

---

对于一串连续的字符，如果它的长度超过了`k`，那么从里面挑`k`个最大值就行。没了！

（比赛的时候我以为是要把这个串分割成若干个长度小于`k`的串。）

[soln1]: [Codeforces Blog - Quick unofficial editorial for Educational Round 59 (Div. 2)](https://codeforces.com/blog/entry/64833)

[soln2]: [Codeforces Blog - Educational Codeforces Round 59 Editorial](https://codeforces.com/blog/entry/64847)

## 代码

```cpp
#include <iostream>
#include <algorithm>
using namespace std;
typedef long long LL;
int k, n;
LL a[200005];
char s[200005];
int main() {
    cin >> n >> k;
    for (int i = 0; i < n; i++) {
        cin >> a[i];
    }
    cin >> s;
    LL ans = 0;
    int start = 0;
    for (int i = 1; i <= n; i++) {
        if (i == n || s[i] != s[i-1]) {
            if (i - start > k)
                sort(a + start, a + i);
            for (int j = 0; j < k && i-j-1 >= start; j++)
                ans += a[i - j - 1];
            start = i;
        }
    }
    cout << ans << endl;
    return 0;
}
```