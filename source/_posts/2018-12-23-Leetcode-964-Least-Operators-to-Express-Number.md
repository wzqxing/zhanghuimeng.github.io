---
title: Leetcode 964. Least Operators to Express Number
urlname: leetcode-964-least-operators-to-express-number
toc: true
mathjax: true
date: 2018-12-23 19:33:23
updated: 2018-12-27 21:24:00
tags: [Leetcode, Leetcode Contest, alg:Math, alg:Dynamic Programming]
---

题目来源：[https://leetcode.com/problems/least-operators-to-express-number/](https://leetcode.com/problems/least-operators-to-express-number/)

标记难度：Hard

提交次数：1/3

代码效率：93.04%

## 题意

给定正整数`x`，要求用`x`和运算符`+-*/`组成表达式，表达式值为`target`，问最少使用多少个运算符。

其中：

1. `/`返回的是有理数
2. 没有括号
3. 优先级和平时相同
4. 不允许使用单目运算符`-`

## 分析

比赛的时候看到这题，我感到非常懵逼。随便想了一下，大概只想到，这道题大概是要求`a[0] + a[1]*x + a[2]*x^2 + ... = target`。如果先不管运算符的数量，第一个问题是怎么把`target`拼出来。考虑到可以使用`m/m=1`，`target`肯定可以拼出来，问题是怎么拼比较好。我没有多认真想，比赛就完了。

事后发现似乎需要用到类似于记忆化搜索的方法。

---

这道题在本质上非常类似于进制表示：

$$target = a_0 + a_1 x + a_2 x^2 + \cdots$$

但是很大的一个区别在于，它对系数（理论上）没有限制，也因此对$x$的最大幂次没有限制（因为系数可以是负的）。而且我们的目标是最小化代价。容易推断出，生成一个$x$的幂次的代价为（包含前面的+/-运算符）：

{% raw %}
$$
cost(x^e) =
\begin{cases}
2 & e = 0\\
e & e \ge 1
\end{cases}
$$
{% endraw %}

显然，如果要生成一个$a_e x^e$，最优的选择是都采用+或者都采用-运算符（因为同时用显然是浪费）。

因此整体的代价可以表示为

$$cost = 2 \cdot abs(a_0) + 2 \cdot abs(a_1) + 3\cdot abs(a_2) + \cdots$$

不妨首先把$target$表示成普通的$x$进制的形式：

$$target = b_0 + b_1 x + b_2 x^2 + \cdots + b_n x^n, \, 0 \le b_i < x$$

可以认为$a_i$是在$b_i$的基础上得到的，例如：

$$target = (b_0 - x + x^2) + (b_1+1)x + (b_2-1) x^2 + \cdots$$

这个式子看起来好像借位，不过比实际中的借位要自由，因为现实的进制中不会出现$b_0$这一位向$b_2$借位的情况。

此时有$a_0 = b_0 - x + x^2, \, a_1 = b_1 + 1, \, a_2 = b_2 - 1$。显然此时整体的代价也变化了：

$$cost = 2b_0 + 2b_1 + 3b_2 + \cdots$$

$$cost' = 2\cdot |b_0 - x + x^2| + 2 \cdot |b_1 + 1| + 3 \cdot |b_2 - 1| + \cdots$$

显然我们希望整体代价最小。考虑到$x >= 2$，显然可以得出这样一个结论：从比前一位更往前的位借位是不值得的，而且一定是借一个负位。原因很简单：假定$b_i$从$b_j$借了$k$位（$j \geq i+2$），则这两位的cost之和从$|b_i| (i+1) + |b_j|(j + 1)$变成了$|b_i + k x^{j-i}| (i+1) + |b_j-k|(j + 1)$。通过各种分类讨论可以发现，这个cost不可能减小。（懒得去讨论了……）

所以可以用递推的方法来考虑这个问题：令

{% raw %}
$$
\begin{aligned}
f[i][0] =& \min{cost(a_0 + a_1 x + a_2 x^2 + \cdots + a_i x^i)} \\
f[i][1] =& \min{cost(a_0 + a_1 x + a_2 x^2 + \cdots + (a_i - x) x^i)}
\end{aligned}
$$
{% endraw %}

也就是说$f[i][1]$是借了一个负位之后最小可能的表示的代价。则

{% raw %}
$$
\begin{aligned}
f[i+1][0] =& \min{cost(a_0 + a_1 x + a_2 x^2 + \cdots + a_i x^i + a_{i+1} x^{i+1})} \\
=& \min cost(a_0 + a_1 x + a_2 x^2 + \cdots + a_i x^i) + cost(a_{i+1}x^{i+1}), \\
& cost(a_0 + a_1 x + a_2 x^2 + \cdots + (a_i - x) x^i) + cost((a_{i+1} + 1)x^{i+1}) \\
=& \min{f[i][0] + a_{i+1}(i+1), \, f[i][1] + (a_{i+1} + 1)(i + 1)}
\end{aligned}
$$
{% endraw %}

同理：

{% raw %}
$$f[i+1][1] = \min{f[i][0] + |a[i+1] - x|(i+1), \, f[i][1] + |a[i+1] - x + 1| (i + 1)}$$
{% endraw %}

然后就可以递推了，时间复杂度为`O(log(N))`。

从借位的想法translate到递推还是不太容易啊……

## 代码

我也不知道我怎么会开大小到1000的数组的……

以及，考虑到有借位的可能，需要递推到`f[n]`，而不是`f[n-1]`。

```cpp
class Solution {
public:
    int leastOpsExpressTarget(int x, int target) {
        int a[1000], n = 0;
        memset(a, 0, sizeof(a));
        int t = target;
        while (t > 0) {
            a[n++] = t % x;
            t /= x;
        }
        int f[1000][2];
        f[0][0] = 2 * a[0];
        f[0][1] = 2 * abs(a[0] - x);
        for (int i = 1; i <= n; i++) {
            f[i][0] = min(f[i-1][0] + a[i] * i, f[i-1][1] + (a[i] + 1) * i);
            f[i][1] = min(f[i-1][0] + abs(a[i] - x) * i, f[i-1][1] + abs(a[i] - x + 1) * i);
        }
        return f[n][0] - 1;
    }
};
```