---
title: Leetcode 338. Counting Bits（动态规划）
urlname: leetcode-338-counting-bits
toc: true
date: 2018-08-05 10:50:53
updated: 2018-08-05 10:50:53
tags: [Leetcode, alg:Dynamic Programming, alg:Bit Manipulation]
---

题目来源：[https://leetcode.com/problems/counting-bits/description/](https://leetcode.com/problems/counting-bits/description/)

标记难度：Medium

提交次数：2/2

代码效率：

* naive算法：99.67%
* 动态规划：99.67%

## 题意

分别计算出`[0, N]`中每个数字的二进制表示中`1`的数量。（注意是分别计算，返回的结果是一个vector，而不是总和。）

进阶：

* 想出时间复杂度为`O(N * sizeof(integer))`的算法是很容易的。你能否尝试用`O(N)`的时间和空间复杂度解决这个问题？
* 请不要使用类似于C++中的`__builtin_popcount`的函数。

## 分析

### navie算法

好吧，其实我之前从未听说过`__builtin_popcount`这个函数，所以我立即去查了一下，并用到了我的naive版本的算法中。事实上，这是一个**GCC编译器**[内置的函数](https://gcc.gnu.org/onlinedocs/gcc/Other-Builtins.html)（而不是属于C或C++标准），利用特定CPU的指令（比如x86的[LZCNT](https://www.felixcloutier.com/x86/LZCNT.html)）完成计算二进制数中`1`的个数的操作。未来版本的C++[可能也会加入相应的支持](https://www.quora.com/What-is-__builtin_popcount-in-c++)。

所以这么做差不多可以达到时间复杂度为`O(N)`的要求。

### 动态规划

但是如果不用这种指令呢……？我开始时只能想到把一个32位数分成4或者8份然后分别去查表，但这样只是减少了一点常数，并不能摆脱`sizeof(integer)`的限制。于是我去看了[题解](https://leetcode.com/problems/counting-bits/discuss/79539/Three-Line-Java-Solution)——

事实上，因为题目要求统计所有数的二进制表示中`1`的数量，这反而为我们提供了很大的便利。虽然统计一个数的代价很难降低，但显然，一个数的二进制表示和它之前的数是相关的：`binary(x) = binary(floor(x/2)) || lastbit(x)`。因此我们可以采用类似于动态规划的方法直接推导出当前的数的二进制表示中`1`的数量。

## 代码

### naive算法

```
class Solution {
public:
    vector<int> countBits(int num) {
        // naive的算法想都不用想。但是不naive的……
        // 我感觉二进制表示已经是很fundamental的了，不知道怎么优化比较好。
        // 突然想到一个简单的方法：把每个32位数分成4或8份，然后直接查表。但是这并没有突破sizeof...
        vector<int> ans;
        for (int i = 0; i <= num; i++)
            ans.push_back(__builtin_popcount(i));
        return ans;
    }
};
```

### 动态规划法

```
class Solution {
public:
    vector<int> countBits(int num) {
        vector<int> ans;
        ans.push_back(0);
        for (int i = 1; i <= num; i++)
            ans.push_back(ans[i >> 1] + (i & 1));
        return ans;
    }
};
```
