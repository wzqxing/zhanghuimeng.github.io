---
title: Leetcode 429. N-ary Tree Level Order Traversal（BFS）
urlname: leetcode-429-n-ary-tree-level-order-traversal
toc: true
date: 2018-09-04 18:49:53
updated: 2018-09-04 19:13:00
tags: [Leetcode, Breadth-firth Search, Depth-first Search]
---

题目来源：[https://leetcode.com/problems/n-ary-tree-level-order-traversal/description/](https://leetcode.com/problems/n-ary-tree-level-order-traversal/description/)

标记难度：Easy

提交次数：2/3

代码效率：

* 未优化的BFS：5.07%
* DFS：36.95%

## 题意

返回一棵多叉树的层次遍历结果。不同层要分开。

## 分析

直接BFS即可。不过我感觉我有点傻，每一层都新开了一个队列；事实上不需要，只要记录当前层的结点个数就足以判断何时这一层结束了。[^cpp]中间WA了一次，是因为对于空结点应该返回`[]`，而非`[[]]`。（我似乎经常在这一点上犯错。）

[^cpp]: [Basic C++ solution using queue. Super easy for beginners.](https://leetcode.com/problems/n-ary-tree-level-order-traversal/discuss/159086/Basic-C++-solution-using-queue.-Super-easy-for-beginners.)

另一种相对比较神奇的做法是用DFS。此时的核心问题是不同的子结点返回的层次遍历结果的合并：显然，子结点的层次遍历结果总是比当前结点正好低一层。令`r = levelOrder(root->child)`，`ret`表示当前结点的层次遍历结果。显然，`r[0]`应该与`ret[1]`合并，`r[1]`应该与`ret[2]`合并，以此类推。[^dfs]

[^dfs]: [Easy to understand, recursive solution based on DFS (44 ms, beats 98.67%)](https://leetcode.com/problems/n-ary-tree-level-order-traversal/discuss/157521/C++-Easy-to-understand-recursive-solution-based-on-DFS-%2844-ms-beats-98.67%29)

以及，从上述代码里我学习了把一个`vector`里的内容（而不是整个`vector`）插入到别的`vector`里的正确方法[^insert]：

```cpp
// 在 pos 前插入来自范围 [first, last) 的元素。
template< class InputIt >
void insert( iterator pos, InputIt first, InputIt last);
```

[^insert]: [std::vector::insert](https://zh.cppreference.com/w/cpp/container/vector/insert)

## 代码

### 未优化的BFS

```cpp
/*
// Definition for a Node.
class Node {
public:
    int val = NULL;
    vector<Node*> children;

    Node() {}

    Node(int _val, vector<Node*> _children) {
        val = _val;
        children = _children;
    }
};
*/
class Solution {
public:
    vector<vector<int>> levelOrder(Node* root) {
        if (root == NULL) return {};

        vector<vector<int>> levelOrder;
        queue<Node*> q1;
        q1.push(root);

        while (!q1.empty()) {
            vector<int> level;
            queue<Node*> q2;
            while (!q1.empty()) {
                Node* x = q1.front();
                q1.pop();
                level.push_back(x->val);
                for (Node* c: x->children)
                    q2.push(c);
            }
            q1 = q2;
            levelOrder.push_back(level);
        }

        return levelOrder;
    }
};
```

### DFS

```cpp
/*
// Definition for a Node.
class Node {
public:
    int val = NULL;
    vector<Node*> children;

    Node() {}

    Node(int _val, vector<Node*> _children) {
        val = _val;
        children = _children;
    }
};
*/
class Solution {
public:
    vector<vector<int>> levelOrder(Node* root) {
        if (root == NULL) return {};
        vector<vector<int>> level;
        level.push_back({root->val});
        for (Node* child: root->children) {
            vector<vector<int>> chLevel = levelOrder(child);
            for (int i = 0; i < chLevel.size(); i++) {
                if (i < level.size() - 1)
                    level[i + 1].insert(level[i + 1].end(), chLevel[i].begin(), chLevel[i].end());
                else
                    level.push_back(chLevel[i]);
            }
        }
        return level;
    }
};
```
