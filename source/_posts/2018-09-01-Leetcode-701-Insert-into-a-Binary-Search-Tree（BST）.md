---
title: Leetcode 701. Insert into a Binary Search Tree（BST）
urlname: leetcode-701-insert-into-a-binary-search-tree
toc: true
date: 2018-09-01 16:31:34
updated: 2018-09-01 16:34:00
tags: [Leetcode, alg:Binary Search Tree]
---

题目来源：[https://leetcode.com/problems/insert-into-a-binary-search-tree/description/](https://leetcode.com/problems/insert-into-a-binary-search-tree/description/)

标记难度：Easy

提交次数：1/1

代码效率：15.50%

## 题意

在二叉搜索树中插入元素。

## 分析

感觉和[Leetcode 700](/post/leetcode-700-search-in-a-binary-search-tree)是同一系列的题目。同样直接插入就可以了。

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
    TreeNode* insertIntoBST(TreeNode* root, int val) {
        if (root == NULL)
            return new TreeNode(val);
        TreeNode* p = root;
        while (p != NULL) {
            if (val < p->val) {
                if (p->left == NULL) {
                    p->left = new TreeNode(val);
                    break;
                }
                p = p->left;
            }
            else {
                if (p->right == NULL) {
                    p->right = new TreeNode(val);
                    break;
                }
                p = p->right;
            }
        }
        return root;
    }
};
```
