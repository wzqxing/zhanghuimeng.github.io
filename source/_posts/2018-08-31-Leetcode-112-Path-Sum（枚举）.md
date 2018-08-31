---
title: Leetcode 112. Path Sum（枚举）
urlname: leetcode-112-path-sum
toc: true
date: 2018-08-31 14:33:54
updated: 2018-08-31 14:46:00
tags: [Leetcode, Tree, DFS]
---

题目来源：[https://leetcode.com/problems/path-sum/description/](https://leetcode.com/problems/path-sum/description/)

标记难度：Easy

提交次数：1/4

代码效率：5.68%

## 题意

给定一棵二叉树和一个数`sum`，问这棵二叉树上是否存在一条从根到叶结点的路径，这条路径上的结点之和为`sum`。

## 分析

这道题的思路很简单：既然对这棵树没有什么其他的规定，那就只能枚举所有路径了。通过DFS进行搜索，复杂度应该是`O(n)`。

结果我竟然错了3次……真叫人头大。

* 第一次错是因为对`[] 0`的输入给出了`true`的返回值，特判没做好。当然这份代码本来也不对。（就这样还过了101个测试点。）于是特判了一下。
* 第二次错是因为根本没有判断当前结点是否为叶结点，就用求和结果和`sum`相比较了。于是判断了一下。
* 第三次错是因为改了判断方式后求和结果搞错了，忘了把根节点的值加上去了。改完后终于对了……

这可真叫人头大。这的确是道水题，但我可能把题目想得太复杂了，代码也不够简洁——答案里有些C++代码只有3行[^cpp]（虽然可能为了简洁牺牲了一些效率，比如不能剪枝；不过把问题转化一下之后写法是变得简单了）。

[^cpp]: [3 lines of c++ solution](https://leetcode.com/problems/path-sum/discuss/36367/3-lines-of-c++-solution)

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
    bool found;
    int sum;

    void dfs(TreeNode* root, int curSum) {
        if (found)
            return;
        if (root == NULL)
            return;
        // cout << root << ' ' << root->left << ' ' << root->right <<' ' << curSum << endl;
        if (root->left == NULL && root->right == NULL) {
            if (curSum + root->val == sum)
                found = true;
            return;
        }
        dfs(root->left, curSum + root->val);
        dfs(root->right, curSum + root->val);
    }

public:
    bool hasPathSum(TreeNode* root, int sum) {
        found = false;
        this->sum = sum;
        dfs(root, 0);
        return found;
    }
};
```
