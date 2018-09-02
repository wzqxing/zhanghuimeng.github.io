---
title: Leetcode 899. Orderly Queue
urlname: leetcode-899-orderly-queue
toc: true
date: 2018-09-02 17:41:14
updated: 2018-09-02 20:17:00
tags: [Leetcode, alg:Math, alg:String]
---

题目来源：[https://leetcode.com/problems/orderly-queue/description/](https://leetcode.com/problems/orderly-queue/description/)

标记难度：Hard

提交次数：1/1

代码效率：4ms

## 题意

给定一个字符串和正整数`K`，可以对该字符串进行任意次下列操作：将字符串的前`K`的字符之一移动到字符串最后。求上述操作所能得到的字典序最小的字符串。

## 分析

在比赛的时候我尝试做了一下这道题，感觉暴力应该时间复杂度太高，贪心或者动态规划的方法一时又想不出来。我尝试手动模拟一个长度为5的字符串的暴力结果（`K=2`），但没有模拟完。

---

其实暴力的做法对于一些较小的数据是有用的，用处不在于解题，而在于观察结果。[^solution]

[^solution]: [Leetcode 899 Solution](https://leetcode.com/problems/orderly-queue/solution/)

如果真的把所有的结果都列出来了（虽然对于长度为5的字符串，`5!=120`，还是太大了，不适合手动模拟），就会发现，`K=2`时，这种操作可以得到字母的所有可能排列。事实上，当`K>=2`时，上述移动操作相当于可以任意交换字符串中相邻的两个字母，也就是相当于冒泡排序。所以`K>=2`时，只需将字符串排序即可。`K=1`时，可以将字符串循环枚举一遍。[^bubble]

[^bubble]: [K>1 is bubblesort](https://leetcode.com/problems/orderly-queue/discuss/165862/Kgreater1-is-bubblesort)

## 代码

```cpp
class Solution {
public:
    string orderlyQueue(string S, int K) {
        if (K > 1) {
            sort(S.begin(), S.end());
            return S;
        }
        set<string> smallSet;
        for (int i = 0; i < S.length(); i++) {
            smallSet.insert(S);
            S = S.substr(1, S.length() - 1) + S[0];
        }
        return *smallSet.begin();
    }
};
```
