---
title: Leetcode 942. DI String Match（数学）
urlname: leetcode-942-di-string-match
toc: true
date: 2018-11-18 23:14:01
updated: 2018-11-19 01:34:00
tags: [Leetcode, Leetcode Contest, alg:Math, alg:Dynamic Programming]
---

题目来源：[https://leetcode.com/problems/delete-columns-to-make-sorted/description/](https://leetcode.com/problems/delete-columns-to-make-sorted/description/)

标记难度：Easy

提交次数：3/4

代码效率：

* O(n^2)：336ms
* outside-in：32ms
* inside-out：32ms

## 题意

给定一个DI sequence，求任意一个满足该sequence的0~N-1的排列。

## 分析

我感觉这道题才应该记Medium……

### O(N^2)

这道题一看就让人想起之前的[Leetcode 903. Valid Permutations for DI Sequence](/post/leetcode-903-valid-permutations-for-di-sequence)那道题。那是一道动态规划题，求满足某个DI sequence的排列的总数，核心递推方法是这样的：

* 从1~i的排列得到1~i+1的排列，仍满足DI sequence，且结尾是j
* 将排列中比j大的数都+1，然后把j放在排列的最后，就可以得到一个仍然合法的排列了
* 在递推过程中通过之前的结尾可以保证满足当前的D/I

然后我就literally地应用了这一方法：

* 令序列初始为`seq = [0]`，且保证它始终满足DI性质
* 如果当前DI sequence的值为D，则令`seq += 1`，`seq.append(0)`（保证仍然是0~i的排列，且最后一位是0，必然满足D性质）
* 否则，`seq.append(i)`（因为`i`必然大于之前的所有数）

这个方法没什么问题，不过它是`O(N^2)`的，太浪费了。

### outside-in

这种想法[^lee215]很容易理解，而且它直观地说明了“为什么对于任意DI序列都存在合法的排列”。记录当前还没被用过的最小值（初始为0）和最大值（初始为N-1）；然后，如果当前DI sequence的值为D，则在序列中插入当前最大值（因为这样下一个就必然会比它小）；否则插入当前最小值（因为这样下一个就必然会比它大）。最后这两个值一定是相等的（因为DI sequence的长度为N-1，而每一个sequence中的D/I值都会使当前还没被用过的最小值和最大值之间的差减小1）。

[^lee215]: [lee215's Solution for Leetcode 942](https://leetcode.com/problems/di-string-match/discuss/194904/C++JavaPython-Straight-Forward)

### inside-out

这种想法[^lee215]和上一种是对称的，区别是，我们记录的是当前已经被用过的最大的值，和当前已经被用过的最小的值。因此我们需要一个初值，也就是`S`中D的数量。

## 代码

### O(N^2)

```cpp
class Solution {
public:
    vector<int> diStringMatch(string S) {
        vector<int> ans = {0};
        int n = S.length();
        for (int i = 0; i < n; i++) {
            if (S[i] == 'I') ans.push_back(i + 1);
            else {
                for (int j = 0; j < ans.size(); j++)
                    ans[j]++;
                ans.push_back(0);
            }
        }
        return ans;
    }
};
```

### outside-in

```cpp
class Solution {
public:
    vector<int> diStringMatch(string S) {
        int N = S.length() + 1;
        int minn = 0, maxn = N - 1;
        vector<int> ans;
        for (int i = 0; i < N - 1; i++) {
            if (S[i] == 'D') ans.push_back(maxn--);
            else ans.push_back(minn++);
        }
        ans.push_back(minn);
        return ans;
    }
};
```

### inside-out

```cpp
class Solution {
public:
    vector<int> diStringMatch(string S) {
        int N = S.length() + 1;
        int minn = 0, maxn;
        for (char ch: S) minn += ch == 'D' ? 1 : 0;
        maxn = minn;
        vector<int> ans = {minn};
        for (int i = 0; i < N - 1; i++) {
            if (S[i] == 'D') ans.push_back(--minn);
            else ans.push_back(++maxn);
        }
        return ans;
    }
};
```