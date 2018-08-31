---
title: Leetcode 894. All Possible Full Binary Trees（递归）
urlname: leetcode-894-all-possible-full-binary-trees
toc: true
mathjax: true
date: 2018-08-31 09:37:52
updated: 2018-08-31 10:08:52
tags: [Leetcode, LeetcodeContest, Tree, Recursion]
---

题目来源：[https://leetcode.com/problems/maximum-frequency-stack/description/](https://leetcode.com/problems/maximum-frequency-stack/description/)

标记难度：Medium

提交次数：1/1

代码效率：77.64%

## 题意

返回所有结点总数为`N`的完全二叉树。

## 分析

这是周赛（99）的第三题。比赛的时候我大概花了一个小时在这道题上，但是并没有做出来。我企图用一遍DFS完成所有的枚举任务，但事实上，如果不明确给定左子树和右子树的结点数目，是无法正常进行枚举的。直到最后我也没想出来怎么正常地枚举，于是我就挂了。

---

事实上这道题有非常明显的子问题结构，但我比赛的时候对这一点毫无知觉。显然，对于完全二叉树的每一个结点，要么它是一个叶结点，要么它的左右子树都是完全二叉树。而且显然完全二叉树的结点数目必然是奇数。于是我们可以通过递归解决这个问题：对于要求的结点总数`N`，枚举所有可能的左子树结点数（`i`，奇数）以及对应的右子树结点数（`N - i - 1`），然后把生成的子树拼在一起，得到所有可能的拼法。可以将`N`对应的所有树存起来，减少迭代的次数。

以及，令$N = 2k + 1$，则$N$个结点的不同构满二叉树的总数为卡特兰数$C_k = \frac{1}{k+1} \binom{2k}{k}$。从上述迭代公式应该是可以用数学归纳法证明的。[^solution]

[^solution]: [Leetcode 894 Solution](https://leetcode.com/problems/all-possible-full-binary-trees/solution/)

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
    unordered_map<int, vector<TreeNode*>> cache;

    void generate(int n) {
        if (n <= 0 || n % 2 == 0)
            return;
        if (cache.find(n) != cache.end())
            return;
        if (n == 1) {
            TreeNode* root = new TreeNode(0);
            cache[1].push_back(root);
            return;
        }
        for (int leftNum = 1; leftNum <= n - 1; leftNum += 2) {
            generate(leftNum);
            generate(n - 1 - leftNum);
            for (TreeNode* ltree: cache[leftNum])
                for (TreeNode* rtree: cache[n - 1 - leftNum]) {
                    TreeNode* root = new TreeNode(0);
                    root->left = copy(ltree);
                    root->right = copy(rtree);
                    cache[n].push_back(root);
                }
        }
    }

    TreeNode* copy(TreeNode* root) {
        if (root == NULL)
            return NULL;
        TreeNode* newRoot = new TreeNode(0);
        newRoot->left = copy(root->left);
        newRoot->right = copy(root->right);
        return newRoot;
    }

public:
    vector<TreeNode*> allPossibleFBT(int N) {
        generate(N);
        return cache[N];
    }
};
```
