---
title: Leetcode 124. Binary Tree Maximum Path Sum（递推）
urlname: leetcode-124-binary-tree-maximum-path-sum
toc: true
date: 2018-08-31 18:56:09
updated: 2018-08-31 19:18:09
tags: [Leetcode, alg:Tree, alg:Recursion]
---

题目来源：[https://leetcode.com/problems/binary-tree-maximum-path-sum/description/](https://leetcode.com/problems/binary-tree-maximum-path-sum/description/)

标记难度：Hard

提交次数：2/3

代码效率：

* 不太简洁的版本：98.55%
* 比较简洁的版本：98.57%

## 题意

给定一棵非空二叉树，求树上所有路径中结点总和的最大值。

## 分析

这是一道比较简单的递推题。对于一棵有根树，显然树上的每一条路径都有一个深度最小的结点，所以这种做法可以覆盖到所有的路径。对于树上的每一个结点`x`，计算从`x`开始向下延伸的路径的最大结点总和，记为`maxDown(x)`；显然，这一总和只有3种可能性，取的是其中的最大值：

* `x.val + maxDown(x.left)`（如果左子树存在；此时路径向左子树延伸）
* `x.val + maxDown(x.right)`（如果右子树存在；此时路径向右子树延伸）
* `x.val`（因为负数结点值的存在，可能不如不延长路径；此时路径上只有`x`一个点）

通过DFS的方法可以比较方便地推出`maxDown(x)`。接下来的问题是，树上的路径除了向下延伸之外，还有另外一类，就是从某个结点开始，既向左也向右延伸。（之前忘了这种情况所以WA了一次。）因此需要另外计算`maxSum(x) = max(maxDown(x), x.val + maxDown(x.left) + maxDown(x.right))`。答案就是树上所有结点中`maxSum`的最大值。

## 代码

### 我的不太简洁的代码

这个代码对边界情况考虑得并不好，有更简洁也更好的写法。

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
    int maxn;

    int dfs(TreeNode* root) {
        if (root == NULL)
            return -1000000000;
        int leftSum = dfs(root->left);
        int rightSum = dfs(root->right);

        int endSum = max(leftSum + root->val, rightSum + root->val);
        endSum = max(endSum, root->val);
        int sum = endSum;
        endSum = max(endSum, leftSum + rightSum + root->val);
        maxn = max(endSum, maxn);

        return sum;
    }

public:
    int maxPathSum(TreeNode* root) {
        if (root == NULL)
            return 0;
        maxn = root->val;
        dfs(root);
        return maxn;
    }
};
```

### 比较简洁的代码

参考了[Simple O(n) algorithm with one traversal through the tree](https://leetcode.com/problems/binary-tree-maximum-path-sum/discuss/39869/Simple-O%28n%29-algorithm-with-one-traversal-through-the-tree)的写法。

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
    int maxn;

    int dfs(TreeNode* root) {
        if (root == NULL) return 0;
        int leftSum = dfs(root->left);
        int rightSum = dfs(root->right);

        if (leftSum < 0) leftSum = 0;
        if (rightSum < 0) rightSum = 0;
        maxn = max(maxn, max(max(leftSum, rightSum) + root->val, leftSum + rightSum + root->val));

        return max(leftSum, rightSum) + root->val;
    }

public:
    int maxPathSum(TreeNode* root) {
        if (root == NULL)
            return 0;
        maxn = root->val;
        dfs(root);
        return maxn;
    }
};
```
