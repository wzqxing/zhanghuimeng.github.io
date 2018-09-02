---
title: Leetcode 890. Find and Replace Pattern（map）
urlname: leetcode-890-find-and-replace-pattern
toc: true
date: 2018-08-19 17:36:49
updated: 2018-08-19 17:45:00
tags: [Leetcode, Leetcode Contest, alg:Hash Table]
---

题目来源：[https://leetcode.com/problems/find-and-replace-pattern/description/](https://leetcode.com/problems/find-and-replace-pattern/description/)

标记难度：Medium

提交次数：1/1

代码效率：4ms

## 题意

给定一系列`word`和一个`pattern`，判断对于每一个`word`，是否存在一种字母双射（实际上就是把字母映射到它的一个排列上），能把它映射为`pattern`。

## 分析

这是周赛的第三题，我在0:45时才提交，一共做了25分钟；但是回想起来，觉得这真是一道相当水的题，我怎么花了那么长时间的？

具体做法没有任何难度，直接开一个`map`，比较`word`和`pattern`的每一个字母的同时记录映射，并判断映射有无矛盾。至于判断是否是双射，既可以再开一个`map`，也可以在`map`构造完成之后用`set`来判断`value`中有无重复的字母。

## 代码

```cpp
class Solution {
public:
    vector<string> findAndReplacePattern(vector<string>& words, string pattern) {
        vector<string> permuteOk;
        for (string word: words) {
            // try to map word to pattern
            map<char, char> patternMap;
            if (word.length() != pattern.length())
                continue;
            bool isOk = true;
            for (int i = 0; i < word.length(); i++) {
                char source = word[i], target = pattern[i];
                if (patternMap.count(source) > 0) {
                    if (patternMap[source] != target) {
                        isOk = false;
                        break;
                    }
                }
                else
                    patternMap[source] = target;
            }
            if (!isOk)
                continue;

            set<char> inverseMap;
            for (auto const& i: patternMap)
                if (inverseMap.count(i.second) > 0) {
                    isOk = false;
                    break;
                }
                else
                    inverseMap.insert(i.second);
            if (!isOk)
                continue;

            permuteOk.push_back(word);
        }

        return permuteOk;
    }
};
```
