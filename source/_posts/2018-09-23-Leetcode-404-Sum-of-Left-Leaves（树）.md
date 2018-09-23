---
title: Leetcode 404. Sum of Left Leaves（树）
urlname: leetcode-404-sum-of-left-leaves
toc: true
date: 2018-09-23 11:32:32
updated: 2018-09-23 11:38:00
tags: [Leetcode, alg:Tree]
---

题目来源：[https://leetcode.com/problems/sum-of-left-leaves/description/](https://leetcode.com/problems/sum-of-left-leaves/description/)

标记难度：Easy

提交次数：1/1

代码效率：1.40%


## 题意

给定一棵二叉树，计算树中所有左侧叶结点的值之和。

## 分析

水题……

当然，仍然至少有三种策略：递归DFS，用栈模拟DFS，以及BFS。我在代码中明确给出了`left`和`right`的标记，用来识别当前结点是父结点的左子结点还是右子结点；当然，这并不是必要的。[^java]

[^java]: [Java iterative and recursive solutions](https://leetcode.com/problems/sum-of-left-leaves/discuss/88950/Java-iterative-and-recursive-solutions)

## 代码

```cpp
class Solution {
    int dfs(TreeNode* root, bool isLeft) {
        if (root == nullptr) return 0;
        if (root->left == nullptr && root->right == nullptr) {
            if (isLeft) return root->val;
            return 0;
        }
        return dfs(root->left, true) + dfs(root->right, false);
    }

public:
    int sumOfLeftLeaves(TreeNode* root) {
        return dfs(root, false);
    }
};
```
