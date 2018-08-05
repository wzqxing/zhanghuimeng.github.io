---
title: Leetcode 14. Longest Common Prefix（最长公共前缀）
urlname: leetcode-14-longest-common-prefix
toc: true
date: 2018-08-05 15:40:19
updated: 2018-08-05 15:40:19
tags: [Leetcode]
---

题目来源：[https://leetcode.com/problems/longest-common-prefix/description/](https://leetcode.com/problems/longest-common-prefix/description/)

标记难度：Easy

提交次数：1/1

代码效率：97.59%

## 题意

找若干个字符串的最长公共前缀。

## 分析

直接找公共前缀就行。因为不是找公共子串，所以就变得特别水，而且很容易只能找到空串……

这回真的是超级大水题了，都没有太大分析的意义。虽然也可以稍微想一下执行效率什么的吧，但我觉得本质实在差别不大。而且我觉得这道题质量不高。

## 代码

```cpp
class Solution {
public:
    string longestCommonPrefix(vector<string>& strs) {
        // 既然是最长公共前缀，那么应该要求是连续的吧……
        if (strs.size() == 0)
            return "";
        string prefix = strs[0];
        for (int i = 1; i < strs.size(); i++) {
            if (prefix.length() == 0)
                return "";
            int len = 0;
            while (len < min(prefix.length(), strs[i].length())) {
                if (prefix[len] != strs[i][len])
                    break;
                len++;
            }
            prefix = prefix.substr(0, len);
        }
        return prefix;
    }
};
```
