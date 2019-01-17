---
title: Codeforces 1102C. Doors Breaking and Repairing（博弈）
urlname: codeforces-1102c-doors-breaking-and-repairing
toc: true
date: 2019-01-17 16:34:56
updated: 2019-01-17 16:46:00
tags: [Codeforces, Codeforces Contest, alg:Games]
---

题目来源：[https://codeforces.com/contest/1102/problem/C](https://codeforces.com/contest/1102/problem/C)

提交次数：1/1

## 题意

有`n`扇门，每扇门的耐久度是`a[i]`，两个人轮流对这些门做一些操作：

* 我可以任选一扇门，如果它的耐久度是`b[i]`，则可以将它降低到`max(0, b[i]-x)`
* 之后对方可以选择一扇门，如果它的耐久度是`b[i]`且`b[i] > 0`，则可以将它增加到`b[i]+y`
* 两人轮流进行游戏

游戏共有`10^100`轮。问游戏结束后耐久度变成0的门的数量最多是多少？

## 分析

这道题说是博弈论，其实没啥博弈的……（或者说没有什么复杂的算法）

如果`x > y`，那么最后肯定是我方完全胜利（这么多轮，无论对方怎么加耐久，我们都可以和对方对敲，然后把所有门都敲爆……）。

如果`x <= y`，事情就变得有些复杂了。如果我们任选一扇门来敲（且没有敲到0），那么对方在下一轮就可以把它补好，甚至比之前还牢靠（`y > x`）；所以我们选能敲到0的门才有效果。对方的最佳策略显然就是把那些耐久值`<=x`的门赶紧补齐，补到一下敲不坏为止。所以我方能最终敲坏的门的数量就是耐久值`<=x`的门的总数除2（取ceil）（因为是我方先开始游戏，可以多敲坏一扇门）。

## 代码

```cpp
#include <iostream>
using namespace std;
int a[100];
int main() {
    int n, x, y, cnt = 0;
    cin >> n >> x >> y;
    for (int i = 0; i < n; i++) {
        cin >> a[i];
        if (a[i] <= x) cnt++;
    }
    if (x > y) cout << n << endl;
    else cout << (cnt + 1) / 2 << endl;
    return 0;
}
```