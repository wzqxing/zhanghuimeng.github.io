---
title: Leetcode 771. Jewels and Stones（集合）
urlname: leetcode-771-jewels-and-stones
toc: true
date: 2018-08-04 16:44:33
updated: 2018-08-04 16:54:00
tags: [Leetcode]
---

题目来源：[https://leetcode.com/problems/jewels-and-stones/description/](https://leetcode.com/problems/jewels-and-stones/description/)

标记难度：Easy

提交次数：1/1

代码效率：98.31%

## 题意

统计一个字符串中属于某一特定集合的字符的数量。

## 分析

超级大水题，直接用STL集合就行。

借此机会复习了一下[C++ set](https://en.cppreference.com/w/cpp/container/set)的用法：

* 定义：`set<int> s;`
* 查看大小：
  * 集合是否为空：`if (s.empty()) {}`
  * 集合中元素的数量：`int size = s.size();`
* 插入：`s.insert(val);`
* 删除：`s.erase(iterator);`
* 查找：
  * 计数：`int cnt = s.count(val);`
  * 查找：`auto iterator = s.find(val);`
  * （C++20里才会有`contains`函数可用，真是令人窒息……）

## 代码

```
class Solution {
public:
    int numJewelsInStones(string J, string S) {
        set<char> jewels;
        for (char c: J)
            jewels.insert(c);
        int ans = 0;
        for (char c: S)
            if (jewels.count(c) > 0)
                ans++;
        return ans;
    }
};
```
