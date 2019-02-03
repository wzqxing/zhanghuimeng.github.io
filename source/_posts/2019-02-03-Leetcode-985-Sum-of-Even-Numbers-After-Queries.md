---
title: Leetcode 985. Sum of Even Numbers After Queries
urlname: leetcode-985-sum-of-even-numbers-after-queries
toc: true
date: 2019-02-03 19:02:27
updated: 2019-02-03 19:02:27
tags: [Leetcode, Leetcode Contest, alg:Array]
---

题目来源：[https://leetcode.com/problems/sum-of-even-numbers-after-queries/description/](https://leetcode.com/problems/sum-of-even-numbers-after-queries/description/)

标记难度：Easy

提交次数：1/1

代码效率：96ms

## 题意

给定一个数组`A`，对其中的数做以下操作：

* 更新：`A[index] += val`
* 查询：问`A`中所有偶数的和

## 分析

首先预处理出`A`中所有偶数的和，然后在每次更新的时候，对这个和相应地进行更新。总的来说是道水题……

## 代码

```cpp
class Solution {
public:
    vector<int> sumEvenAfterQueries(vector<int>& A, vector<vector<int>>& queries) {
        int n = A.size();
        int sum = 0;
        for (int i = 0; i < n; i++)
            if (A[i] % 2 == 0)
                sum += A[i];
        vector<int> ans;
        for (int i = 0; i < queries.size(); i++) {
            int x = queries[i][0], k = queries[i][1];
            if (A[k] % 2 == 0) {
                if (x % 2 == 0) sum += x;
                else sum -= A[k];
            }
            else {
                if (x % 2 != 0) sum += A[k] + x;
            }
            A[k] += x;
            ans.push_back(sum);
        }
        return ans;
    }
};
```