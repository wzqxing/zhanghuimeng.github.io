---
title: Leetcode 974. Subarray Sums Divisible by K
urlname: leetcode-974-subarray-sums-divisible-by-k
toc: true
mathjax: true
date: 2019-01-16 02:02:39
updated: 2019-01-16 02:19:39
tags: [Leetcode, Leetcode Contest, alg:Array]
---

题目来源：[https://leetcode.com/problems/subarray-sums-divisible-by-k/description/](https://leetcode.com/problems/subarray-sums-divisible-by-k/description/)

标记难度：Medium

提交次数：1/1

代码效率：60.00%（36ms）

## 题意

给定整数数组`A`，求所有满足和模`K`余0的子数组的数量。

## 分析

比赛的时候，我的做法是：依次扫描数组，求前缀和之后模`N`，从`map`里读出这个余数对应的前缀和数量，加到`ans`里，然后余数对应前缀和数量+1。原理非常简单，子数组和可以用前缀和之差来表示。需要注意的是，0也代表没有前缀时的和。

不过这么做其实并不本质。首先，前缀和模`K`是一个比较小的数，可以用数组来做（而不一定需要`map`）。其次，实际上求每种余数对应前缀和里取两个的结果就可以了！也就是直接用组合数去算。[^sln]

[^sln]: [Leetcode Official Solution for 974](https://leetcode.com/problems/subarray-sums-divisible-by-k/solution/)

显然这么做更加本质。

（以及这让我发现了$\binom{n}{2} = \frac{n(n-1)}{2} = 1 + 2 + \cdots + (n-1)$这个事实，我火星了）

## 代码

```cpp
class Solution {
public:
    int subarraysDivByK(vector<int>& A, int K) {
        map<int, int> cntMap;
        int sum = 0, ans = 0;
        cntMap[0]++;
        for (int i = 0; i < A.size(); i++) {
            sum += A[i];
            int mod = (sum % K + K) % K;
            if (cntMap[mod] > 0) ans += cntMap[mod];
            cntMap[mod]++;
        }
        return ans;
    }
};
```