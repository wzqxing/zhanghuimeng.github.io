---
title: Leetcode 987. Vertical Order Traversal of a Binary Tree（树）
urlname: leetcode-987-vertical-order-traversal-of-a-binary-tree
toc: true
date: 2019-02-04 21:59:35
updated: 2019-02-05 16:21:00
tags: [Leetcode, Leetcode Contest, alg:Tree]
---

题目来源：[https://leetcode.com/problems/vertical-order-traversal-of-a-binary-tree/description/](https://leetcode.com/problems/vertical-order-traversal-of-a-binary-tree/description/)

标记难度：Medium

提交次数：1/1

代码效率：0ms

## 题意

给定一棵二叉树，定义每个结点的坐标如下：如果父结点的坐标是`(X, Y)`，则左子结点的坐标是`(X - 1, Y - 1)`，右子结点的坐标是`(X - 1, Y + 1)`。将横坐标相同的结点的值放在一个列表中，按值排序，并将这些列表按横坐标排序。

## 分析

总的来说是道简单的题，用一个map存放横坐标相同的结点的值，然后再排序即可。不过我倒是复习了一下怎么遍历C++ map……（如果不用语法糖的话）[^map]：

```cpp
// 定义迭代器
map<int, vector<pair<int, int>>>::iterator it;
// 用迭代器进行遍历
for (it = nodeMap.begin(); it != nodeMap.end(); it++) {
    // 用指针进行访问
    sort(it->second.begin(),it->second.end());
    ...
}
```

[^map]: [stackoverflow - Sort Vector in a Map c++](https://stackoverflow.com/questions/18424026/sort-vector-in-a-map-c)

## 代码

```cpp
class Solution {
private:
    map<int, vector<pair<int, int>>> nodeMap; // x -> (-y, value)
    
    void dfs(int x, int y, TreeNode* root) {
        if (root == NULL) return;
        nodeMap[x].emplace_back(-y, root->val);
        dfs(x - 1, y - 1, root->left);
        dfs(x + 1, y - 1, root->right);
    }
    
public:
    vector<vector<int>> verticalTraversal(TreeNode* root) {
        dfs(0, 0, root);
        vector<vector<int>> ans;
        map<int, vector<pair<int, int>>>::iterator it;
        for (it = nodeMap.begin(); it != nodeMap.end(); it++) {
            vector<int> cols;
            sort(it->second.begin(),it->second.end());
            for (int i = 0; i < it->second.size(); i++)
                cols.push_back(it->second[i].second);
            ans.push_back(cols);
        }
        return ans;
    }
};
```