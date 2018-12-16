---
title: Leetcode 958. Check Completeness of a Binary Tree（树）
urlname: leetcode-958-check-completeness-of-a-binary-tree
toc: true
date: 2018-12-16 20:20:28
updated: 2018-12-16 21:29:00
tags: [Leetcode, Leetcode Contest, alg:Tree]
---

题目来源：[https://leetcode.com/problems/check-completeness-of-a-binary-tree/description/](https://leetcode.com/problems/check-completeness-of-a-binary-tree/description/)

标记难度：Medium

提交次数：2/4

代码效率：

* 比赛时的做法：4ms
* 题解的做法：4ms

## 题意

判断一棵树是否为完全二叉树。

## 分析

比赛的时候我的做法是，首先给整棵树做一个层次遍历，然后尝试找到完全二叉树里第一个位于倒数第二层的至少有一个空孩子的结点；如果在更高的层就有结点没有两个孩子，或者发现之后的结点又有孩子了，则这不是完全二叉树。不过这种判断有很多tricky的地方，所以我错了两次。而且代码写得很不优雅。不过当然比赛时写代码的第一目标也不是优雅……

---

题解[^solution]的做法不错：直接判断按照一个堆的形态来组织树，能否得到一个中间没有空隙的数组。不需要把数组真的建出来，只要给每个结点分配一个index，然后判断index总数是否和结点总数相同就行了。

[^solution]: [Leetcode Official Solution for 958](https://leetcode.com/problems/check-completeness-of-a-binary-tree/solution/)

## 代码

### 层次遍历后直接判断

```cpp
class Solution {
public:
    bool isCompleteTree(TreeNode* root) {
        vector<pair<TreeNode*, int>> q;
        int maxDepth = -1;
        int n = 0;
        q.emplace_back(root, 0);
        while (n < q.size()) {
            TreeNode* p = q[n].first;
            int depth = q[n].second;
            maxDepth = depth;
            n++;
            if (!p->left && p->right) return false;
            if (p->left) q.emplace_back(p->left, depth + 1);
            if (p->right) q.emplace_back(p->right, depth + 1);
        }
        if (maxDepth == 0) return true;  // 如果不特判大小为1的树会错
        bool foundm1 = false;
        for (int i = 0; i < q.size(); i++) {
            TreeNode* p = q[i].first;
            int depth = q[i].second;
            if (foundm1) {
                if (p->left || p->right) return false;
            }
            else {
                // 因为没加depth != maxDepth所以错了一次；显然最后一层也是没有孩子的
                if (depth != maxDepth - 1 && depth != maxDepth && (!p->left || !p->right)) return false;
                if (depth == maxDepth - 1 && (!p->left || !p->right)) foundm1 = true;
            }
        }
        return true;
    }
};
```

### 题解的做法

```cpp
class Solution {
public:
    bool isCompleteTree(TreeNode* root) {
        vector<pair<TreeNode*, int>> q;
        q.emplace_back(root, 1);
        int n = 0;
        while (n < q.size()) {
            TreeNode* p = q[n].first;
            int index = q[n].second;
            n++;
            if (p->left) q.emplace_back(p->left, index * 2);
            else if (p->right) return false;
            if (p->right) q.emplace_back(p->right, index * 2 + 1);
        }
        return q.size() == q.back().second;
    }
};
```