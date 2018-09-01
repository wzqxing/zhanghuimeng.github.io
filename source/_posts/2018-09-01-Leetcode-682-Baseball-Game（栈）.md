---
title: Leetcode 682. Baseball Game（栈）
urlname: leetcode-682-baseball-game
toc: true
date: 2018-09-01 17:49:24
updated: 2018-09-01 19:49:00
tags: [Leetcode, Stack]
---

题目来源：[https://leetcode.com/problems/baseball-game/description/](https://leetcode.com/problems/baseball-game/description/)

标记难度：Easy

提交次数：1/1

代码效率：96.89%

## 题意

对一个栈中的元素进行以下几种操作：

* 入栈一个元素
* 把栈顶前两个元素的和入栈
* 把栈顶元素的2倍入栈
* 弹出一个元素

求最后栈中所有元素的和。

## 分析

简单的栈操作。不过题目描述有点莫名其妙，获得了很多差评……

## 代码

```cpp
class Solution {
public:
    int calPoints(vector<string>& ops) {
        vector<int> rounds;
        int n = 0, sum = 0;
        for (string op: ops) {
            if (op == "+") {
                rounds.push_back(rounds[n-1] + rounds[n-2]);
                n++;
            }
            else if (op == "C") {
                rounds.pop_back();
                n--;
            }
            else if (op == "D") {
                rounds.push_back(rounds[n-1] * 2);
                n++;
            }
            else {
                rounds.push_back(stoi(op));
                n++;
            }
        }
        for (int i = 0; i < n; i++)
            sum += rounds[i];
        return sum;
    }
};
```
