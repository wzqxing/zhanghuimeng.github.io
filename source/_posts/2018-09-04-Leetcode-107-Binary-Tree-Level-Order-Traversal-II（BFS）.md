---
title: Leetcode 107. Binary Tree Level Order Traversal II（BFS）
urlname: leetcode-107-binary-tree-level-order-traversal-ii
toc: true
date: 2018-09-04 21:38:15
updated: 2018-09-05 00:35:00
tags: [Leetcode, alg:Breadth-firth Search, alg:Depth-first Search]
---

题目来源：[https://leetcode.com/problems/binary-tree-level-order-traversal-ii/description/](https://leetcode.com/problems/binary-tree-level-order-traversal-ii/description/)

标记难度：Easy

提交次数：3/3

代码效率：

* 递推：0.00%
* reverse：98.33%
* 两次DFS：98.33%

## 题意

返回一棵二叉树**自底向上**的层次遍历。

## 分析

这次我用上了[Leetcode 429](/post/leetcode-429-n-ary-tree-level-order-traversal)里面的那种递推法。[^recursive]令`leftLevels`表示当前结点左子树的自底向上层次遍历结果，`rightLevels`表示当前结点左子树的自底向上层次遍历结果。显然，左右两侧的深度不同，所以我们需要把它们“自顶向下”合并。以`leftLevels.size > rightLevels`的情形为例：

```
n1 = leftLevels.size, n2 = rightLevels.size

leftLevels[0]                                 -->  levels[0]
leftLevels[1]                                 -->  levels[1]
        .                                              .
        .                                              .
        .                                              .
leftLevels[n1-n2]     +    rightLevels[0]     -->  levels[n1-n2]
leftLevels[n1-n2+1]   +    rightLevels[1]     -->  levels[n1-n2+1]
        .                        .                     .
        .                        .                     .
        .                        .                     .
leftLevels[n1-1]      +    rightLevels[n2-1]  -->  levels[n1-1]
```

[^recursive]: [Easy to understand, recursive solution based on DFS (44 ms, beats 98.67%)](https://leetcode.com/problems/n-ary-tree-level-order-traversal/discuss/157521/C++-Easy-to-understand-recursive-solution-based-on-DFS-%2844-ms-beats-98.67%29)

当然，我们都知道，最简单的一种方法就是做一次普通的层次遍历，然后把整个`vector`倒转过来。不过这种解法就很平凡了。[^reverse]

[^reverse]: [C++ 4ms solution!](https://leetcode.com/problems/binary-tree-level-order-traversal-ii/discuss/35108/C++-4ms-solution!)

另一种不那么平凡的方法是，首先进行一次DFS，得到树的深度；然后再进行一次DFS，此时我们就可以得到结点在层次遍历中应有的位置，然后直接把结点插入对应的`vector`中即可。

一个很有趣的问题是，如果`vector`能够以`O(1)`时间在头部插入的话，这个问题和正向层次遍历就是正好对偶的了。对于Java用户，他们的函数返回值是`List<List<int>>`，因此可以通过`LinkedList`达到一种比较好的效果。[^java]

[^java]: [Simple Java solution with LinkedList.](https://leetcode.com/problems/binary-tree-level-order-traversal-ii/discuss/35045/Simple-Java-solution-with-LinkedList.)

## 代码

### 递推

```cpp
class Solution {
    public:
    vector<vector<int>> levelOrderBottom(TreeNode* root) {
        if (root == nullptr) return {};
        vector<vector<int>> leftLevels = levelOrderBottom(root->left);
        vector<vector<int>> rightLevels = levelOrderBottom(root->right);
        int n1 = leftLevels.size(), n2 = rightLevels.size(), n = max(n1, n2);
        vector<vector<int>> levels(n + 1, vector<int>());
        // i表示从数组末端开始的距离，也即对应的层相对root的深度
        // 这样表示是因为数组是末端对齐的
        for (int i = n; i > 0; i--) {
            if (n1 - i >= 0)
                levels[n - i].insert(levels[n - i].end(), leftLevels[n1 - i].begin(), leftLevels[n1 - i].end());
            if (n2 - i >= 0)
                levels[n - i].insert(levels[n - i].end(), rightLevels[n2 - i].begin(), rightLevels[n2 - i].end());
        }
        levels[n].push_back(root->val);
        return levels;
    }
};
```

### reverse

```cpp
class Solution {
private:
    vector<vector<int>> levels;

    void dfs(TreeNode* root, int depth) {
        if (root == NULL) return;
        if (levels.size() <= depth) levels.push_back({});
        levels[depth].push_back(root->val);
        dfs(root->left, depth+1);
        dfs(root->right, depth+1);
    }

public:
    vector<vector<int>> levelOrderBottom(TreeNode* root) {
        dfs(root, 0);
        reverse(levels.begin(), levels.end());
        return levels;
    }
};
```

### 两次DFS

```cpp
class Solution {
private:

    int maxDepth;

    void getMaxDepth(TreeNode* root, int depth) {
        if (root == nullptr) return;
        maxDepth = max(maxDepth, depth);
        getMaxDepth(root->left, depth+1);
        getMaxDepth(root->right, depth+1);
    }

    void dfs(TreeNode* root, int depth, vector<vector<int>>& levels) {
        if (root == nullptr) return;
        levels[maxDepth - depth].push_back(root->val);
        dfs(root->left, depth+1, levels);
        dfs(root->right, depth+1, levels);
    }

public:
    vector<vector<int>> levelOrderBottom(TreeNode* root) {
        maxDepth = -1;
        getMaxDepth(root, 0);
        vector<vector<int>> levels(maxDepth+1, vector<int>());
        dfs(root, 0, levels);
        return levels;
    }
};
```
