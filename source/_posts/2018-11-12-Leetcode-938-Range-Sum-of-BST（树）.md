---
title: Leetcode 938. Range Sum of BST（树）
urlname: leetcode-938-range-sum-of-bst
toc: true
date: 2018-11-12 00:31:39
updated: 2018-11-12 00:49:00
tags: [Leetcode, Leetcode Contest, alg:Binary Search Tree]
---

题目来源：[https://leetcode.com/problems/reorder-log-files/description/](https://leetcode.com/problems/reorder-log-files/description/)

标记难度：Medium

提交次数：1/1

代码效率：

* 剪枝：120ms
* 不剪枝：120ms

## 题意

给定一棵二叉查找树、`L`和`R`，返回`[L, R]`中的结点的值的和。

## 分析

这个题总的来说很简单。直接DFS，然后根据当前`node`的值进行剪枝即可。（事实上不剪枝也能过，而且没慢多少。）

## 代码

### 不剪枝

```cpp
class Solution {
public:
    int rangeSumBST(TreeNode* root, int L, int R) {
        if (root == NULL) return 0;
        return rangeSumBST(root->left, L, R) + rangeSumBST(root->right, L, R) + 
            (L <= root->val && root->val <= R ? root->val : 0);
    }
};
```

### 剪枝

```cpp
class Solution {
public:
    int rangeSumBST(TreeNode* root, int L, int R) {
        if (root == NULL) return 0;
        int sum = 0;
        if (root->val > L) sum += rangeSumBST(root->left, L, R);
        if (root->val < R) sum += rangeSumBST(root->right, L, R);
        return sum + (L <= root->val && root->val <= R ? root->val : 0);
    }
};
```