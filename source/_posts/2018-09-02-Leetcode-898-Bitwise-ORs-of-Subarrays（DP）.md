---
title: 2018-09-02-Leetcode 898. Bitwise ORs of Subarrays（DP）
urlname: leetcode-898-bitwise-ors-of-subarrays
toc: true
date: 2018-09-02 15:32:33
updated: 2018-09-02 16:19:00
tags: [Leetcode, Leetcode Contest, alg:Bit Manipulation, alg:Dynamic Programming]
---

题目来源：[https://leetcode.com/problems/bitwise-ors-of-subarrays/description/](https://leetcode.com/problems/bitwise-ors-of-subarrays/description/)

标记难度：Medium

提交次数：1/2

代码效率：

* 暴力：过不了
* 优化过的暴力：796ms

## 题意

给定一个正整数数组，求其中（连续）子序列的**或**总共有多少种可能取值。

## 分析

我也不知道这道题做了多久，总之首先就看错题了（把“或”看成了“异或”），做出来是不太可能的了。其次当时Leetcode服务器大概比较爆炸，平均5分钟才能运行一次。

---

记数组为`A`，显然我们的目标就是求出所有`or(i, j) = A[i] | ... | A[j] (i <= j)`的可能取值。如果直接暴力计算，则复杂度为`O(N^3)`。一个显而易见的优化是，对于当前的`j`，我们可以维护一个数组，其中保存了所有以`j`结尾的子序列的取值；`or(0, j), or(1, j), ..., or(j, j)`的值；对于`j+1`，我们可以通过这些值递推出所有以`j+1`结尾的子序列的取值：`or(0, j+1)=or(0, j)|A[j+1], or(1, j+1)=or(1, j)|A[j+1], ... or(j, j+1)=or(j, j)|A[j+1], or(j+1, j+1)=A[j+1]`。于是复杂度降低到`O(N^2)`。

（也许我可以推导到这一步，但是后来我信心丧失直接去看题解了）

然后是一个信仰之跃：我们要利用一下或操作的性质，在一个或和中不断或上新的数值，其二进制表示中`1`的数量不会减少。刚才维护的子序列除了能够递推之外，还满足一个很好的性质：`or(0, j) = or(1, j) | A[0] = ... = or(j, j) | A[0] | A[1] | .. | A[j-1]`；因此，`cnt1(or(0, j)) >= cnt1(or(1, j)) >= ... >= cnt1(or(j, j))`，且同一位置上的`1`出现后就不会消失。而题目给定了`0 <= A[i] <= 10^9`，所以这些数的二进制位最多只有30个。

这也就说明，以`j`结尾的所有子序列的或和的取值最多只有30种。

所以只需把上述保存以`j`结尾的子序列的或和的数组改成`HashSet`，然后维护这个`HashSet`中的值，就可以得到`O(30N)`的复杂度。

（上述内容基本参考的都是[^lee215]。）

[^lee215]: [O(30N)](https://leetcode.com/problems/bitwise-ors-of-subarrays/discuss/165881/C++JavaPython-O%2830N%29)

## 代码

### 暴力

过了八十多个测试点中的七十多个，果然`O(N^2)`暴力还是不可取。

```cpp
class Solution {
public:
    int subarrayBitwiseORs(vector<int>& A) {
        int n = A.size();
        int ends[n + 1];
        unordered_set<int> allSet;

        for (int i = 0; i < n; i++) {
            for (int j = 0; j < i; j++)
                ends[j] |= A[i];
            ends[i] = A[i];
            for (int j = 0; j <= i; j++)
                allSet.insert(ends[j]);
        }

        return allSet.size();
    }
};
```

### 优化过的暴力

```cpp
class Solution {
public:
    int subarrayBitwiseORs(vector<int>& A) {
        unordered_set<int> sums;
        unordered_set<int> all;
        int n = A.size();
        sums.insert(0);
        for (int i = 0; i < n; i++) {
            unordered_set<int> sums2 = {A[i]};
            for (const auto& j: sums)
                sums2.insert(j | A[i]);
            for (const auto& j: sums2)
                all.insert(j);
            sums = sums2;
        }
        return all.size();
    }
};
```
