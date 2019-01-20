---
title: Leetcode 977. Squares of a Sorted Array
urlname: leetcode-977-squares-of-a-sorted-array
toc: true
date: 2019-01-20 14:49:13
updated: 2019-01-20 17:02:00
tags: [Leetcode, Leetcode Contest, alg:Array, alg:Two Pointers]
---

题目来源：[https://leetcode.com/problems/squares-of-a-sorted-array/description/](https://leetcode.com/problems/squares-of-a-sorted-array/description/)

标记难度：Easy

提交次数：2/3

代码效率：

* 排序：124ms
* 双指针：104ms

## 题意

给定一组已经排序了的整数，返回它们的平方排序后的结果。

## 分析

最简单的方法就是直接平方之后排序，复杂度是`O(N^log(N))`。

复杂一点的话，就是利用这些整数已经排序了的条件，找到中间最靠近0的数，然后分别“合并”两边平方的大小。举个例子：对于`[-3, -2, -1, 4, 5, 6]`，这么做相当于合并`[-1, -2, -3]`和`[4, 5, 6]`平方的结果。具体的实现方法也是用两个指针。[^sln]

[^sln]: [Leetcode official solution for 977. Squares of a Sorted Array](https://leetcode.com/problems/squares-of-a-sorted-array/solution/)

## 代码

### 排序

```cpp
class Solution {
public:
    vector<int> sortedSquares(vector<int>& A) {
        vector<int> B;
        for (int x: A)
            B.push_back(x*x);
        sort(B.begin(), B.end());
        return B;
    }
};
```

### 双指针

实现的时候需要稍微注意一下判断条件……

```cpp
class Solution {
public:
    vector<int> sortedSquares(vector<int>& A) {
        int n = A.size();
        int i = 0;
        while (i < n - 1 && A[i] < 0) i++;
        if (A[i] > 0) i--;
        int j = i + 1;
        vector<int> ans;
        while (i >= 0 || j < n) {
            if (i < 0 || j < n && abs(A[i]) >= abs(A[j])) {
                ans.push_back(A[j] * A[j]);
                j++;
            }
            else if (j >= n || i >= 0 && abs(A[i]) <= abs(A[j])) {
                ans.push_back(A[i] * A[i]);
                i--;
            }
        }
        return ans;
    }
};
```