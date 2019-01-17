---
title: Codeforces 1102B. Array K-Coloring（贪心）
urlname: codeforces-1102b-array-k-coloring
toc: true
date: 2019-01-17 16:09:08
updated: 2019-01-17 16:28:00
tags: [Codeforces, Codeforces Contest, alg:Greedy]
---

题目来源：[https://codeforces.com/contest/1102/problem/B](https://codeforces.com/contest/1102/problem/B)

提交次数：1/4

## 题意

给定`n`个整数，要求将它们染成`k`种颜色，满足：

* 每种颜色至少出现一次
* 相同的整数不能染成相同的颜色

求任意一种染色方法。

## 分析

比赛的时候被Hack了，不过好像也无所谓，因为被Hack说明本来就做错了（

最开始的做法是这样的：

* 计算每种整数的数量（如果有超过`k`个的，则显然无法满足要求）
* 为每种整数记录当前颜色上限，当前出现过的颜色的upper-bound
* 如果当前整数之前还没有被染过色，则将它对应的初始颜色值设为当前upper-bound；否则进行对应的染色，更新颜色上限和upper-bound
  
显然这种做法存在一个问题：颜色还是会重复，在比较tricky的情况下就会爆掉，比如`n=k`的情况。于是我被hack了。

这个做法的想法本身倒是没有什么问题，在颜色还没有出现完之前，认定每种整数出现颜色的起始值是之前所有类型整数的数量之和就行了。

题解里的做法看起来更巧妙（不过复杂度升高了……），就是先把整个数组排序，然后直接把每个元素染成颜色`i % k`。这样，连续的小于`k`个元素恰好不会被染成重复的颜色。[^sln]这也说明了，判断完整数数量之后，只要`n>=k`，就不会有失败的情况……

[^sln]: [Codeforces Round #531 (Div. 3) Editorial](https://codeforces.com/blog/entry/64439)

## 代码

```cpp
#include <iostream>
#include <cstring>
using namespace std;
int a[5000];
int cnt[5001];
int colorCnt[5001];
int color[5000];
int main() {
    int n, k;
    cin >> n >> k;
    for (int i = 0; i < n; i++) {
        cin >> a[i];
        cnt[a[i]]++;
    }
    // 必然失败的情况
    for (int i = 1; i <= 5000; i++) {
        if (cnt[i] > k) {
            cout << "NO" << endl;
            return 0;
        }
    }

    int nextColor = 1, maxColor = -1;
    for (int i = 0; i < n; i++) {
        // 如果nextColor大于k，说明已经全部出现完了，可以随便染
        if (colorCnt[a[i]] == 0) {
            colorCnt[a[i]] = nextColor > k ? 1 : nextColor;
            nextColor += cnt[a[i]];
        }
        // 模k循环染色
        color[i] = colorCnt[a[i]]++;
        if (colorCnt[a[i]] > k) colorCnt[a[i]] -= k;
        maxColor = max(color[i], maxColor);
    }
    if (nextColor < k) {
        cout << "NO" << endl;
        return 0;
    }
    cout << "YES" << endl;
    for (int i = 0; i < n; i++)
        cout << color[i] << ' ';
    cout << endl;
    return 0;
}
```