---
title: Leetcode 844. Backspace String Compare（栈）
urlname: leetcode-844-backspace-string-compare
toc: true
date: 2018-08-02 00:49:44
updated: 2018-08-02 01:01:00
tags: [Leetcode, alg:Stack, alg:Two Pointers]
---

题目来源：[https://leetcode.com/problems/backspace-string-compare/description/](https://leetcode.com/problems/backspace-string-compare/description/)

标记难度：Easy

提交次数：2/3

代码效率：

* 栈版本：12.03%
* 指针版本：100.00%

## 题意

给定两个由小写字母组成的字符串，其中包含一些向前删除字符（`#`）；问完成删除之后两个字符串是否相等。

进阶：

* 能否在`O(N)`时间复杂度和`O(1)`空间复杂度内解决？（虽然我觉得应该是“额外”空间，毕竟输入大小已经是`O(N)`了）

## 分析

不考虑空间的话，直接用两个栈模拟一下删除的过程。

考虑额外空间的话，直接把字符串自己当成栈，用两个指针分别指向栈顶就可以了。

## 代码

### O(N)额外空间复杂度（栈）

```
class Solution {
public:
    bool backspaceCompare(string S, string T) {
        stack<char> sstack, tstack;
        for (char c: S) {
            if (c == '#') {
                if (!sstack.empty())  // 不小心把#也插入栈中了
                    sstack.pop();
            }
            else
                sstack.push(c);
        }
        string ms;
        while (!sstack.empty()) {
            ms = sstack.top() + ms;
            sstack.pop();
        }
        for (char c: T) {
            if (c == '#') {
                if (!tstack.empty())
                    tstack.pop();
            }
            else
                tstack.push(c);
        }
        string mt;
        while (!tstack.empty()) {
            mt = tstack.top() + mt;
            tstack.pop();
        }

        cout << ms << ' ' << mt << endl;
        return ms == mt;
    }
};
```

### O(1)额外空间复杂度（指针）

```
class Solution {
public:
    bool backspaceCompare(string S, string T) {
        int topS = 0, topT = 0;
        for (int i = 0; i < S.length(); i++) {
            char c = S[i];
            if (c == '#') {
                if (topS > 0)
                    topS--;
            }
            else
                S[topS++] = c;
        }
        for (int i = 0; i < T.length(); i++) {
            char c = T[i];
            if (c == '#') {
                if (topT > 0)
                    topT--;
            }
            else
                T[topT++] = c;
        }

        if (topS != topT)
            return false;
        return S.substr(0, topS) == T.substr(0, topT);
    }
};
```
