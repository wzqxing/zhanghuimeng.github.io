---
title: Leetcode 696. Count Binary Substrings（观察）
urlname: leetcode-696-count-binary-substrings
toc: true
date: 2018-07-31 22:08:02
updated: 2018-07-31 22:39:00
mathjax: true
tags: [Leetcode, alg:String]
---

题目来源：[https://leetcode.com/problems/count-binary-substrings/description/](https://leetcode.com/problems/count-binary-substrings/description/)

标记难度：Easy

提交次数：1/1

代码效率：5.22%

## 题意

给定一个01串，问其中形如`0011`或`1100`（所有0连在一起，所有1也连在一起，数目相等）的子串共有多少个，不去重。

## 分析

这就是一个大水题，把所有连续的0或1子串的长度统计出来，就可以直接做了。

P.S. 我刚才本来想用正则表达式来表达$0^n 1^n$或$1^n 0^n$这个意思的，结果突然想起来，正则表达式的表示能力相当于有限状态自动机，而有限状态自动机表示不了$0^n 1^n$……（如果我没记错的话）

## 代码

```
class Solution {
public:
    int countBinarySubstrings(string s) {
        // 需要找出01数量相同，且所有0和所有1都连在一起（也就是形如0011或1100的串）
        // 只需把字符串分成连续的0或1，然后统计即可

        // 这个写法效率低，但是很直观
        vector<int> sub;
        char cur;
        int len = 0;
        for (int i = 0; i < s.length(); i++) {
            if (len == 0) {
                cur = s[i];
                len++;
            }
            else if (cur == s[i])
                len++;
            else {
                sub.push_back(len);
                cur = s[i];
                len = 1;
            }
        }
        if (len != 0)
            sub.push_back(len);

        if (sub.size() < 1)
            return 0;
        int ans = 0;
        for (int i = 0; i < sub.size() - 1; i++)
            ans += min(sub[i], sub[i+1]);
        return ans;
    }
};
```
