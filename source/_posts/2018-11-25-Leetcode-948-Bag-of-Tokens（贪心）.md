---
title: Leetcode 948. Bag of Tokens（贪心）
urlname: leetcode-948-bag-of-tokens
toc: true
mathjax: true
date: 2018-11-25 15:18:56
updated: 2018-11-25 15:18:56
tags: [Leetcode, Leetcode Contest, alg:Greedy, alg:Two Pointers]
---

题目来源：[https://leetcode.com/problems/validate-stack-sequences/description/](https://leetcode.com/problems/validate-stack-sequences/description/)

标记难度：Medium

提交次数：1/1

代码效率：4ms

## 题意

你有两个属性：power和points（初始为0），并且有一些token，每个token自己有一个值（`token[i]`）。你可以对一个token做如下两种操作之一（或者可以什么都不做）：

* face up：如果你的`power >= token[i]`，则可以令`power -= token[i]`，且`points++`
* face down：如果你的`points > 0`，则可以令`power += token[i]`，且`points--`

问你可以用这些token达到的最大points数量是多少。

## 分析

我当时想了好久，怎么平衡两类token的数量（并得到一个合法的操作序列）之类的。但总的来说肯定是这样的：为了最大化points，肯定face up的token的**数量**越多越好，face down的token的**数量**越少越好。那么显然需要face up的token的值尽可能小，face down的token的值尽可能大。虽然不知道贪心是不是对的，但不妨先写一个看看……嗯，过了。

贪心的思路就很直接，先把token排序，然后小token从小到大尝试置为face up，直到power不够位置；然后尝试把一个最大的token置为face down，再接着试剩下的小token，以此类推……

每次Leetcode出这样的题，题解里也不给个形式证明，就很烦。（事实上我感觉贪心几乎是所有算法题里唯一需要形式化证明算法正确性的。）我觉得这可以用stay ahead[^stayhead]方法：记算法的每一步操作为$A = \{a_1, a_2, ..., a_k\}$，最优解的每一步操作为$O = \{o_1, o_2, ..., o_k\}$；令$f(\cdot) = (points, power)$为量度函数，规定

{% raw %}
$$(points_A, power_A) > (points_B, power_B) \iff 
\begin{cases}
points_A > points_B \\
points_A = points_B, \quad power_A > power_B
\end{cases}
$$
{% endraw %}

（就是规定了一个`std::pair`出来……）

下面用数学归纳法证明$f(a_1, ..., a_r) \geq f(o_1, ..., o_r)$。对于第一步操作，由于贪心算法会尝试选择最小的token进行face down，如果能成功，则point数量会+1，且power数量减小最少，必然有$f(a_1) > f(o_1)$；否则两种算法必然根本都无法行动。之后的操作也是同理。

[^stayhead]: [CS 482 2003 - Proof Techniques: Greedy Stays Ahead](http://www.cs.cornell.edu/courses/cs482/2003su/handouts/greedy_ahead.pdf)

## 代码

代码不重要，贪心比较重要。

```cpp
class Solution {
public:
    int bagOfTokensScore(vector<int>& tokens, int P) {

        int maxPoints = 0;
        
        int n = tokens.size();
        sort(tokens.begin(), tokens.end());
        int i = 0, j = n - 1;
        int power = P, points = 0;
        while (i <= j) {
            bool moved = false;
            // small face up
            while (tokens[i] <= power && i <= j) {
                power -= tokens[i];
                points++;
                i++;
                moved = true;
            }
            maxPoints = max(maxPoints, points);
            if (i > j) break;
            // big face down
            if (points > 0) {
                power += tokens[j];
                points--;
                j--;
                moved = true;
            }
            if (!moved) break;
        }
        return maxPoints;
    }
};
```