---
title: Leetcode 623. Add One Row to Tree（树）
urlname: leetcode-623-add-one-row-to-tree
toc: true
date: 2018-08-15 00:57:05
updated: 2018-08-15 01:12:05
tags: [Leetcode, alg:Tree]
---

题目来源：[https://leetcode.com/problems/add-one-row-to-tree/description/](https://leetcode.com/problems/add-one-row-to-tree/description/)

标记难度：Medium

提交次数：1/1

代码效率：

* DFS：10.14%
* 栈：6.76%
* BFS：98.07%

## 题意

按照一定的规则，在一棵二叉树中增加一整层。

## 分析

总的来说是道水题，不过我开始做的时候并不能立刻反应出DFS、栈和BFS这三种[经典思路](https://leetcode.com/problems/add-one-row-to-tree/solution/)。当然，这里面DFS是最好写的，且思路的本质是一样的。

以及我发现，如果现在输入数据是树，Leetcode的自定义输入数据框在聚焦的时候，上方会自动渲染出树的图形。Leetcode真是越来越强了……

## 代码

这三份代码的内容真是差不多。但是队列实现的BFS明显比另外两种更快，不知道是什么缘故。

### DFS

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
    void findDepth(TreeNode* curNode, int curDep, const int& targetDep, const int& value) {
        if (curNode == NULL)
            return;
        if (curDep < targetDep) {
            findDepth(curNode->left, curDep + 1, targetDep, value);
            findDepth(curNode->right, curDep + 1, targetDep, value);
            return;
        }
        if (curDep == targetDep) {
            TreeNode* newLeft = new TreeNode(value);
            newLeft->left = curNode->left;
            TreeNode* newRight = new TreeNode(value);
            newRight->right = curNode->right;
            curNode->left = newLeft;
            curNode->right = newRight;
            return;
        }
    }

public:
    TreeNode* addOneRow(TreeNode* root, int v, int d) {
        if (d == 1) {
            TreeNode* newRoot = new TreeNode(v);
            newRoot->left = root;
            return newRoot;
        }

        findDepth(root, 1, d - 1, v);
        return root;
    }
};
```

### 栈

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
public:
    TreeNode* addOneRow(TreeNode* root, int v, int d) {
        if (d == 1) {
            TreeNode* newRoot = new TreeNode(v);
            newRoot->left = root;
            return newRoot;
        }
        stack<pair<TreeNode*, int>> s;
        d--;
        s.push(make_pair(root, 1));
        while (!s.empty()) {
            pair<TreeNode*, int> p = s.top();
            s.pop();
            TreeNode* cur = p.first;
            int depth = p.second;
            if (depth > d)
                continue;
            if (depth < d) {
                if (cur->left != NULL)
                    s.push(make_pair(cur->left, depth + 1));
                if (cur->right != NULL)
                    s.push(make_pair(cur->right, depth + 1));
                continue;
            }
            TreeNode* newLeft = new TreeNode(v);
            newLeft->left = cur->left;
            TreeNode* newRight = new TreeNode(v);
            newRight->right = cur->right;
            cur->left = newLeft;
            cur->right = newRight;
        }
        return root;
    }
};
```

### BFS

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
public:
    TreeNode* addOneRow(TreeNode* root, int v, int d) {
        if (d == 1) {
            TreeNode* newRoot = new TreeNode(v);
            newRoot->left = root;
            return newRoot;
        }
        queue<pair<TreeNode*, int>> q;
        d--;
        q.push(make_pair(root, 1));
        while (!q.empty()) {
            pair<TreeNode*, int> p = q.front();
            q.pop();
            TreeNode* cur = p.first;
            int depth = p.second;
            if (depth > d)
                continue;
            if (depth < d) {
                if (cur->left != NULL)
                    q.push(make_pair(cur->left, depth + 1));
                if (cur->right != NULL)
                    q.push(make_pair(cur->right, depth + 1));
                continue;
            }
            TreeNode* newLeft = new TreeNode(v);
            newLeft->left = cur->left;
            TreeNode* newRight = new TreeNode(v);
            newRight->right = cur->right;
            cur->left = newLeft;
            cur->right = newRight;
        }
        return root;
    }
};
```
