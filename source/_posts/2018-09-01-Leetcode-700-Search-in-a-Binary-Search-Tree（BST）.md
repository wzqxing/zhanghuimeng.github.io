---
title: Leetcode 700. Search in a Binary Search Tree（BST）
urlname: leetcode-700-search-in-a-binary-search-tree
toc: true
date: 2018-09-01 16:25:39
updated: 2018-09-01 16:29:00
tags: [Leetcode, alg:Binary Search Tree]
---

题目来源：[https://leetcode.com/problems/search-in-a-binary-search-tree/description/](https://leetcode.com/problems/search-in-a-binary-search-tree/description/)

标记难度：Easy

提交次数：1/1

代码效率：14.34%

## 题意

在二叉搜索树中查找元素。

## 分析

水题。直接按BST的性质查找就可以了。

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
public:
    TreeNode* searchBST(TreeNode* root, int val) {
        TreeNode* p = root;
        while (p != NULL) {
            if (p->val == val)
                break;
            if (val < p->val)
                p = p->left;
            else
                p = p->right;
        }
        return p;
    }
};
```
