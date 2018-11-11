---
title: Leetcode 940. Distinct Subsequences II（DP）
urlname: leetcode-940-distinct-subsequences-ii
toc: true
date: 2018-11-12 01:05:23
updated: 2018-11-12 01:05:23
tags: [Leetcode, Leetcode Contest, alg:Dynamic Programming]
---

题目来源：[https://leetcode.com/problems/distinct-subsequences-ii/description/](https://leetcode.com/problems/distinct-subsequences-ii/description/)

标记难度：Hard

提交次数：1/3

代码效率：

* 按序列结尾字母进行递推：12ms

## 题意

## 分析

这些做法使我感到，这种计数DP是在非常有限的信息下做到不重不漏地统计的过程，所以保留哪些信息是十分关键的。

### 按序列可能的结尾位置进行递推

这是题解给出的做法，看起来还不错。[^solution]用一个数组`dp[k]`表示`S[0:k]`中不重复的sequence的数量。在不考虑重复的情况下，`dp[k] = 2 * dp[k-1] + 1`。问题是如何从中去掉那些重复的sequence。显然那些被重复的原有的sequence的结尾字母必然是`S[k]`。（想不出来为什么了。。）

[^solution]: [Leetcode Official Solution for 940](https://leetcode.com/problems/distinct-subsequences-ii/solution/)

### 按序列结尾位置进行递推

同样用一个数组`dp[i]`表示以`S[i]`结尾的和之前不重复的sequence的数量。对于每个`dp[i]`，枚举`j = 0 ... i-1`：

* 假如`S[j] != S[i]`，说明直接在以`S[j]`结尾的sequence后面接上`S[i]`不会导致任何重复，所以`S[i] += S[j]`
* 假如`S[j] == S[i]`（。。。又想不出来为什么了）

### 按序列结尾字母进行递推

这是我见到的最优雅的做法。[^lee215]用一个数组`endsWith[26]`来表示（截止到当前位置的字符串）中以每个字母结尾的非空sequence的数量。假设当前这个统计是不重不漏的。考虑加入下一个字母`S[i]`后增加的所有sequence。在不考虑重复的情况下，这个数量是`sum(endsWith) + 1`（原来的所有sequence后面加上这个字母，还有这个字母自己）。如果要发生重复，显然那些被重复的序列的结尾只能是`S[i]`；而那些被重复的以`S[i]`结尾的sequence去掉`S[i]`的结果必定以某种形式存在于`endsWith`数组中，因此它们必定都被重复了一遍：因此重复序列的总数就是`endsWith[S[i] - 'a']`。于是最后的效果是每次更新的时候`endsWith[S[i] - 'a'] = sum(endsWith) + 1`。

[^lee215]: [lee215's Solution for Leetcode 940](https://www.jianshu.com/p/02501f516437)

## 代码

### 按序列结尾字母进行递推

我感到我写得并不好，不如原来用`std::accumulate`的做法优雅。[^lee215]

```cpp
class Solution {
public:
    int distinctSubseqII(string S) {
        int endsWith[26];
        const int MOD = 1e9 + 7;
        memset(endsWith, 0, sizeof(endsWith));
        for (char ch: S) {
            int sum = 0;
            for (int i = 0; i < 26; i++)
                sum = (endsWith[i] + sum) % MOD;
            endsWith[ch - 'a'] = (sum + 1) % MOD;
        }
        int sum = 0;
        for (int i = 0; i < 26; i++)
            sum = (endsWith[i] + sum) % MOD;
        return sum;
    }
};
```