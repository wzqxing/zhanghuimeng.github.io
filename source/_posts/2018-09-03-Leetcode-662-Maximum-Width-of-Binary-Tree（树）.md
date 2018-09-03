---
title: Leetcode 662. Maximum Width of Binary Tree（树）
urlname: leetcode-662-maximum-width-of-binary-tree
toc: true
date: 2018-09-03 14:47:02
updated: 2018-09-03 15:28:00
tags: [Leetcode, alg:Tree]
---

题目来源：[https://leetcode.com/problems/maximum-width-of-binary-tree/description/](https://leetcode.com/problems/maximum-width-of-binary-tree/description/)

标记难度：Medium

提交次数：2/3

代码效率：100.00%

## 题意

计算二叉树的最大宽度。“宽度”指的是，把树中所有空的位置都用`null`填上，填成一棵完全二叉树之后，树上同层的两个非空结点之间的最大距离。题目保证答案在32位有符号整数范围内。

## 分析

很显然，这个要求和二叉树的数组表示很相似，所以可以为每个结点计算一个`pos`：根结点的`pos = 1`，每个结点的左子结点的`left.pos = pos*2 + 1`，右子结点的`right.pos = pos*2 + 2`。然后用DFS或BFS统计每一层的最小和最大`pos`，记录最大的宽度。

这个方法看起来很好，但是有一点微小的问题，就是数据范围。看到题目里的保证之后，我就自以为聪明地把`pos`都换成`long long int`类型的了，觉得这样就能保证结果不溢出了。结果仍然有一个点没过，因为这个测试点长这个样子：

```
[0,null,0,null,0,null,0,null,0,null,0,null,0,null,0,null,0,null,0,null,0,null,0,null,0,null,0,null,0,null,0,null,0,...
```

也就是一条很长的不断向右延伸的链。在这种情况下，`pos`连`long long int`都会溢出，变成负数。

后来我修改了一下代码的写法，过了这个测试点，但我感觉还可能会有更极端的情形出现。一种更好的做法可能是重设每一层的`pos`。

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
    struct Node {
        TreeNode* treeNode;
        int depth;
        long long int pos;

        Node(TreeNode* x, int d, int p): treeNode(x), depth(d), pos(p) { }
    };

public:
    int widthOfBinaryTree(TreeNode* root) {
        if (root == NULL)
            return 0;
        queue<Node> q;
        q.push(Node(root, 0, 1));

        int curDepth = 0;
        long long int ans = -1;
        while (!q.empty()) {
            long long int left = -1, right = -1;
            while (!q.empty() && q.front().depth == curDepth) {
                Node node = q.front();
                q.pop();
                if (left == -1)
                    left = node.pos;
                right = node.pos;
                // 重设本层的pos
                // node.pos -= left - 1;
                if (node.treeNode->left != NULL)
                    q.push(Node(node.treeNode->left, curDepth+1, node.pos*2+1));
                if (node.treeNode->right != NULL)
                    q.push(Node(node.treeNode->right, curDepth+1, node.pos*2+2));
            }
            // 居然爆long long了
            // cout << left << ' ' << right << endl;
            ans = max(ans, right - left + 1);
            curDepth++;
        }
        return ans;
    }
};
```
