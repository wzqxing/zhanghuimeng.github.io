---
title: Leetcode 965. Univalued Binary Tree（树）
urlname: leetcode-965-univalued-binary-tree
toc: true
date: 2018-12-31 02:27:41
updated: 2018-12-31 14:17:00
tags: [Leetcode, Leetcode Contest, alg:Tree, alg:Depth-first Search]
---

题目来源：[https://leetcode.com/problems/univalued-binary-tree/description/](https://leetcode.com/problems/univalued-binary-tree/description/)

标记难度：Easy

提交次数：1/1

代码效率：4ms（100.00%）

## 题意

判断一棵二叉树中结点的值是否都相同。

## 分析

直接DFS判断即可。用其他遍历方法当然也行。总之是道很简单的题。

## 代码

```cpp
class Solution {
private:
    int univalue;
    
    bool dfs(TreeNode* root) {
        if (root == NULL) return true;
        if (root->val != univalue)
            return false;
        return dfs(root->left) && dfs(root->right);
    }
    
public:
    bool isUnivalTree(TreeNode* root) {
        univalue = root->val;
        return dfs(root);
    }
};
```
