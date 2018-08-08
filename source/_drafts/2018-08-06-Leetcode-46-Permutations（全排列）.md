---
title: Leetcode 46. Permutations（全排列）
urlname: leetcode-46-permutations
toc: true
date: 2018-08-06 09:44:30
updated: 2018-08-06 09:44:30
tags:
---

题目来源：[https://leetcode.com/problems/permutations/description/](https://leetcode.com/problems/permutations/description/)

标记难度：Medium

提交次数：1/1

代码效率：

* `next_permutation`版本：99.91%
* DFS版本：
* Heap算法：
* Steinhaus–Johnson–Trotter算法：

## 题意

## 分析

## 代码

### STL版本

```cpp
class Solution {
public:
    vector<vector<int>> permute(vector<int>& nums) {
        vector<vector<int>> ans;
        sort(nums.begin(), nums.end());  // 如果使用next_permutation()，是需要排序的
        do {
            vector<int> x(nums);
            ans.push_back(x);
        } while (next_permutation(nums.begin(), nums.end()));
        return ans;
    }
};
```
