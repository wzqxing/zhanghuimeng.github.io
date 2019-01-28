---
title: Leetcode 984. String Without AAA or BBB（贪心）
urlname: leetcode-984-string-without-aaa-or-bbb
toc: true
date: 2019-01-28 17:52:38
updated: 2019-01-28 17:52:38
tags: [Leetcode, Leetcode Contest, alg:Greedy]
---

题目来源：[https://leetcode.com/problems/string-without-aaa-or-bbb/description/](https://leetcode.com/problems/string-without-aaa-or-bbb/description/)

标记难度：Easy

提交次数：1/1

代码效率：100.00%（0ms）

## 题意

给定自然数`A`和`B`，返回任意满足下列要求的字符串`S`：

* `S`的长度为`A+B`，且恰好包含`A`个`"a"`，`B`个`"b"`
* `S`中不包含任何`"aaa"`
* `S`中不包含任何`"bbb"`

## 分析

其实就是要求返回一个`"a"/"aa"`和`"b"/"bb"`交替出现的字符串。这道题怎么想都能做，不过我觉得一种比较简单的方法是，轮流在字符串后面添加`"a"`和`"b"`，如果当前`A > B`就添加两个`"a"`，否则添加一个`"a"`，然后再考虑`"b"`的情形，以此类推。

我写得比较麻烦，题解里更简洁一些。

## 代码

```cpp
class Solution {
public:
    string strWithout3a3b(int A, int B) {
        string s;
        string a = "a", b = "b";
        // 确保第一个字符可以是A对应的字符
        if (A < B) {
            swap(a, b);
            swap(A, B);
        }
        while (A > 0 || B > 0) {
            // 添加一个a，两个a还是不添加？
            if (A > B) {
                if (A >= 2) {
                    s += a + a;
                    A -= 2;
                }
                else if (A > 0) {
                    s += a;
                    A--;
                }
            }
            else if (A > 0) {
                s += a;
                A--;
            }
            // 添加一个b，两个b还是不添加？
            if (B > A) {
                if (B >= 2) {
                    s += b + b;
                    B -= 2;
                }
                else if (B > 0) {
                    s += b;
                    B--;
                }
            }
            else if (B > 0) {
                s += b;
                B--;
            }
        }
        return s;
    }
};
```
