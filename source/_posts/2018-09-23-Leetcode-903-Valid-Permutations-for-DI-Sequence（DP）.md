---
title: Leetcode 903. Valid Permutations for DI Sequence（DP）
urlname: leetcode-903-valid-permutations-for-di-sequence
toc: true
date: 2018-09-23 11:41:02
updated: 2018-09-23 19:19:00
tags: [Leetcode, Leetcode Contest, alg:Dynamic Programming]
---

题目来源：[https://leetcode.com/problems/valid-permutations-for-di-sequence/description/](https://leetcode.com/problems/valid-permutations-for-di-sequence/description/)

标记难度：Hard

提交次数：2/3

代码效率：

* O(N^2) DP：79.78%
* O(N^3) DP：14.07%

## 题意

给定`n`，求所有数字满足一定上升和下降规律的`n`的排列。

## 分析

比赛的时候看到这道题，我心里是十分懵逼的，觉得应当用搜索的方法去做，但又感觉复杂度太高，即使剪枝也很离谱。显然，我根本没有怎么接触过这个类型的题。所以比赛之后我仍然很懵逼。

---

事实上这是一道动态规划题。但是递推的方式并不好想。令`dp[i][j]`表示长度为`i`且结尾为`j`的符合DI序列的排列数量。（在这里，长度为`i`也就意味着，是从1到`i`的数字的排列。）下一个问题是如何进行递推。如果我们之前已经得到了一个`1~i`的排列，如何把`j`合理地添加进去，使得这个排列变成`1~i+1`的排列，仍然满足原有的DI序列，且结尾是`j`？

这个想法无疑十分巧妙：把比`j`更大的数都加上1，然后把j附在排列的最后。这样就可以得到一个仍然合法且合理的排列了。[^visual]

[^visual]: [\[Visualization\] Key to the DP solution: imagine cutting a piece of paper and separating the halves](https://leetcode.com/problems/valid-permutations-for-di-sequence/discuss/169126/Visualization-Key-to-the-DP-solution:-imagine-cutting-a-piece-of-paper-and-separating-the-halves)

![图示：为何对部分数值的+1操作不会破坏DI性质](visualization.gif)

如果DI序列当前的值为`D`，则`dp[i][j] = dp[i-1][j] + ... + dp[i-1][N]`；如果当前的值为`I`，则`dp[i][j] = dp[i-1][1] + ... + dp[i-1][j-1]`。

如果不进行优化，则这是一个`O(N^3)`的算法。但是实际上可以非常迅速地用前缀和数组的方法加以改进，这样就变成了一个`O(N^2)`的算法。[^exp]

[^exp]: [Share my O(N^3) => O(N^2) C++ DP solution. Including the thoughts of improvement.](https://leetcode.com/problems/valid-permutations-for-di-sequence/discuss/168289/Share-my-O%28N3%29-greater-O%28N2%29-C++-DP-solution.-Including-the-thoughts-of-improvement.)

除此之外，还有一种比较好的思考方法：先构思一种自顶向下的回溯法，再把这种方法逐渐转换为DP。不过具体内容我不是很想看了。[^wangzi]

[^wangzi]: [Top-down with Memo -> Bottom-up DP -> N^3 DP -> N^2 DP -> O(N) space](https://leetcode.com/problems/valid-permutations-for-di-sequence/discuss/168612/Top-down-with-Memo-greater-Bottom-up-DP-greater-N3-DP-greater-N2-DP-greater-O%28N%29-space)

---

最后一个有趣的问题是：是否对于任何DI序列，都存在合法的排列？我觉得可以用和动态规划类似的方法进行推导。令函数`f(str)`表示DI序列`str`所对应的所有可能末尾元素的集合。

* 当`str.length = 1`时，显然`f("D") = {1}, f("I") = {2}`，集合均非空
* 当`str.length = N, N >= 2`时，对于某一`str`，由归纳假设，有`f(str[0:-1])`非空。
  * 当`str[-1] = 'I'`时，从`f(str[0:-1])`中任取元素，直接将`N`附在该元素对应的排列后，即可构造出新的合法排列
  * 当`str[-1] = 'D'`时，从`f(str[0:-1])`中任取元素，记该元素为`j`，将对应排列中`>= j`的元素均增加1，并把`j`附在排列最后，即可构造出新的合法排列

所以对于任意长度的任意DI序列，`f(str)`都不为空，得证。

## 代码

### O(N^3) DP

```cpp
class Solution {
public:
    int numPermsDISequence(string S) {
        int N = S.length() + 1;
        long long MOD = 1000000007;
        long long int dp[N+1][N+1];  // length i, ends with j
        memset(dp, 0, sizeof(dp));
        dp[1][1] = 1;
        long long int ans = 0;
        for (int i = 2; i <= N; i++) {
            for (int j = 1; j <= i; j++) {
                if (S[i-2] == 'D') {
                    for (int k = j; k <= N; k++)
                        dp[i][j] = (dp[i][j] + dp[i-1][k]) % MOD;
                }
                else {
                    for (int k = 1; k < j; k++)
                        dp[i][j] = (dp[i][j] + dp[i-1][k]) % MOD;
                }
                if (i == N)
                    ans = (ans + dp[i][j]) % MOD;
            }
        }
        return ans;
    }
};
```

### O(N^2) DP

```cpp
class Solution {
public:
    int numPermsDISequence(string S) {
        int N = S.length() + 1;
        long long MOD = 1000000007;
        long long int dp[N+1][N+1];  // length i, ends with j
        long long int g[N+1][N+1];
        memset(dp, 0, sizeof(dp));
        memset(g, 0, sizeof(g));
        dp[1][1] = 1;
        for (int i = 1; i <= N; i++)
            g[1][i] = g[1][i-1] + dp[1][i];
        long long int ans = 0;
        for (int i = 2; i <= N; i++) {
            for (int j = 1; j <= i; j++) {
                if (S[i-2] == 'D') {
                    // [j, N]
                    dp[i][j] = (g[i-1][N] - g[i-1][j-1] + MOD) % MOD;
                    // beware of mod subtraction...
                }
                else {
                    // [1, j-1]
                    dp[i][j] = g[i-1][j-1] % MOD;
                }
                if (i == N)
                    ans = (ans + dp[i][j]) % MOD;
            }
            for (int j = 1; j <= N; j++)
                g[i][j] = (g[i][j-1] + dp[i][j]) % MOD;
        }
        return ans;
    }
};
```
