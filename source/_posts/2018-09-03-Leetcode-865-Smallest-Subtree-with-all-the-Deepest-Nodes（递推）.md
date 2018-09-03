---
title: Leetcode 865. Smallest Subtree with all the Deepest Nodes（递推）
urlname: leetcode-865-smallest-subtree-with-all-the-deepest-nodes
toc: true
date: 2018-09-03 13:43:06
updated: 2018-09-03 14:39:06
tags: [Leetcode, alg:Tree, alg:Recursion, alg:Depth-first Search]
---

题目来源：[https://leetcode.com/problems/smallest-subtree-with-all-the-deepest-nodes/description/](https://leetcode.com/problems/smallest-subtree-with-all-the-deepest-nodes/description/)

标记难度：Medium

提交次数：1/1

代码效率：

* 两遍递归：61.92%
* 一遍递归：64.44%

## 题意

找出包含了树中所有深度最大的结点的最深的子树。

## 分析

一种简单的思路是，首先通过一遍深度优先搜索找出所有深度最大的结点的个数，然后再通过一遍深度优先搜索，计算左右子树中深度最大的结点的总个数，如果已经包含了所有最深的结点，则当前结点符合要求。因为是DFS，所以找到的第一个结点就是深度最大的子树的根。

但事实上我们可以通过递推的方法只进行一遍DFS。令`f(root) = (depth, deepestTreeNode)`，其中`depth`表示在以`root`为根的子树中最深的结点的深度，`deepestTreeNode`表示在该子树中，包含了所有最深的结点的最深的子树。然后就可以递推了：

* `f(root.left).depth > f(root.right).depth`时，说明最深的结点在左子树中，`f(root) = (f(root.left).depth+1, f(root.left).deepestTreeNode)`
* `f(root.left).depth < f(root.right).depth`时，说明最深的结点在右子树中，`f(root) = (f(root.right).depth+1, f(root.right).deepestTreeNode)`
* `f(root.left).depth = f(root.right).depth`时，说明两棵子树中都有最深的结点，`f(root) = (f(root.left).depth+1, root)`

最后返回的结果是`f(root).deepestTreeNode`。[^onepass]

[^onepass]: [One pass](https://leetcode.com/problems/smallest-subtree-with-all-the-deepest-nodes/discuss/146808/One-pass)

值得注意的是，在上述解法中，是以`root`为根计算深度的，而非绝对深度；当然用相对整棵树的绝对深度去做也可以，具体代码差不多，就不再列出了。

## 代码

### 两遍递归

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
    int maxDepth;
    int maxNum;

    void findMaxDepth(TreeNode* root, int depth) {
        if (root == NULL)
            return;
        if (depth == maxDepth)
            maxNum++;
        else if (depth > maxDepth) {
            maxDepth = depth;
            maxNum = 1;
        }
        findMaxDepth(root->left, depth + 1);
        findMaxDepth(root->right, depth + 1);
    }

    TreeNode* father;
    int maxDepthNum(TreeNode* root, int depth) {
        if (root == NULL)
            return 0;
        int sum = maxDepthNum(root->left, depth + 1) + maxDepthNum(root->right, depth + 1);
        if (father != NULL)
            return 0;
        if (depth == maxDepth)
            sum++;
        if (sum == maxNum)
            father = root;
        return sum;
    }

public:
    TreeNode* subtreeWithAllDeepest(TreeNode* root) {
        maxDepth = 0;
        maxNum = 0;
        findMaxDepth(root, 0);
        // cout << maxDepth << ' ' << maxNum << endl;
        father = NULL;
        maxDepthNum(root, 0);
        return father;
    }
};
```

### 一遍递归

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
    // depth, deepestTree
    typedef pair<int, TreeNode*> Node;
private:
    Node dfs(TreeNode* root) {
        if (root == NULL)
            return {0, NULL};
        Node leftNode(dfs(root->left));
        Node rightNode(dfs(root->right));
        if (leftNode.first < rightNode.first)
            return {rightNode.first+1, rightNode.second};
        if (leftNode.first > rightNode.first)
            return {leftNode.first+1, leftNode.second};
        return {leftNode.first+1, root};
    }

public:
    TreeNode* subtreeWithAllDeepest(TreeNode* root) {
        Node node(dfs(root));
        return node.second;
    }
};
```
