---
title: Leetcode 102. Binary Tree Level Order Traversal（BFS）
urlname: leetcode-102-binary-tree-level-order-traversal
toc: true
date: 2018-09-04 19:30:24
updated: 2018-09-04 20:04:00
tags: [Leetcode, alg:Breadth-first Search, alg:Depth-first Search]
---

题目来源：[https://leetcode.com/problems/binary-tree-level-order-traversal/description/](https://leetcode.com/problems/binary-tree-level-order-traversal/description/)

标记难度：Medium

提交次数：2/2

代码效率：

* 未优化的DFS：0.00%
* 优化过的DFS：98.83%

## 题意

返回一棵二叉树的层次遍历结果。不同层要分开。

## 分析

这道题和[Leetcode 429](/post/leetcode-429-n-ary-tree-level-order-traversal)差不多，那道题是多叉树遍历，这道题是二叉树遍历，似乎还更简单一些。我参考其中DFS的解法写了一份代码，结果很慢，后来我意识到，`vector`之间的拷贝可能花了太多时间，还不如记录当前结点的深度，然后直接把结点的值放到对应的层里面，于是就快了很多。[^java]

[^java]: [Java Solution using DFS](https://leetcode.com/problems/binary-tree-level-order-traversal/discuss/33445/Java-Solution-using-DFS)

## 代码

### 未优化的DFS

```cpp
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        if (root == NULL) return {};
        vector<vector<int>> level;
        level.push_back({root->val});
        vector<TreeNode*> children = {root->left, root->right};
        for (TreeNode* ch: children) {
            vector<vector<int>> chLevel = levelOrder(ch);
            for (int i = 0; i < chLevel.size(); i++) {
                if (i + 1 < level.size())
                    level[i + 1].insert(level[i + 1].end(), chLevel[i].begin(), chLevel[i].end());
                else
                    level.push_back(chLevel[i]);
            }
        }
        return level;
    }
};
```

### 优化的DFS

```cpp
class Solution {
private:

    vector<vector<int>> levels;

    // just put the node value in levels[depth]
    void dfs(TreeNode* root, int depth) {
        if (root == NULL) return;
        if (depth < levels.size())
            levels[depth].push_back(root->val);
        else
            levels.push_back({root->val});
        dfs(root->left, depth+1);
        dfs(root->right, depth+1);
    }

public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        dfs(root, 0);
        return levels;
    }
};
```
