---
title: Leetcode 922. Sort Array By Parity II（数组）
urlname: leetcode-922-sort-array-by-parity-ii
toc: true
date: 2018-10-14 16:18:57
updated: 2018-10-14 16:32:00
tags: [Leetcode, Leetcode Contest, alg:Array, alg:Two Pointers]
---

题目来源：[https://leetcode.com/problems/sort-array-by-parity-ii/description/](https://leetcode.com/problems/sort-array-by-parity-ii/description/)

标记难度：Easy

提交次数：2/2

代码效率：

* 额外空间：96ms
* 双指针：68ms

## 题意

有一个数组`A`，其中奇元素和偶元素的数量相等。请把`A`排序，使得奇元素位于奇数位置，偶元素位于偶数位置。任何满足上述条件的排序都是合法的。

## 分析

### 额外空间

需要额外空间的做法是非常简单的：再开一个数组`B`，维护`even`和`odd`指针，对于每一个`A[i]`，根据它的奇偶性放到`B`中对应的位置。

### 双指针

事实上并不需要额外的空间，而仍然可以采用类似于快排的做法：用两个指针分别维护合法的奇序列和合法的偶序列的结束位置，将元素交换，直到排序完成。[^solution]

[^solution]: [Leetcode 922 - Official Solution](https://leetcode.com/problems/sort-array-by-parity-ii/solution/)

## 代码

### 额外空间

```cpp
class Solution {
public:
    vector<int> sortArrayByParityII(vector<int>& A) {
        vector<int> B(A);
        int odd = 1, even = 0;
        for (int i = 0; i < A.size(); i++) {
            if (A[i] % 2 == 1) {
                B[odd] = A[i];
                odd += 2;
            }
            else {
                B[even] = A[i];
                even += 2;
            }
        }
        return B;
    }
};
```

### 双指针

```cpp
class Solution {
public:
    vector<int> sortArrayByParityII(vector<int>& A) {
        int odd = 1, even = 0, n = A.size();
        while (odd < n && even < n) {
            while (odd < n && A[odd] % 2 == 1) odd += 2;
            while (even < n && A[even] % 2 == 0) even += 2;
            if (odd >= n || even >= n) break;
            swap(A[odd], A[even]);
            odd += 2;
            even += 2;
        }
        return A;
    }
};
```
