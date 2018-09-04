---
title: Leetcode 637. Average of Levels in Binary Tree（BFS）
urlname: leetcode-637-average-of-levels-in-binary-tree
toc: true
date: 2018-09-04 20:19:14
updated: 2018-09-04 20:27:00
tags: [Leetcode, alg:Breadth-first Search, alg:Depth-first Search]
---

题目来源：[https://leetcode.com/problems/average-of-levels-in-binary-tree/description/](https://leetcode.com/problems/average-of-levels-in-binary-tree/description/)

标记难度：Easy

提交次数：1/2

代码效率：99.60%

## 题意

计算一棵二叉树每层结点的平均值。

## 分析

这道题和[Leetcode 102](/post/leetcode-102-binary-tree-level-order-traversal)差不多，不过那道题是层次遍历，这道题是求每层的平均值。具体做法也差不多，至少可以通过BFS和DFS两种做法完成层次遍历。

不过我又被`long long int`卡了一次，我真是非常容易忘记这一点。

## 代码

```cpp
class Solution {
public:
    vector<double> averageOfLevels(TreeNode* root) {
        if (root == nullptr) return {};

        vector<double> averages;
        queue<TreeNode*> q;
        q.push(root);
        while (!q.empty()) {
            // [2147483647,2147483647,2147483647]
            int size = q.size();
            long long int sum = 0;
            for (int i = 0; i < size; i++) {
                TreeNode* x = q.front();
                q.pop();
                sum += x->val;
                if (x->left != nullptr) q.push(x->left);
                if (x->right != nullptr) q.push(x->right);
            }
            averages.push_back((double) sum / size);
        }

        return averages;
    }
};
```
