---
title: Leetcode 513. Find Bottom Left Tree Value（BFS）
urlname: leetcode-513-find-bottom-left-tree-value
toc: true
date: 2018-09-05 15:13:38
updated: 2018-09-05 15:27:00
tags: [Leetcode, alg:Tree, alg:Breadth-first Search, alg:Depth-first Search]
---

题目来源：[https://leetcode.com/problems/find-bottom-left-tree-value/description/](https://leetcode.com/problems/find-bottom-left-tree-value/description/)

标记难度：Medium

提交次数：1/1

代码效率：98.56%

## 题意

给定一棵二叉树，找到最底层最靠左的结点的值。

## 分析

这道题看起来像是[Leetcode 513](/post/leetcode-199-binary-tree-right-side-view)的反过来的超简化版，因为只需要最底层的最左边的结点。我是用BFS直接做的——逐层访问，记录每层最左侧的结点，最后找到最底层最左侧的结点——不过这个思路可以再简化一些：对于每一层都从右往左访问，这样我们就不需要记录每层最左侧的结点，只需要返回最后一个访问的结点就可以了。[^stefan]

[^stefan]: [Right-to-Left BFS (Python + Java)](https://leetcode.com/problems/find-bottom-left-tree-value/discuss/98779/Right-to-Left-BFS-%28Python-+-Java%29)

## 代码

```cpp
class Solution {
public:
    int findBottomLeftValue(TreeNode* root) {
        queue<TreeNode*> q;
        TreeNode* bottomLeft = nullptr;
        q.push(root);
        while (!q.empty()) {
            TreeNode* left = nullptr;
            // 这个代码实际上复用了Leetcode 513的BFS
            // 同理，仍然可以不新开一个queue的
            queue<TreeNode*> q2;
            while (!q.empty()) {
                TreeNode* x = q.front();
                q.pop();
                if (left == nullptr)
                    left = x;
                if (x->left != nullptr)
                    q2.push(x->left);
                if (x->right != nullptr)
                    q2.push(x->right);
            }
            q = q2;
            bottomLeft = left;
        }
        return bottomLeft->val;
    }
};
```
