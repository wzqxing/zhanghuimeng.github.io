---
title: Leetcode 385. Mini Parser（字符串）
urlname: leetcode-385-mini-parser
toc: true
date: 2018-08-04 17:09:42
updated: 2018-08-04 17:47:00
tags: [Leetcode, alg:Stack, alg:String, alg:Recursion]
---

题目来源：[https://leetcode.com/problems/mini-parser/description/](https://leetcode.com/problems/mini-parser/description/)

标记难度：Medium

提交次数：1/1

代码效率：99.61%

## 题意

把一个整数嵌套列表的字符串解析成一个整数嵌套列表对象。

## 分析

这道题显然没什么算法难度，重点是写法，以及一些需要注意的细节。

* 要求是利用题目中给出的`NestedInteger`类型，而不是自己实现一个这种类型（我刚打开题目的时候被吓到了，以为要我自己用C++实现一个这种东西，那我肯定火速转Python啊。）
* 既然是多层嵌套列表，那么用递归逐层进行处理是很自然的想法。换成栈节省开销也不错。
* 如何在一层列表中判断每一项的界限？我最开始想直接用`,`作为分隔符split之，但很快发现这个想法大错特错，因为每一项自己内部也有嵌套的`,`。所以要加上判断`[`和`]`的个数的条件。
* 直接用`stoi`把`string`转成`int`，也不用自己考虑`-`的问题了，标准库真好用……

## 代码

```
/**
 * // This is the interface that allows for creating nested lists.
 * // You should not implement it, or speculate about its implementation
 * class NestedInteger {
 *   public:
 *     // Constructor initializes an empty nested list.
 *     NestedInteger();
 *
 *     // Constructor initializes a single integer.
 *     NestedInteger(int value);
 *
 *     // Return true if this NestedInteger holds a single integer, rather than a nested list.
 *     bool isInteger() const;
 *
 *     // Return the single integer that this NestedInteger holds, if it holds a single integer
 *     // The result is undefined if this NestedInteger holds a nested list
 *     int getInteger() const;
 *
 *     // Set this NestedInteger to hold a single integer.
 *     void setInteger(int value);
 *
 *     // Set this NestedInteger to hold a nested list and adds a nested integer to it.
 *     void add(const NestedInteger &ni);
 *
 *     // Return the nested list that this NestedInteger holds, if it holds a nested list
 *     // The result is undefined if this NestedInteger holds a single integer
 *     const vector<NestedInteger> &getList() const;
 * };
 */
class Solution {
private:
    NestedInteger decomposeList(string s) {
        // s是一个数字
        // 好消息是，我感觉用了stoi之后，就不需要自己考虑-的问题了
        if (s.length() > 0 && s[0] != '[')
            return NestedInteger(stoi(s));
        // s是一个列表
        s = s.substr(1, s.length() - 2);
        NestedInteger nestedInteger;
        // 并不能简单地用,作为分隔，而是要找到一整个子列表
        int start = 0, leftBrackets = 0;
        for (int i = 0; i < s.length(); i++) {
            if (s[i] == ',') {
                if (leftBrackets != 0)
                    continue;
                nestedInteger.add(decomposeList(s.substr(start, i - start)));
                start = i + 1;
            }
            if (s[i] == '[')
                leftBrackets++;
            if (s[i] == ']')
                leftBrackets--;
        }
        if (start < s.length())
            nestedInteger.add(decomposeList(s.substr(start, s.length() - start)));
        return nestedInteger;
    }

public:
    NestedInteger deserialize(string s) {
        // 刚看到这个的时候我很懵逼
        // 原来是让我们利用这个interface来做题啊……
        return decomposeList(s);
    }
};
```
