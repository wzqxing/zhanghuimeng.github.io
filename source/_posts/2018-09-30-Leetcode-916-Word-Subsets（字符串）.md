---
title: Leetcode 916. Word Subsets（字符串）
urlname: leetcode-916-word-subsets
toc: true
mathjax: true
date: 2018-09-30 21:18:40
updated: 2018-09-30 21:30:00
tags: [Leetcode, Leetcode Contest, alg:String, alg:Hash Table]
---

题目来源：[https://leetcode.com/problems/word-subsets/description/](https://leetcode.com/problems/word-subsets/description/)

标记难度：Medium

提交次数：1/1

代码效率：176ms

## 题意

给定字符串数组`A`和`B`，如果字符串`b`中每个字符的出现次数都<=该字符在`a`中的出现次数，则称`b`是`a`的子集。如果对于`B`中的每个字符串`b`，`b`都是`a`的子集，则称`a`为“universal”。问`A`中有多少个字符串是“universal”的。

## 分析

这道题也非常简单。如果`a`是“universal”的，说明

$$\forall b \in B, \forall char, count_{char}(a) \geq count_{char}(b)$$

即

$$\forall char, \forall b \in B, count_{char}(a) \geq count_{char}(b)$$

$$\forall char, count_{char}(a) \geq \max_{\forall b \in B}{count_{char}(b)}$$

只要`a`满足以上条件就可以了。[^solution]

[^solution]: [Leetcode 916 Solution](https://leetcode.com/problems/word-subsets/solution/)

## 代码

```cpp
class Solution {
public:
    vector<string> wordSubsets(vector<string>& A, vector<string>& B) {
        int allb[26], b[26], a[26];
        memset(allb, 0, sizeof(allb));
        for (string bs: B) {
            memset(b, 0, sizeof(b));
            for (char ch: bs)
                b[ch - 'a']++;
            for (int i = 0; i < 26; i++)
                allb[i] = max(allb[i], b[i]);
        }

        vector<string> ans;
        for (string as: A) {
            memset(a, 0, sizeof(a));
            for (char ch: as)
                a[ch - 'a']++;
            bool ok = true;
            for (int i = 0; i < 26; i++)
                if (a[i] < allb[i]) {
                    ok = false;
                    break;
                }
            if (ok)
                ans.push_back(as);
        }
        return ans;
    }
};
```
