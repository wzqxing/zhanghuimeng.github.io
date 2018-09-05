---
title: Leetcode 111. Minimum Depth of Binary Tree（BFS）
urlname: leetcode-111-minimum-depth-of-binary-tree
toc: true
date: 2018-09-05 11:03:12
updated: 2018-09-05 11:17:00
tags: [Leetcode, alg:Breadth-first Search, alg:Depth-first Search]
---

题目来源：[https://leetcode.com/problems/minimum-depth-of-binary-tree/description/](https://leetcode.com/problems/minimum-depth-of-binary-tree/description/)

标记难度：Easy

提交次数：2/2

代码效率：

* BFS：100.00%
* 递推：100.00%

## 题意

找出二叉树中深度最小的结点的深度。

## 分析

显然用BFS可以做，DFS也可以。另一种可能的做法是递推，从子结点的最小深度推出当前子树的最小深度。[^stefan]事实上，我现在觉得，递推是把代码写得简短且不需要额外函数的一种好方法。

[^stefan]: [3 lines in Every Language](https://leetcode.com/problems/minimum-depth-of-binary-tree/discuss/36060/3-lines-in-Every-Language)

## 代码

### BFS

```cpp
class Solution {
public:
    int minDepth(TreeNode* root) {
        if (root == nullptr) return 0;
        queue<pair<int, TreeNode*>> q;
        q.emplace(1, root);
        while (!q.empty()) {
            TreeNode* x = q.front().second;
            int depth = q.front().first;
            q.pop();

            if (x->left == nullptr && x->right == nullptr) return depth;
            if (x->left != nullptr) q.emplace(depth+1, x->left);  // 居然写成了root……
            if (x->right != nullptr) q.emplace(depth+1, x->right);
        }
        return -1;
    }
};
```

### 递推

```cpp
class Solution {
public:
    int minDepth(TreeNode* root) {
        if (root == nullptr) return 0;
        int L = minDepth(root->left), R = minDepth(root->right);
        // 如果L,R>0，说明两棵子树都不为空，最浅的叶结点可能在两棵子树中，所以取min
        // 如果L,R中至少有一个数为0，说明至少有一棵子树为空，最浅的叶结点只能在另一棵子树中，所以取max
        // 如果L==R==0，说明root就是叶结点，这种情况包含在上一种里
        // 因为是子树所以深度要+1
        return 1 + ((L > 0 && R > 0) ? min(L, R) : max(L, R));
    }
};
```
