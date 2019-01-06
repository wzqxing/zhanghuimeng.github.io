---
title: Leetcode 971. Flip Binary Tree To Match Preorder Traversal（树）
urlname: leetcode-971-flip-binary-tree-to-match-preorder-traversal
toc: true
date: 2019-01-07 00:41:33
updated: 2019-01-07 01:12:00
tags: [Leetcode, Leetcode Contest, alg:Tree, alg:Depth-first Search]
---

题目来源：[https://leetcode.com/problems/flip-binary-tree-to-match-preorder-traversal/description/](https://leetcode.com/problems/flip-binary-tree-to-match-preorder-traversal/description/)

标记难度：Medium

提交次数：1/1

代码效率：12ms

## 题意

给定一棵二叉树（每个结点对应的值都是唯一的）和一个前序遍历序列，问，能否通过交换其中一些结点左右子树，使得这棵树的前序遍历结果和给定的相同？

## 分析

既然是前序遍历，那么接下来的遍历序列中的第一个值必然需要和根结点相同，如果不等，则肯定不能实现。

接下来的问题就是要不要翻转。如果有左子结点，那么下一个值必然是左子结点的值；否则，如果有右子结点，下一个值必然是右子结点的值；否则（也就是说没有子结点），下一个值和之前的遍历相关，可能是父节点的右子结点之类的。

所以做出翻转的决策就很简单：如果左子结点有值，且这个值和遍历序列中预期的下一个值不等，那么就交换左右子树（显然不交换肯定不行了）。判断是否可行的方法也很简单，在上述翻转的基础上，每次都判断当前根结点的值和遍历序列中预期的值是否相等。

---

事实上，这个判断左子结点的方法是题解[^solu]里给出的。我比赛的时候用的是，判断是否有右子结点，如果有的话，它的值如果和预期的下一个值相等，才有可能有必要进行交换（当然也有可能根本实现不了）。这个判断本质上和上述方法是一样的。

[^solu]: [Leetcode Official Solution - 971. Flip Binary Tree To Match Preorder Traversal](https://leetcode.com/problems/flip-binary-tree-to-match-preorder-traversal/solution/)

## 代码

```cpp
class Solution {
    vector<int> ans;
    int index;
    int n;
    
    bool pre(TreeNode* root, vector<int>& voyage) {
        if (root == NULL) return true;
        if (root->val != voyage[index]) return false;
        if (index == n - 1 && (root->left != NULL || root->right != NULL)) return false;
        // cout << root->val << endl;
        if (root->left != NULL && root->right != NULL && voyage[index + 1] == root->right->val) {
            swap(root->left, root->right);
            ans.push_back(root->val);
        }
        index++;
        if (!pre(root->left, voyage)) return false;
        if (!pre(root->right, voyage)) return false;
        return true;
    }
    
public:
    vector<int> flipMatchVoyage(TreeNode* root, vector<int>& voyage) {
        index = 0;
        n = voyage.size();
        bool ok = pre(root, voyage);
        if (!ok) return {-1};
        return ans;
    }
};
```