---
title: Codeforces 1102E. Monotonic Renumeration（数组）
urlname: codeforces-1102e-monotonic-renumeration
toc: true
date: 2019-01-17 17:08:19
updated: 2019-01-19 16:52:00
tags: [Codeforces, Codeforces Contest, alg:Array]
---

题目来源：[https://codeforces.com/contest/1102/problem/E](https://codeforces.com/contest/1102/problem/E)

提交次数：1/1

## 题意

给定一个长度为`n`的数组`a`，要求为`a`生成一个对应的数组`b`，满足：

* `b[0] = 0`
* 对于任意`0 <= i < j <= n`，如果满足`a[i] == a[j]`，必有`b[i] == b[j]`（不过`a[i] != a[j]`时也可能有`b[i] == b[j]`）
* 任取`0 <= i < n - 1`，必有`b[i] = b[i+1]`或`b[i] + 1 = b[i+1]`

问共有多少种可能的`b`。

## 分析

显然`b[i]`是一个递增序列，因此可以自然推出，若`a[i] == a[j]`，则必有`b[i] == b[i+1] == ... = b[j]`，也就是说，对于`a`中任意位置两个相等的元素，它们在`b`中对应的是一整段相等的元素。显然这种元素相等是可能会发生重叠的，因此一个自然的想法就是，把重复的元素建模成线段，然后合并发生overlap的线段以得到相等元素的最长长度。

我的做法是，从后向前遍历`a`，如果发现当前元素和后面的元素重复了，则取index最靠后的元素，组成一条线段，插入到栈中与其他元素合并；否则把它自己的index作为一条线段插入到栈中。最后栈中留下的就是几条互不相交（且并组成了整个区间）的线段。

对于（除了第一条之外）每条线段，我们可以选择让它的值和前一条相等，也可以选择让它的值是前一条+1。每种选择都会导致生成一种新的`b`。于是结果是`2^{线段数-1}`。

例子：对于`a = {1, 2, 1, 2, 3}`，1对应的线段是`[0, 2]`，2对应的线段是`[1, 3]`，3对应的线段是`[4, 4]`；合并之后得到两条线段，`[0, 3]`和`[1, 4]`；只有两种`b`，分别是`{0, 0, 0, 0, 0}`和`{0, 0, 0, 0, 1}`。

## 代码

```cpp
#include <iostream>
#include <vector>
#include <map>
using namespace std;
int a[200005];
int n;

typedef long long int LL;
const LL P = 998244353;

LL pow2(LL x) {
    LL pow = 2, ans = 1;
    while (x > 0) {
        if (x & 1)
            ans = (ans * pow) % P;
        pow = (pow * pow) % P;
        x >>= 1;
    }
    return ans;
}

int main() {
    map<int, int> indMap;
    vector<pair<int, int>> s;
    cin >> n;
    for (int i = 0; i < n; i++) {
        cin >> a[i];
        if (indMap.find(a[i]) == indMap.end()) {
            indMap[a[i]] = i;
        }
    }
    for (int i = n - 1; i >= 0; i--) {
        pair<int, int> interval;
        if (indMap.find(a[i]) != indMap.end() && indMap[a[i]] < i) {
            interval = make_pair(indMap[a[i]], i);
        }
        else {
            interval = make_pair(i, i);
        }
        if (!s.empty() && s.back().first <= interval.first && s.back().second >= interval.second)
            continue;
        if (!s.empty() && interval.second >= s.back().first) {
            interval.second = s.back().second;
            s.pop_back();
            s.push_back(interval);
        }
        if (s.empty() || interval.second < s.back().first)
            s.push_back(interval);
    }


    int cnt = 0;
    if (!s.empty() && s.front().second < n - 1) cnt++;
    if (!s.empty() && s.back().first > 0) cnt++;
    for (int i = 0; i < s.size(); i++) {
        cnt++;
        // 本条线段和前一条线段之间的间隔
        if (i > 0 && s[i - 1].second < s[i].first - 1)
            cnt++;
    }
    cout << pow2(cnt - 1) << endl;
    return 0;
}
```