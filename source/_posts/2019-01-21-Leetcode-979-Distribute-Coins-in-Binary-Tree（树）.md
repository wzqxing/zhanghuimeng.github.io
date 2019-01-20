---
title: Leetcode 979. Distribute Coins in Binary Tree（树）
urlname: leetcode-979-distribute-coins-in-binary-tree
toc: true
date: 2019-01-21 00:24:25
updated: 2019-01-21 00:37:00
tags: [Leetcode, Leetcode Contest, alg:Tree, alg:Depth-first Search]
---

题目来源：[https://leetcode.com/problems/distribute-coins-in-binary-tree/description/](https://leetcode.com/problems/distribute-coins-in-binary-tree/description/)

标记难度：Medium

提交次数：1/1

代码效率：8ms

## 题意

有一颗二叉树，共有`N`个结点，每个结点上有若干枚硬币，硬币总数为`N`。每次操作可以把一枚硬币移动到相邻的结点。问最少进行几次操作，才能使每个结点上恰好有一枚硬币？

## 分析

我觉得这道题挺简单的……

我的思路很简单：先DFS，到达最底层之后判断每个叶结点上的硬币数量`node->val`是否恰好为1，如果大于1则把多出来的硬币向父结点移动，总移动数量`+=node->val-1`，如果小于1则**把少的硬币向父结点移动**，也就是移动`-(1-node->val)`枚硬币，总移动数量`+=1-node->val`。对于其他的结点，因为它们的子结点都已经递归处理完了，所以有多出来的硬币的时候，只能向上移动若干枚硬币或者负的若干枚硬币，也就是和叶结点差不多。

这里面比较好的是移动**负数枚硬币**这个想法，否则可能还得做第二次递归。

## 代码

```cpp
class Solution {
private:
    int ans;
    
    int dfs(TreeNode* root) {
        if (root == NULL) return 0;
        int x = dfs(root->left) + dfs(root->right);
        x += root->val;
        ans += abs(x - 1);
        root->val = 1;
        return x - 1;
    }
    
public:
    int distributeCoins(TreeNode* root) {
        ans = 0;
        dfs(root);
        return ans;
    }
};
```