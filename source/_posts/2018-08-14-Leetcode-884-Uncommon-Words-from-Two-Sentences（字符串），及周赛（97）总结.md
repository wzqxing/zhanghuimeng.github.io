---
title: Leetcode 884. Uncommon Words from Two Sentences（字符串），及周赛（97）总结
urlname: leetcode-884-uncommon-words-from-two-sentences-and-weekly-contest-97
toc: true
date: 2018-08-14 22:31:52
updated: 2018-08-15 02:00:00
tags: [Leetcode, Leetcode Contest, alg:Hash Table, alg:String]
---

题目来源：[https://leetcode.com/problems/uncommon-words-from-two-sentences/description/](https://leetcode.com/problems/uncommon-words-from-two-sentences/description/)

标记难度：Easy

提交次数：1/1

代码效率：100.00%

## 题意

找出两个字符串中所有不重叠的词。不要求顺序。

## 分析

第一次参加Leetcode周赛，得到了233 / 3760的“好”成绩，这个数看起来简直特别吉利……

![233](rank.png)

此题完成于0:09:46，总的来说是一道超级水题，但我做的时候又遇到了那个经典的问题……嗯，没错，C++ std::string不提供split方法。于是我又一次紧张地去[stackoverflow](https://stackoverflow.com/questions/236129/the-most-elegant-way-to-iterate-the-words-of-a-string)上查了一通。这之后我得出了一个这样的结论：分隔符为` `时的情况是平凡的，可以直接用`std::stringstream`的方法解决；至于其他情况，还是自己手动瞎搞吧。

知乎上有一些[相关的讨论](https://www.zhihu.com/question/36642771)，但我还是对此很不满，每次有字符串处理需求的时候，就想丢下C++去用Python算了。

## 代码

```cpp
class Solution {
private:
    vector<string> split(string str) {
        // https://stackoverflow.com/questions/236129/the-most-elegant-way-to-iterate-the-words-of-a-string
        std::string buf;                 // Have a buffer string
        std::stringstream ss(str);       // Insert the string into a stream

        std::vector<std::string> tokens; // Create vector to hold our words

        while (ss >> buf)
            tokens.push_back(buf);

        return tokens;
    }

public:
    vector<string> uncommonFromSentences(string A, string B) {
        map<string, int> freq;
        vector<string> v1 = split(A);
        vector<string> v2 = split(B);
        for (string s: v1)
            freq[s]++;
        for (string s: v2)
            freq[s]++;

        vector<string> ans;
        for (string s: v1)
            if (freq[s] == 1)
                ans.push_back(s);
        for (string s: v2)
            if (freq[s] == 1)
                ans.push_back(s);

        return ans;
    }
};
```
