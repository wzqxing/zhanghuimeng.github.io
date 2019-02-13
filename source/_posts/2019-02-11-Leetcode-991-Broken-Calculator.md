---
title: Leetcode 991. Broken Calculator
urlname: leetcode-991-broken-calculator
toc: true
date: 2019-02-11 15:00:35
updated: 2019-02-13 17:05:00
tags: [Leetcode, Leetcode Contest, alg:Math]
categories: Leetcode
---

题目来源：[https://leetcode.com/problems/broken-calculator/description/](https://leetcode.com/problems/broken-calculator/description/)

标记难度：Medium

提交次数：1/1

代码效率：100.00%（4ms）

## 题意

给定两个数`X`和`Y`，可以对`X`执行以下两种操作：

* 乘2
* 减1

问最少需要对`X`执行多少次操作才能将`X`变成`Y`？（`X`和`Y`的范围都是1e9）

## 分析

比赛的时候我居然首先就写了一个BFS，然后自然是超时了……

后来就思考怎么用数学方法解决，也没想出来，各种各样奇怪的贪心大部分也是错的。

后来想到了对`X`的操作就等同于对`Y`的除2和加1两种操作，不过想到了也没什么大的突破……

然后就没做出来……

---

这道题考虑对`Y`的操作比考虑对`X`的操作要更容易。如果对`Y`加1两次再除2，显然不如先除2再加1，因此不会出现两次连续的加1操作。题解[^sln]是这么说的，不过我感觉有一点不太严谨。倒不如这么说：首先假设有一个最优的操作顺序，需要除2`n`次，且在第一次除2之前需要加1`a[0]`次，在第二次除2之前需要加1`a[1]`次，……，在最后一次除2之后需要加1`a[n]`次。此时总操作次数为`a[0] + a[1] + ... + a[n] + n`。假如存在`i < n`且`a[i] > 1`，那显然可以把多余的+1操作下移，变成`a'[i] = a[i] - 2 * (a[i]/2)`，`a'[i+1] = a[i+1] + a[i]/2`，总操作次数减少`a[i]/2`次。如果`a'[i+1]`仍然大于1，则可以继续尝试下移，直到除了`a[n]`以外的所有`a[i]`都变成0或1为止。

事实上我大概是做了一个Exchange类型的贪心证明……

[^sln]: [Leetcode Official Solution for 991. Broken Calculator](https://leetcode.com/problems/broken-calculator/solution/)

## 代码

```cpp
class Solution {
public:
    int brokenCalc(int X, int Y) {
        int ans = 0;
        while (Y > X) {
            if (Y % 2 == 0) {
                Y /= 2;
                ans++;
            }
            else {
                Y = (Y + 1) / 2;
                ans += 2;
            }
        }
        return ans + (X - Y);
    }
};
```