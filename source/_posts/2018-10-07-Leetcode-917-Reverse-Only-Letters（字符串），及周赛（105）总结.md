---
title: Leetcode 917. Reverse Only Letters（字符串），及周赛（105）总结
urlname: leetcode-917-reverse-only-letters-and-weekly-contest-105
toc: true
date: 2018-10-07 12:21:36
updated: 2018-10-07 12:21:36
tags: [Leetcode, Leetcode Contest, alg:String, alg:Two Pointers]
---

题目来源：[https://leetcode.com/problems/reverse-only-letters/description/](https://leetcode.com/problems/reverse-only-letters/description/)

标记难度：Easy

提交次数：2/2

代码效率：

* 双指针：0ms
* 栈：4ms

## 题意

给定字符串`S`，将`S`中的**字母**逆序排列，其他部分保持不变。

## 分析

这次的排名是270 / 3528。我觉得这次题目的难度排序是1 < 3 < 2 < 4（为什么老有这种第3题比第2题简单的情况出现），而且第4题还比较简单（我感觉到是DP了，虽然不会做）。做出4道题的人一共67个（而且还真有做出来第4题而没做出第2题的人），比上次多一些。

---

本题显然很水。比赛时我的做法是直接用两个指针分别指向两侧的字母。另一种做法需要额外的空间，开一个栈，把字母顺序放进去再弹出。[^solution]

[^solution]: [Leetcode 917 - Reverse Only Letters - Solution](https://leetcode.com/articles/reverse-only-letters/)

## 代码

### 双指针

```cpp
class Solution {
private:
    bool isAlpha(char c) {
        return ('A' <= c && c <= 'Z') || ('a' <= c && c <= 'z');
    }

public:
    string reverseOnlyLetters(string S) {
        int i = 0, j = S.length() - 1;
        while (i < j) {
            while (!isAlpha(S[i]) && i < j) i++;
            while (!isAlpha(S[j]) && i < j) j--;
            if (i >= j) break;
            swap(S[i], S[j]);
            i++, j--;
        }
        return S;
    }
};
```

### 栈

（非常简洁的写法）

```cpp
class Solution {
private:
    bool isAlpha(char c) {
        return ('A' <= c && c <= 'Z') || ('a' <= c && c <= 'z');
    }

public:
    string reverseOnlyLetters(string S) {
        stack<char> s;
        for (char c: S)
            if (isAlpha(c))
                s.push(c);
        for (char& c: S)
            if (isAlpha(c)) {
                c = s.top();
                s.pop();
            }
        return S;
    }
};
```
