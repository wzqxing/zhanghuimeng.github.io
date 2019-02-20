---
title: Leetcode 993. Cousins in Binary Tree（树）
urlname: leetcode-993-cousins-in-binary-tree
toc: true
date: 2019-02-19 02:51:51
updated: 2019-02-19 02:51:51
tags: [Leetcode, Leetcode Contest, alg:Tree]
categories: Leetcode
---

题目来源：[https://leetcode.com/problems/cousins-in-binary-tree/description/](https://leetcode.com/problems/cousins-in-binary-tree/description/)

标记难度：Easy

提交次数：1/1

代码效率：8ms

## 题意

定义二叉树中的“堂兄弟”结点：两个深度相同且父结点不同的结点。给定一棵结点值互异的二叉树和两个树中的结点值`x`和`y`，问这两个结点是否为堂兄弟。

## 分析

只要遍历二叉树，并记录对应的两个结点的父节点和深度即可。不过为了降低复杂度，甚至可以干脆把每个结点的父节点和深度都记录下来，然后在里面查。

## 代码

这里遍历用的是DFS。

```cpp
class Solution {
private:
    int father[102];
    int depth[102];
    
    void dfs(TreeNode* root, int d, int p) {
        if (root == NULL) return;
        father[root->val] = p;
        depth[root->val] = d;
        dfs(root->left, d + 1, root->val);
        dfs(root->right, d + 1, root->val);
    }
    
public:
    bool isCousins(TreeNode* root, int x, int y) {
        dfs(root, 0, -1);
        return depth[x] == depth[y] && father[x] != father[y];
    }
};
```
