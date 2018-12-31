---
title: Leetcode 966. Vowel Spellchecker（字符串）
urlname: leetcode-966-vowel-spellchecker
toc: true
date: 2018-12-31 14:21:11
updated: 2018-12-31 14:37:00
tags: [Leetcode, Leetcode Contest, alg:String, alg:Hash Table]
---

题目来源：[https://leetcode.com/problems/univalued-binary-tree/description/](https://leetcode.com/problems/univalued-binary-tree/description/)

标记难度：Medium

提交次数：1/2

代码效率：112ms（40.00%）

## 题意

给定一个`wordlist`，其中包含了若干个单词（只包含英文字母的字符串），给定一系列`query`，要求从`wordlist`中找出匹配单词：

* 如果`wordlist`中包含单词与该`query`相同，则直接返回`query`
* 如果在不考虑大小写的情况下，`wordlist`中包含单词与该`query`相同（例：`"AbC"`和`"aBc"`是相同的），则返回第一个匹配单词
* 如果在不考虑大小写和将所有元音字母认为是同一个字母的情况下，`wordlist`中包含单词与该`query`相同（例：`"Aab"`与`"eIb"`是相同的），则返回第一个匹配单词

## 分析

首先一个事实是，`O(N*M)`（`N = wordlist.size()`，`M = queries.size()`）的算法是过不了的，因为`N, M <= 5000`。那么显然就要用到哈希表了。

* 首先用一个`set<string> exact`存储原来的单词
* 然后用一个`map<string, int> lower`存储每个单词的lowercase版本到它的索引，如果一个lowercase版本对应多个单词，则只存储第一个出现的单词的索引
* 然后用一个`map<string, int> vowel`存储每个单词的lowercase且把所有元音都替换成`"a"`的版本到它的索引，如果一个替换后的版本对应多个单词，则只存储第一个出现的单词的索引
* 最后，对于每一个`query`，分别在这三个结构里查找，如果查到则立刻返回，否则最后返回`""`

总的来说也不是很难。我WA了一次，主要是因为没看清，替换元音字母的同时也要替换大小写……

## 代码

```cpp
class Solution {
private:
    string toLower(string x) {
        string l;
        for (char ch: x)
            l += 'A' <= ch && ch <= 'Z' ? ch - 'A' + 'a' : ch;
        return l;
    }
    
    string toVowel(string x) {
        string v;
        for (char ch: x)
            v += ch == 'e' || ch == 'i' || ch == 'o' || ch == 'u' ? 'a' : ch;
        return v;
    }
    
public:
    vector<string> spellchecker(vector<string>& wordlist, vector<string>& queries) {
        set<string> exact;
        map<string, int> lower;
        map<string, int> vowel;
        for (int i = 0; i < wordlist.size(); i++) {
            exact.insert(wordlist[i]);
            string l = toLower(wordlist[i]);
            if (lower.find(l) == lower.end())
                lower[l] = i;
            string v = toVowel(l);
            if (vowel.find(v) == vowel.end())
                vowel[v] = i;
        }
        
        vector<string> ans;
        for (string x: queries) {
            string l = toLower(x);
            string v = toVowel(l);
            if (exact.find(x) != exact.end())
                ans.push_back(x);
            else if (lower.find(l) != lower.end())
                ans.push_back(wordlist[lower[l]]);
            else if (vowel.find(v) != vowel.end())
                ans.push_back(wordlist[vowel[v]]);
            else
                ans.push_back("");
        }
        return ans;
    }
};
```