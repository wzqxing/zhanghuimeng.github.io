---
title: Leetcode 687. Longest Univalue Path（递推）
urlname: leetcode-687-longest-univalue-path
toc: true
date: 2018-09-01 16:19:14
updated: 2018-09-01 16:23:00
tags: [Leetcode, Tree, Recursion]
---

题目来源：[https://leetcode.com/problems/longest-univalue-path/description/](https://leetcode.com/problems/longest-univalue-path/description/)

标记难度：Easy

提交次数：1/1

代码效率：39.40%

## 题意

给定一棵二叉树，找到树上满足下列条件的最长的路径：路径中每个结点的值都是相同的。

## 分析

这道题和[Leetcode 124](/post/leetcode-124-binary-tree-maximum-path-sum)基本是一样的，只是递推条件换成了路径中的结点是否相等。不过我也是很不理解，怎么那道题是Hard，这道题就变成Easy了。

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
    int maxLength;

    // number, length
    pair<int, int> dfs(TreeNode* root) {
        if (root == NULL)
            return {-1, -1};
        pair<int, int> leftPair = dfs(root->left);
        pair<int, int> rightPair = dfs(root->right);

        int length = 1;
        if (leftPair.second != -1 && leftPair.first == root->val)
            length = max(leftPair.second + 1, length);
        if (rightPair.second != -1 && rightPair.first == root->val)
            length = max(rightPair.second + 1, length);
        int sum = length;
        if (leftPair.second != -1 && leftPair.first == root->val && rightPair.second != -1 && rightPair.first == root->val)
            length = max(leftPair.second + rightPair.second + 1, length);
        maxLength = max(length, maxLength);

        return {root->val, sum};
    }

public:
    int longestUnivaluePath(TreeNode* root) {
        if (root == NULL)
            return 0;

        maxLength = -1;
        dfs(root);
        return maxLength - 1;
    }
};
```
