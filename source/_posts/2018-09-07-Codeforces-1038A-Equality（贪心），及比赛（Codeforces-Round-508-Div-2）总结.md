---
title: 'Codeforces 1038A. Equality（贪心），及比赛（Codeforces Round #508 Div. 2）总结'
urlname: codeforces-1038a-equality-and-contest-codeforces-round-508-div-2
toc: true
date: 2018-09-07 21:22:32
updated: 2018-09-07 21:40:32
tags: [Codeforces, Codeforces Contest, alg:Greedy]
---

题目来源：[http://codeforces.com/contest/1040/problem/A](http://codeforces.com/contest/1040/problem/A)

提交次数：1/1

## 题意

给定一个长度为`n`的字符串`s`，其中只包含前`k`个大写字母。定义一个“好”的子序列为其中这`k`个字母的出现频率都相等的子序列，问`s`的“好”的子序列的最大长度是多少。

## 分析

这次一共有6个题。前3题都很水；后3题也相对比较简单（当然还是有点难度的，但仍然比之前的比赛更简单一些）。做（水）完前3题之后（大概花了半个小时），我发现第5题的形式之前曾经见过（[SGU 101](https://codeforces.com/problemsets/acmsguru/problem/99999/101)，模型的抽象方法是一样的，但那道题要求把欧拉路算出来，我还没做），大概能做出来，于是决定写上一番。

写了一个小时之后，我交了，结果挂在了pretest 26上。我百思不得其解（过了25个test呢，而且估计也不是因为数据量大才挂的），也不知道该怎么debug，最后就没去管它了。

后来比赛结束之后我看了好久不同的人的各种题解，终于理解我挂哪了，这是一个比较微妙的错误，的确是有个细节没有想清楚。

这次的排名是1486 / 7631，Rating增加了4。（虽然我其实不知道CF现在的Rating是怎么算的。）

---

这道题显然非常水。只需要找到这`k`个字母中出现次数最少的字母，然后必然可以构造出字母出现频率相等且最大的子序列，返回`k`乘以最小出现次数就可以了。

## 代码

```cpp
#include <iostream>
using namespace std;
char a[100005];
int h[26];
int main() {
    int n, k;
    cin >> n >> k;
    cin >> a;
    for (int i = 0; i < n; i++)
        h[a[i] - 'A']++;
    int minn = h[0];
    for (int i = 0; i < k; i++)
        minn = min(minn, h[i]);
    cout << minn * k << endl;
    return 0;
}
```
