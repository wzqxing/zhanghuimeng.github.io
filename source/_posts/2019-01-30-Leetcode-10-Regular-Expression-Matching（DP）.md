---
title: Leetcode 10. Regular Expression Matching（DP）
urlname: leetcode-10-regular-expression-matching
toc: true
date: 2019-01-30 15:47:38
updated: 2019-01-30 15:47:38
tags: [Leetcode, alg:Dynamic Programming]
---

题目来源：[https://leetcode.com/problems/regular-expression-matching/description/](https://leetcode.com/problems/regular-expression-matching/description/)

标记难度：Hard

提交次数：2/8

代码效率：

* 递归：40ms
* DP：4ms

## 题意

给定一个字符串和一个只包含小写字母、`.`和`*`的模板字符串，问这两个字符串能否匹配？

## 分析

这道题我看到过大概也有些日子了，但是当时完全不知道应该怎么做，满脑子都是什么“把正则表达式转化成[非确定有限状态自动机（NFA）](https://zh.wikipedia.org/wiki/%E9%9D%9E%E7%A1%AE%E5%AE%9A%E6%9C%89%E9%99%90%E7%8A%B6%E6%80%81%E8%87%AA%E5%8A%A8%E6%9C%BA)，然后再转换成[确定有限自动机（DFA）](https://zh.wikipedia.org/wiki/%E7%A1%AE%E5%AE%9A%E6%9C%89%E9%99%90%E7%8A%B6%E6%80%81%E8%87%AA%E5%8A%A8%E6%9C%BA)，然后再进行匹配……”当然，要是真这么写出来，肯定也不是不能做，但是我也懒得去写……

于是这回我就直接先写了个递归匹配，居然就过了。当然，这大概是因为数据比较弱……

结果“正解”（或者说时间复杂度和编程复杂度都比较合理的一种解法）竟然是动态规划。想了想，其实挺合理的：不就是简化一下递归匹配方法的状态数量吗。

## 代码

### 递归

```cpp
class Solution {
public:
    bool isMatch(string s, string p) {
        int n = s.length(), m = p.length();
        if (n == 0 && m == 0) return true;
        if (m == 0) return false;

        if (m > 1 && p[1] != '*' || m == 1) {
            if (n == 0) return false;
            if (p[0] == '.')
                return isMatch(s.substr(1, n - 1), p.substr(1, m - 1));
            else {
                if (s[0] != p[0]) return false;
                return isMatch(s.substr(1, n - 1), p.substr(1, m - 1));
            }
        }
        for (int i = 0; i <= n; i++) {
            if (isMatch(s.substr(i, n - i), p.substr(2, m - 2))) return true;
            if (p[0] != '.' && s[i] != p[0]) break;
        }
        return false;
    }
};
```

### DP

DP的解法其实就是在递归解法上改出来的。结果因为各种各样的细节WA了超级多次……

顺便还复习了一下多维vector的初始化：`vector<vector<int>> vec(r, vector<int>(c, 0));`[^csdn]

[^csdn]: [CSDN - C++——二维vector初始化大小方法](https://blog.csdn.net/sinat_36053757/article/details/71053706)

如果真的是面试题，很难一遍写对的吧。

```cpp
class Solution {
private:
    int ns, np;
    string s, p;
    vector<vector<int>> f;
    // ls和lp是串和模板串剩余的长度
    // 考虑的是s[-ls:]和p[-lp:]
    bool calc(int ls, int lp) {
        if (ls == 0 && lp == 0) return true;
        if (lp == 0) return false;
        if (f[ls][lp] != -1) return f[ls][lp];
        // 不是匹配多个的*
        if (lp > 1 && p[np - lp + 1] != '*' || lp == 1) {
            if (ls == 0 || p[np - lp] != '.' && s[ns - ls] != p[np - lp]) {
                f[ls][lp] = false;
                return false;
            }
            f[ls][lp] = calc(ls - 1, lp - 1);
            return f[ls][lp];
        }
        // 是匹配多个的*，尝试匹配i个
        for (int i = 0; i <= ls; i++) {
            if (calc(ls - i, lp - 2)) {
                f[ls][lp] = true;
                return true;
            }
            if (p[np - lp] != '.' && s[ns - ls + i] != p[np - lp]) break;
        }
        f[ls][lp] = false;
        return false;
    }
    
public:
    bool isMatch(string s, string p) {
        ns = s.length();
        np = p.length();
        this->s = s;
        this->p = p;
        f = vector<vector<int>>(ns + 1, vector<int>(np + 1, -1));
        return calc(ns, np);
    }
};
```