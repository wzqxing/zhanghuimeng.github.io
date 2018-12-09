---
title: Leetcode 953. Verifying an Alien Dictionary（hash），及周赛（114）总结
urlname: leetcode-953-verifying-an-alien-dictionary-hash
toc: true
date: 2018-12-09 16:07:16
updated: 2018-12-09 16:23:00
tags: [Leetcode, Leetcode Contest, alg:Hash Table]
---

题目来源：[https://leetcode.com/problems/verifying-an-alien-dictionary/description/](https://leetcode.com/problems/verifying-an-alien-dictionary/description/)

标记难度：Easy

提交次数：1/1

代码效率：8ms

## 题意

给定一堆字符串和一个字母排列`order`，问在该字母表顺序下，这些字符串是否是排好序的。

## 分析

这次我还是去参加了internal contest。但是这次的状况比较混乱，第三题出现了函数签名搞错的情况，赛后也没有人统计反馈改题什么的。（虽然也许我知道为什么。）这次我第四题没做出来。

---

这道题很水。一种方法是手动比较相邻两个字符串的大小（注意“空”和“有字符”的情况的比较）[^solution]；另一种方法就是直接根据字母排列把字符串映射到正常顺序，然后直接比较[^lee215]。

[^solution]: [LeetCode Official Solution for 953. Verifying an Alien Dictionary](https://leetcode.com/problems/verifying-an-alien-dictionary/solution/)

[^lee215]: [lee215's Solution for LeetCode 953 - \[C++/Python\] Mapping to Normal Order](https://leetcode.com/problems/verifying-an-alien-dictionary/discuss/203185/C++Python-Mapping-to-Normal-Order)

## 代码

```cpp
class Solution {
public:
    bool isAlienSorted(vector<string>& words, string order) {
        int orderMap[26];
        for (int i = 0; i < 26; i++) {
            orderMap[order[i] - 'a'] = i;
        }
        vector<string> mappedWords;
        for (string word: words) {
            string x = word;
            for (int i = 0; i < x.length(); i++)
                x[i] = orderMap[x[i] - 'a'] + 'a';
            mappedWords.push_back(x);
        }
        for (int i = 1; i < mappedWords.size(); i++)
            if (mappedWords[i] < mappedWords[i-1])
                return false;
        return true;
    }
};
```