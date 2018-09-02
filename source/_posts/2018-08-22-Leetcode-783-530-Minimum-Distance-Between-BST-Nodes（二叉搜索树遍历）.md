---
title: Leetcode 783 (530). Minimum Distance Between BST Nodes（二叉搜索树遍历）
urlname: leetcode-783-530-minimum-distance-between-bst-nodes
toc: true
date: 2018-08-22 11:03:57
updated: 2018-08-22 11:21:00
tags: [Leetcode, alg:Binary Search Tree]
---

题目来源：[https://leetcode.com/problems/minimum-distance-between-bst-nodes/description/](https://leetcode.com/problems/minimum-distance-between-bst-nodes/description/)

标记难度：Easy

提交次数：1/1

代码效率：12.62%

## 题意

给定一个二叉搜索树，返回树中任意两个结点之间差的最小值。

## 分析

Leetcode有些题目，真是水得有点过头……这道题数据规模只有100，简直是直接枚举都能做了。当然正确的做法是中序遍历。我最开始还以为要像[173题](/post/leetcode-173-binary-search-tree-iterator)那样手写一个找后继的方法，结果发现根本不用，直接中序遍历然后记录上一个元素就行了。

甚至更水的是，这道题和[530题](https://leetcode.com/problems/minimum-absolute-difference-in-bst/description/)基本一模一样（通过[评论区](https://leetcode.com/problems/minimum-distance-between-bst-nodes/discuss/118531/Come-on-guys-it-is-obviously-the-same-as-Problems-530.-Minimum-Absolute-Difference-in-BST)的提示得知……），直接用同一份代码交上去就可以AC了，于是我收获了两个AC……

不过，一个有趣的问题是，如果我们把530题的描述改一下，从“差的绝对值”改成“绝对值的差”会怎么样？这样改过之后，我觉得我们实际上获得了0-2棵子BST，以及一棵“反的”子BST（因为从负数取反了）。从树的角度来做处理其实不太好做，还不如干脆就把负数结点从树里全删掉，然后再把它们的绝对值加回去，最后再遍历。当然，更简单的做法是把树中的全部结点的绝对值都取出来，排序，然后直接做……

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
    int last;
    int minDif;
    void traverse(TreeNode* cur) {
        if (cur == NULL)
            return;
        traverse(cur->left);
        if (last != -1)
            minDif = min(minDif, cur->val - last);
        last = cur->val;
        traverse(cur->right);
    }

public:
    int minDiffInBST(TreeNode* root) {
        last = -1;
        minDif = 1000000;
        traverse(root);
        return minDif;
    }
};
```
