---
title: Leetcode 988. Smallest String Starting From Leaf（树）
urlname: leetcode-988-smallest String Starting From Leaf（树）
toc: true
date: 2019-02-05 16:25:48
updated: 2019-02-05 16:25:48
tags: [Leetcode, Leetcode Contest, alg:Tree]
---

题目来源：[https://leetcode.com/problems/smallest-string-starting-from-leaf/description/](https://leetcode.com/problems/smallest-string-starting-from-leaf/description/)

标记难度：Medium

提交次数：2/4

代码效率：

* BFS：4ms
* 暴力：0ms

## 题意

给定一棵二叉树，令它的每个结点表示一个字符，问从每个叶结点开始，到树根结束的所有字符串中最小的字符串。

## 分析

考虑到这道题的数据范围（二叉树的最高层的叶结点数量大约最多是总结点数量的一半），完全可以把所有可能的字符串都生成出来，然后从里面找最小的……

另一种稍微不那么暴力的方法是做BFS，找到出现叶结点的第一层，然后再回溯（或者直接暴力……）。

## 代码

### BFS

这份代码比较愚蠢的一点在于，它居然没有存每个结点对应的字符串，而是只存了每个结点的父节点的位置，这么写大概就是故意增加复杂度了……

```cpp
class Solution {
public:
    string smallestFromLeaf(TreeNode* root) {
        vector<pair<TreeNode*, pair<int, int>>> q;  // node, father, depth
        int start = 0, end;
        q.emplace_back(root, make_pair(-1, 0));
        while (start < q.size()) {
            end = q.size();
            while (start < end) {
                TreeNode* p = q[start].first;
                int depth = q[start].second.second;
                if (p->left != NULL) q.emplace_back(p->left, make_pair(start, depth + 1));
                if (p->right != NULL) q.emplace_back(p->right, make_pair(start, depth + 1));
                start++;
            }
        }

        vector<pair<int, string>> ans;  // index, string
        for (int i = q.size() - 1; i >= 0; i--) {
            TreeNode* p = q[i].first;
            int depth = q[i].second.second;
            if (p->left == NULL && p->right == NULL) ans.emplace_back(i, string(1, p->val + 'a'));
        }

        string minimal;
        while (ans.size() > 0) {
            vector<pair<int, string>> a;
            minimal = "";
            for (int i = 0; i < ans.size(); i++) {
                minimal = minimal == "" || minimal > ans[i].second ? ans[i].second : minimal;
            }
            for (int i = 0; i < ans.size(); i++) {
                if (ans[i].second == minimal) {
                    int last = q[ans[i].first].second.first;
                    if (last == -1) return minimal;
                    a.emplace_back(last, minimal + (char) (q[last].first->val + 'a'));
                }
            }
            ans = a;
        }
        
        return minimal;
    }
};
```

### 暴力

```cpp
class Solution {
private:
    string ans;
    void dfs(TreeNode* root, string s) {
        if (root == NULL) return;
        s += (char) (root->val + 'a');
        if (root->left == NULL && root->right == NULL) {
            reverse(s.begin(), s.end());
            ans = ans == "" || s < ans ? s : ans;
            return;
        }
        dfs(root->left, s);
        dfs(root->right, s);
    }
    
public:
    string smallestFromLeaf(TreeNode* root) {
        dfs(root, "");
        return ans;
    }
};
```