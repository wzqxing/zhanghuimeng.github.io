---
title: Codeforces 1038D. Slime（贪心）
urlname: codeforces-1038d-slime
toc: true
date: 2018-09-08 16:09:35
updated: 2018-09-08 16:56:00
tags: [Codeforces, Codeforces Contest, alg:Greedy]
---

题目来源：[http://codeforces.com/contest/1038/problem/D](http://codeforces.com/contest/1038/problem/D)

提交次数：1/2

## 题意

给定一个长度为`n`的数组，可以将数组内任意两个相邻的数`x`和`y`合并，合并结果是`x - y`。问将数组中的数全部合并为一个数之后，得到的数的最大值。

## 分析

比赛的时候我没有做这个题。后来看了看题解，发现实际上非常简单。

---

我们不妨对数组中的元素`a[i]`做这样的标记：如果在最后的和里，它是被加上的，则令`sign[i] = "+"`；如果是被减去的，则令`sign[i] = "-"`。比如：

```
[1, 2, -1]
sum = 2 - 1 - (-1) = 2
```

则将元素标记为
```
sign = ["-", "+", "-"]
```

一个事实是，当`a.length >= 2`时，`sign`数组中非全`"-"`和非全`"+"`的任何组合都是可以得到的，也就是说，几乎所有加减组合都是可以实现的。对于任一符合上述要求的组合，可以给出一个粗略的构造方法：

1. 从数组中选出一个`"+"`，使得它的左右两边至少各有一个`"-"`；或者这个数在边上，左边或右边是空的，另一边有至少一个`"-"`的数。此时数组看起来是这样的：

```
"+1" "+2" "-3" "-4" "+5" "+6"* "+7" "-8" "-9"
```

2. 对于所有的`"-"`的数，令它“吸收”（也就是减去并合并）所有左右两边的连续的`"+"`的数（当然，不要重复“吸收”同一个数）。次数数组看起来是这样的：

```
("-3" - "+1" - "+2")  ("-4" - "+5") "+6"* ("-8" - "+7") "-9"
```

3. 此时，令之前选出来的`"+"`吸收剩下所有的数，于是我们得到：

```
"+6"* - ("-3" - "+1" - "+2") - ("-4" - "+5") - ("-8" - "+7") - "-9"
= "+6"* - "-3" + "+1" + "+2" - "-4" + "+5" - "-8" + "+7" - "-9"
```

显然我们此时已经构造出了这样的和。[^alio]

[^alio]: 这一构造方法参考了[MahmoudAlio's Comment](https://codeforces.com/blog/entry/61692?#comment-456711)

以及我们可以通过数学归纳法证明为什么全`"+"`和全`"-"`的组合不可能实现。对于任何一个通过加减得到的数，其中必然包含至少一个被减去的`a[i]`和一个被加上的`a[j]`。在第一次加减的时候，我们会得到`a[j] - a[i]`。这之后，无论这个数是减去别的数（`a[j] - a[i]`），还是被别的数减去（`a[i] - a[j]`），得到的结果中仍然至少包含一个被减去的数和一个被加上的数。

此时就很容易得出最终的算法了。令`sum`表示数组中所有元素绝对值的和，则：

* 如果数组长度为`1`，则`ans = a[1]`（显然此时不可能进行任何合并和组合。我在这里WA了一次）
* 如果数组元素全为正，则`ans = sum - 2 * minValue`（`minValue`在最后的结果中是减去的，其他数都是加上的）
* 如果数组元素全为负，则`ans = sum + 2 * maxValue`（`maxValue`在最后的结果中是加上的，其他数都是减去的）
* 如果数组元素有正有负，则`ans = sum`（令正数被加上，负数被减去即可）

## 代码

```cpp
#include <iostream>
#include <cmath>
using namespace std;
int n, a[500005];
int main() {
    ios_base::sync_with_stdio(false); cin.tie(nullptr); cout.tie(nullptr);
    cin >> n;
    cin >> a[0];

    // 然而这是一个特判
    if (n == 1) {
        cout << a[0] << endl;
        return 0;
    }

    long long int sum = abs(a[0]);
    int maxn = a[0], minn = a[0], hasPos = a[0] > 0, hasNeg = a[0] < 0;
    for (int i = 1; i < n; i++) {
        cin >> a[i];
        maxn = max(maxn, a[i]);
        minn = min(minn, a[i]);
        hasPos = hasPos ? hasPos : a[i] > 0;
        hasNeg = hasNeg ? hasNeg : a[i] < 0;
        sum += abs(a[i]);
    }
    if (hasPos && !hasNeg) sum -= 2 * minn;
    else if (!hasPos && hasNeg) sum += 2 * maxn;
    cout << sum << endl;
    return 0;
}
```
