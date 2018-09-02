---
title: Leetcode 897. Increasing Order Search Tree（树的遍历）
urlname: leetcode-897-increasing-order-search-tree
toc: true
date: 2018-09-02 14:28:25
updated: 2018-09-02 15:27:00
tags: [Leetcode, LeetcodeContest, Tree, Traverse]
---

题目来源：[https://leetcode.com/problems/monotonic-array/description/](https://leetcode.com/problems/monotonic-array/description/)

标记难度：Easy

提交次数：1/1

代码效率：

* 中序遍历（未优化）：68ms
* 中序遍历+额外空间：60ms
* 优化的中序遍历：44ms

## 题意

给定一棵二叉树，要求把原来的树按照中序遍历的顺序重新组织，拉成一条长链，第一个结点在树根，下一个结点是它的右孩子，以此类推。

## 分析

这道题写了二十多分钟，错了三次，心态崩了。题目描述原来给的是一棵二叉搜索树，于是我就直接把结点全都保存下来，排了个序，然后重新接起来。如果原来的树是BST，那么中序遍历就相当于是排序，这么做没有什么问题，只是复杂度高点而已。但事实上并不是，所以在心态崩坏中错了三次，最后自己悟出来了可能的原因，交了个看上去像是旋转的方法上去。

---

很显然可以使用链表之类的数据结构把中序遍历的结点顺序保存下来，然后再重新连接结点，时间复杂度是`O(N)`，额外空间复杂度是`O(N)`。

除此之外，一种简单的方法是在递归中序遍历的基础上修改一下。先递归修改左子树，然后把左子树的最右侧的结点连到当前的根上，然后递归修改右子树，把当前的根的右子树设为递归得到的新子树。我比赛的时候是这么写的，当时自以为是一种“旋转”；但这和一般所说的BST的“旋转”相差有点远，何况这也不是个BST。

上述做法的时间复杂度为`O(N^2)`，因为在最坏情况下（整棵树完全是一个向左延伸的链）枚举左子树最右侧结点的平摊复杂度大致是`O(N/2)`，需要枚举`O(N)`次。

可以通过一些方法改善这一复杂度，比如在递归修改左子树的时候，把根节点也作为参数传递过去，然后在递归结束时，把根节点连到合适的位置上。[^tail]

[^tail]: [Self-Explained, 5-line, O(N)](https://leetcode.com/problems/increasing-order-search-tree/discuss/165885/C++JavaPython-Self-Explained-5-line-O%28N%29)

## 代码

### 中序遍历（未优化）

比赛的时候脑补出来的代码。

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
    TreeNode* rotate(TreeNode* root) {
        if (root == NULL)
            return NULL;
        if (root->left == NULL && root->right == NULL)
            return root;
        TreeNode* newRoot = root;
        if (root->left != NULL) {
            newRoot = rotate(root->left);
            TreeNode* p = newRoot;
            while (p->right != NULL) {
                p = p->right;
            }
            p->right = root;
            root->left = NULL;
        }
        if (root->right != NULL) {
            root->right = rotate(root->right);
        }
        return newRoot;
    }

    void dfs(TreeNode* root) {
        if (root == NULL)
            return;
        dfs(root->left);
        cout << root->val << endl;
        dfs(root->right);
    }

public:
    TreeNode* increasingBST(TreeNode* root) {
        if (root == NULL)
            return NULL;
        return rotate(root);
    }
};
```

### 中序遍历+额外空间

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
    vector<TreeNode*> inOrder;

    void traverse(TreeNode* root) {
        if (root == NULL) return;
        traverse(root->left);
        inOrder.push_back(root);
        traverse(root->right);
    }

public:
    TreeNode* increasingBST(TreeNode* root) {
        traverse(root);
        root = NULL;
        TreeNode* last = NULL;
        for (TreeNode* x: inOrder) {
            x->left = x->right = NULL;
            if (root == NULL) {
                root = last = x;
            }
            else {
                last->right = x;
                last = x;
            }
        }
        return root;
    }
};
```

### 优化的中序遍历

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

    TreeNode* traverse(TreeNode* root, TreeNode* tail) {
        if (root == NULL) return NULL;
        TreeNode* newRoot = root;
        if (root->right != NULL)
            root->right = traverse(root->right, tail);
        else
            root->right = tail;
        if (root->left != NULL) {
            newRoot = traverse(root->left, root);
            root->left = NULL;
        }
        return newRoot;
    }

public:
    TreeNode* increasingBST(TreeNode* root) {
        return traverse(root, NULL);
    }
};
```

上面是我的代码。如果把递归的截止条件下移一层，规整一下，会变得非常好看（但思路可能更不好理解），就像下面这份代码一样：

```cpp
// author: lee215
// https://leetcode.com/problems/increasing-order-search-tree/discuss/165885/C++JavaPython-Self-Explained-5-line-O(N)
TreeNode* increasingBST(TreeNode* root, TreeNode* tail = NULL) {
    if (!root) return tail;
    TreeNode* res = increasingBST(root->left, root);
    root->left = NULL;
    root->right = increasingBST(root->right, tail);
    return res;
}
```
