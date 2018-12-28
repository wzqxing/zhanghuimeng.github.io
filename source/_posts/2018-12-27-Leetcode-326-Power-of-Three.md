---
title: Leetcode 326. Power of Three（数学）
urlname: leetcode-326-power-of-three
toc: true
date: 2018-12-28 01:13:46
updated: 2018-12-28 20:00:00
tags: [Leetcode, alg:Math]
---

题目来源：[https://leetcode.com/problems/least-operators-to-express-number/](https://leetcode.com/problems/least-operators-to-express-number/)

标记难度：Easy

提交次数：1/3

代码效率：

* 模3法：84ms（19.17%）
* 算log法：88ms（13.78%）
* 打表法：80ms（30.44%）
* 除法：84ms（19.38%）

## 题意

给定一个整数，判断它是否是3的幂。

追加：可以不用循环和递归吗？

## 分析

这可能确实是个水题，但是看到解法的数量[^solutions]和testcase的数量时，我改变了想法。还是有值得一写的东西的。

[^solutions]: [LeetCode Official Solution - 326. Power of Three](https://leetcode.com/articles/power-of-three/)

![21038个testcases？？](testcases.png)

最简单的想法就是把这个数不断除3，直到只剩1（是3的幂）或者除不尽（不是3的幂）为止。复杂度为`O(log(N))`。

上述想法的一个升级版是，不妨直接计算出`log(N)/log(3)`，然后判断结果是否为整数。显然需要考虑到运算精度的问题。我感觉可能专门有一些testcase是考察精度取多小的，最后我发现1e-10是比较合理的。事实上`int`范围内的3的幂最大是1162261467，而`log(1162261467)/log(3) - log(1162261467 - 1)/log(3) = 7.83e-10`，`log(1162261467 + 1)/log(3) - log(1162261467)/log(3) = 7.83e-10`，能判断出这么小的精度就够了。

这不禁会让人想到打表这种方法，毕竟`int`范围内的3的幂一共也没几个，从1到1162261467（`3^19`）。

除了打表之外，还有一种更简单的方法。直接用1162261467去除`N`，能除尽就是3的幂，除不尽就不是……真是简单粗暴。

## 代码

### 模3法

```cpp
class Solution {
public:
    bool isPowerOfThree(int n) {
        if (n <= 0) return false;
        while (n > 1) {
            if (n % 3 != 0) return false;
            n /= 3;
        }
        return true;
    }
};
```

### 算log法

```cpp
class Solution {
public:
    bool isPowerOfThree(int n) {
        if (n <= 0) return false;
        double pow = log(n) / log(3);
        // cout << pow - floor(pow) << ' ' << ceil(pow) - pow << endl;
        return pow - floor(pow) <= 1e-10 || ceil(pow) - pow <= 1e-10;
    }
};
```

### 神奇的方法

非常简洁……

```cpp
class Solution {
public:
    bool isPowerOfThree(int n) {
        return !(n <= 0) && 1162261467 % n == 0;
    }
};
```