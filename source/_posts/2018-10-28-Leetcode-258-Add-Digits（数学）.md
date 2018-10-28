---
title: 2018-10-28-Leetcode 258. Add Digits（数学）
urlname: leetcode-258-add-digits
toc: true
mathjax: true
date: 2018-10-28 14:58:04
updated: 2018-10-28 16:59:00
tags: [Leetcode, alg:Math]
---

题目来源：[https://leetcode.com/problems/add-digits/description/](https://leetcode.com/problems/add-digits/description/)

标记难度：Easy

提交次数：2/2

代码效率：

* 暴力法：99.13%
* 公式法：99.13%

## 题意

不断对某个数的所有数位求和，直到只剩下一位为止。求最后剩下的一位。

## 分析

显然可以直接暴力模拟。不过事实上有更好的方法。求得的结果的学名叫[数根（digital root）](https://en.wikipedia.org/wiki/Digital_root#Congruence_formula)。在十进制下：

$$dr(n) = 1 + ((n-1) \bmod 9)$$

然后直接代公式就可以了。（至于公式的证明，我觉得可以尝试一下，但现在还是算了吧）

## 代码

### 暴力法

```cpp
class Solution {
public:
    int addDigits(int num) {
        if (num < 10) return num;
        int x = 0;
        while (num > 0) {
            x += num % 10;
            num /= 10;
        }
        return addDigits(x);
    }
};
```

### 公式法

```cpp
class Solution {
public:
    int addDigits(int num) {
        return 1 + (num - 1) % 9;
    }
};
```
