---
title: Leetcode 199. Binary Tree Right Side View（BFS）
urlname: leetcode-199-binary-tree-right-side-view
toc: true
date: 2018-09-05 14:56:00
updated: 2018-09-05 14:38:36
tags: [Leetcode, alg:Tree, alg:Breadth-first Search, alg:Depth-first Search]
---

题目来源：[https://leetcode.com/problems/binary-tree-right-side-view/description/](https://leetcode.com/problems/binary-tree-right-side-view/description/)

标记难度：Medium

提交次数：2/2

代码效率：

* BFS：100.00%
* DFS：100.00%

## 题意

给定一棵二叉树，求每一层最右侧的结点的值。

## 分析

显然，一种最直接的想法是用BFS对树进行分层遍历，然后就可以立刻得出每一层最右侧的结点的值了。

另一种方法和[Leetcode 102](/post/leetcode-102-binary-tree-level-order-traversal)的其中一种解题思路很像：使用DFS，然后按照深度把结点放到对应的层里面。但此处的问题是，每一层只需要放最右侧的结点。解决这一问题的方法是，访问子树时，先访问右子结点，再访问左子结点，这样每一层最先被访问到的结点都是最右侧的结点，我们只要在该层为空时才插入结点就可以了。[^dfs]

[^dfs]: [Simple C++ solution (BTW: I like clean codes)](https://leetcode.com/problems/binary-tree-right-side-view/discuss/56203/Simple-C++-solution-%28BTW:-I-like-clean-codes%29)

## 代码

### BFS

```cpp
class Solution {
public:
    vector<int> rightSideView(TreeNode* root) {
        if (root == NULL)
            return {};

        queue<TreeNode*> q;
        vector<int> rightSide;
        q.push(root);
        while (!q.empty()) {
            int right = -1;
            // 此处新开了一个队列作为记录，事实上没有必要
            // 只要记录本层结点的个数就可以了
            queue<TreeNode*> q2;
            while (!q.empty()) {
                TreeNode* x = q.front();
                q.pop();
                right = x->val;
                if (x->left != nullptr)
                    q2.push(x->left);
                if (x->right != nullptr)
                    q2.push(x->right);
            }
            q = q2;
            rightSide.push_back(right);
        }
        return rightSide;
    }
};
```

### DFS

```cpp
class Solution {
private:
    void dfs(TreeNode* root, int depth, vector<int>& views) {
        if (root == nullptr) return;
        // 事实上写的时候是自顶向下的，所以是逐渐push_back
        if (depth >= views.size()) views.push_back(root->val);
        dfs(root->right, depth+1, views);
        dfs(root->left, depth+1, views);
    }

public:
    vector<int> rightSideView(TreeNode* root) {
        vector<int> views;
        dfs(root, 0, views);
        return views;
    }
};
```
