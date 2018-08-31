---
title: Leetcode 129. Sum Root to Leaf Numbers（枚举）
urlname: leetcode-129-sum-root-to-leaf-numbers
toc: true
date: 2018-08-31 15:24:07
updated: 2018-08-31 15:29:07
tags: [Leetcode, Tree, DFS]
---

题目来源：[https://leetcode.com/problems/sum-root-to-leaf-numbers/description/](https://leetcode.com/problems/sum-root-to-leaf-numbers/description/)

标记难度：Easy

提交次数：1/1

代码效率：100.00%

## 题意

给定一棵二叉树，树上的每个结点都是一个`0-9`的数位，问所有从根到叶结点的路径代表的数字之和。

## 分析

这个题和[Leetcode 112](/post/leetcode-112-path-sum)差不多，主要区别是没有剪枝的必要了，因为是求所有数字的和。题解区里也没看见什么特别惊人的做法。

## 代码

```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
private:
    int sum;

    void dfs(TreeNode* root, int num) {
        if (root == NULL)
            return;
        num = num * 10 + root->val;
        if (root->left == NULL && root->right == NULL) {
            sum += num;
            return;
        }
        dfs(root->left, num);
        dfs(root->right, num);
    }

public:
    int sumNumbers(TreeNode* root) {
        sum = 0;
        dfs(root, 0);
        return sum;
    }
};
```
