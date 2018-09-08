---
title: Codeforces 1038C. Gambling（贪心）
urlname: codeforces-1038c-gambling
toc: true
date: 2018-09-08 10:29:01
updated: 2018-09-08 11:31:00
tags: [Codeforces, Codeforces Contest, alg:Greedy]
---

题目来源：[http://codeforces.com/contest/1038/problem/C](http://codeforces.com/contest/1038/problem/C)

提交次数：1/1

## 题意

有两个玩家`A`和`B`，他们各有一个长度相等的数组`a`和`b`，A和B轮流进行以下操作之一：

* 从自己的数组中删除一个数，将这个数加到自己的`sum`中；
* 从对方的数组中删除一个数

每个人的目标都是最大化自己的`sum`和对方的`sum`的差（不是绝对值）。问如果两人都以最优策略行动，`sum_A - sum_B`会是多少？

## 分析

这道题看起来有点像博弈论的问题，但实际上关系好像没有那么大，和贪心关系更大一些。或者说，每个人的最优策略都是贪心策略。

令`sum_A' = sum(a[1]/2 + ... + a[n]/2)`，`sum_B' = sum(b[1]/2 + ... + b[n]/2)`，将`a`和`b`转换成：

```
a' = {a[1]/2, ..., a[n]/2}
b' = {b[1]/2, ..., b[n]/2}
```

此时，若玩家`A`选择`a'`中的元素（不妨设为`a[1]`）并删除，则`sum_A' += a[1]/2`；若玩家`A`选择`b'`中的元素（不妨设为`b[1]`）并删除，则`sum_B' -= b[1]/2`。当`A`和`B`的操作删除完所有元素之后，`sum_A'`和`sum_B'`就会变成合法的`sum`结果。显然，无论`A`选择的是`a'`还是`b'`中的元素，`sum_A' - sum_B'`都会同样增加该元素的值（`a[1]/2`或`b[1]/2`）。这说明对于某个玩家，选择`A`中的元素和`B`中的元素本质上是相同的。所以最优解就是不断选择`a`和`b`中最大的元素并删除（不管元素是不是自己的）。[^alio]

[^alio]: [MahmoudAlio's Excellent Comment on Editorial](https://codeforces.com/blog/entry/61692?#comment-456714)

---

我感觉这不是一个严格的证明。虽然这确实说明了对于每个玩家当前的最优（贪心）策略是什么，但并没有回答最优子结构的问题。不过我猜这个问题的答案是显然的。

## 代码

```cpp
// 因为上述等价性，实际上把两个数组放在一起排序也是合理的，但看起来好像麻烦一些
// https://codeforces.com/contest/1038/submission/42562135
#include <iostream>
#include <algorithm>
using namespace std;
int a[2][100005];
int main() {
    ios_base::sync_with_stdio(false); cin.tie(nullptr); cout.tie(nullptr);
    int n, m[2];
    long long int sum[2];
    cin >> n;
    for (int i = 0; i < n; i++) cin >> a[0][i];
    for (int i = 0; i < n; i++) cin >> a[1][i];
    sort(a[0], a[0] + n);
    sort(a[1], a[1] + n);
    m[0] = m[1] = n - 1;
    sum[0] = sum[1] = 0;

    while (m[0] >= 0 || m[1] >= 0) {
        for (int i = 0; i <= 1; i++) {
            int j = (i + 1) % 2;
            if (m[i] < 0) {
                if (m[j] < 0) break;
                m[j]--;
            }
            else {
                if (m[j] < 0)
                    sum[i] += a[i][m[i]--];
                else {
                    if (a[i][m[i]] < a[j][m[j]]) m[j]--;
                    else sum[i] += a[i][m[i]--];
                }
            }
        }
    }

    cout << sum[0] - sum[1] << endl;
    return 0;
}
```
