---
title: Leetcode 951. Flip Equivalent Binary Trees（树）
urlname: leetcode-951-flip-equivalent-binary-trees
toc: true
date: 2018-12-02 13:19:02
updated: 2018-12-02 14:44:00
tags: [Leetcode, Leetcode Contest, alg:Tree]
---

题目来源：[https://leetcode.com/problems/flip-equivalent-binary-trees/description/](https://leetcode.com/problems/flip-equivalent-binary-trees/description/)

标记难度：Medium

提交次数：1/1

代码效率：4ms

## 题意

对二叉树定义flip操作：任取一个结点，然后交换它的左子树和右子树。给定两棵二叉树，问经过flip操作后能否使这两棵树相等。

## 分析

题解[^solution]中的第一种做法是递归：先判断根是否相等，然后分两种情况进行判断：左子树、右子树分别相等，或者左子树等于右子树，右子树等于左子树。这种做法肯定是对的，但它的复杂度怎么分析呢？不妨设$N = 2^m - 1$，为满二叉树，时间复杂度为$T(N)$。则

{% raw %}
$$T(N) = O(1) + 4T((N-1)/2), \, T(1)=O(1)$$
{% endraw %}

即

{% raw %}
$$
\begin{aligned}
T(2^m - 1) =& O(1) + 4T(2^{m-1}-1) \\
=& O(1) + O(1) + 2^4T(2^{m-2}-1) \\
=& \cdots \\
=& (m-1) O(1) + 2^{2(m-1)}T(1) \\
=& O(N^2)
\end{aligned}
$$
{% endraw %}

分析出来的时间复杂度是`O(N^2)`，我觉得题解写错了。

[^solution]: [LeetCode Official Solution for 951](https://leetcode.com/problems/flip-equivalent-binary-trees/solution/)

第二种做法（也是我的做法）大概更好：为二叉树定义一种“小于”的概念，然后把更小的子树都换到左边，再比较两棵树是否相等；这样每棵树都对应一棵唯一等价的树。当然，这道题规定结点的值都是唯一的，这使得定义方法简化了。如果结点的值可能相等，那仍然大概需要递归定义。比如，先比较根节点的大小，如果相等则递归比较左子树的大小，比较不出来则再比较右子树，什么之类的。

## 代码

```cpp
class Solution {
    void flip(TreeNode* root) {
        if (root == NULL) return;
        TreeNode* left = root->left;
        TreeNode* right = root->right;
        flip(left);
        flip(right);
        if (left == NULL && right == NULL || left != NULL && right == NULL) return;
        if (left == NULL && right != NULL || left->val > right->val) {
            swap(root->left, root->right);
            return;
        }
        return;
    }

    bool equal(TreeNode* root1, TreeNode* root2) {
        if (root1 == NULL || root2 == NULL)
            return root1 == NULL && root2 == NULL;
        if (root1->val != root2->val) return false;
        return equal(root1->left, root2->left) && equal(root1->right, root2->right);
    }

public:
    bool flipEquiv(TreeNode* root1, TreeNode* root2) {
        // always flip the sub-tree with smaller integer left
        // and flip null right.
        flip(root1);
        flip(root2);
        return equal(root1, root2);
    }
};
```