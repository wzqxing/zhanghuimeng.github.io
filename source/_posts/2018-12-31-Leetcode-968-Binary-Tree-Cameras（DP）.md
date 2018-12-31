---
title: Leetcode 968. Binary Tree Cameras（DP）
urlname: leetcode-968-binary-tree-cameras
toc: true
date: 2018-12-31 15:47:19
updated: 2018-12-31 17:00:00
tags: [Leetcode, Leetcode Contest, alg:Greedy, alg:Dynamic Programming]
---

题目来源：[https://leetcode.com/problems/numbers-with-same-consecutive-differences/description/](https://leetcode.com/problems/numbers-with-same-consecutive-differences/description/)

标记难度：Hard

提交次数：2/5

代码效率：

* DP：12ms（100.00%）
* 贪心：12ms（100.00%）

## 题意

给定一棵二叉树，要求在上面一些结点上放照相机（照相机可以覆盖结点自己、它的父结点和子结点），使得所有的结点都被覆盖，且被放照相机的结点总数最少。

## 分析

比赛的时候我认为这是一道树状DP，但是太困了写不动了。但是我还是相信自己是对的，并且在比赛之后成功写了出来，并且写对了。结果看了看题解，大家都在用贪心，有种恍如隔世的感觉……

### 贪心法

令状态`0`表示该结点还没有被cover，`1`表示该结点上有一个照相机，`2`表示该结点上没有照相机但是已经被cover了。（前提：这个结点对应的子树除了它自己以外已经全都被cover了。）

令叶结点的状态为`0`；对于一个结点，如果它的至少一个子结点状态为`0`，则它的状态为`1`；如果它的子结点状态均不为`0`，且至少一个子结点状态为`1`，则它的状态为`2`；否则它的状态为`0`。状态为`1`时照相机计数+1。

注意边界条件（如果根节点的状态为`0`，则需要多加一个照相机）。[^lee215]

[^lee215]: [lee215's solution for Leetcode 968 - \[Java/C++/Python\] Greedy DFS](https://leetcode.com/problems/binary-tree-cameras/discuss/211180/JavaC++Python-Greedy-DFS)

我实在想不通怎么证明这个解法的正确性（虽然我觉得它确实很有道理）。

<!--不行，我觉得必须要来证明一下这个算法的正确性。对于每个结点`node`，令`f[node][3]`表示上述三种状态分别需要的标记数量。则：

* `f[node][0] = f[node.left][2] + f[node.right][2]`（因为`node`自己还没有被cover，所以它的子结点上不能有照相机，但是必须已经被cover了）
* `f[node][1] = min(f[node.left][0], f[node.left][1], f[node].left[2]) + min(f[node.right][0], f[node.right][1], f[node.right][2]) + 1`（从定义上来说，如果`node`上有了一个照相机，左右是什么状态都没关系）
* `f[node][2] = min(f[node.left][1] + min(f[node.right][1], f[node.right][2]), f[node.right][1] + min(f[node.left][1], f[node.left][2]))`（为了让`node`被cover，左边或右边必有一台照相机）

对于一个叶结点，显然有`f[leaf][0] = 0, f[leaf][1] = 1, f[leaf][2] = inf`。则对于一个子结点都是叶结点的结点来说，显然有`f[node][0] = inf, f[node][1] = 1, f[node][2] = 2`。

假定每个结点都有一个最优状态`i`。-->

### DP

我这个方法听起来就比较冗杂（但是我认为很好理解）。令`f[node]`表示覆盖完`node`对应的子树所需的最少照相机数，则

```cpp
f[node] = min(
    // node上放照相机，node.left和node.right上可以放，也可以不放
    min(f[node.left], f[node.left.left] + f[node.left.right]) + min(f[node.right], f[node.right.left] + f[node.right.right]), 
    // node.left上放照相机，node.left.left和node.left.right上可以放，也可以不放
    min(f[node.left.left], f[node.left.left.left] + f[node.left.left.right]) + min(f[node.left.right], f[node.left.right.left] + f[node.left.right.right]) + 1 + f[node.right],
    // node.right上放照相机，node.right.left和node.right.right上可以放，也可以不放
    min(f[node.right.left], f[node.right.left.left] + f[node.right.left.right]) + min(f[node.right.right], f[node.right.right.left] + f[node.right.right.right]) + 1 + f[node.left]
);
```

显然为了覆盖到子树的根结点，只需要考虑它自己上面放照相机，它的左子结点放照相机和右子结点放照相机三种情况就够了。

## 代码

### 贪心法

```cpp
class Solution {
private:
    int ans = 0;
    // 0 = not covered
    // 1 = has camera
    // 2 = covered by camera (no camera)
    int dfs(TreeNode* root) {
        if (root->left == NULL && root->right == NULL) return 0;
        int needCamera = 0;
        int covered = 0;
        if (root->left != NULL) {
            int state = dfs(root->left);
            if (state == 0) needCamera = 1;
            if (state == 1) covered = 1;
        }
        if (root->right != NULL) {
            int state = dfs(root->right);
            if (state == 0) needCamera = 1;
            if (state == 1) covered = 1;
        }
        if (needCamera) {
            ans++;
            return 1;
        }
        if (covered) return 2;
        return 0;
    }
    
public:
    int minCameraCover(TreeNode* root) {
        int state = dfs(root);
        if (state == 0) ans++;
        return ans;
    }
};
```

### DP法

```cpp
class Solution {
private:
    
    int getF(TreeNode* root) {
        return root == NULL ? 0 : root->val;
    }
    
    void dfs(TreeNode* root) {
        if (root == NULL) return;
        dfs(root->left);
        dfs(root->right);
        if (root->left == NULL && root->right == NULL) {
            root->val = 1;
            return;
        }
        root->val = 1e9;
        // put on root
        TreeNode* l = root->left;
        TreeNode* r = root->right;
        TreeNode* ll = l == NULL ? NULL : l->left;
        TreeNode* lr = l == NULL ? NULL : l->right;
        TreeNode* rl = r == NULL ? NULL : r->left;
        TreeNode* rr = r == NULL ? NULL : r->right;
        root->val = min(min(getF(l), getF(ll) + getF(lr)) + min(getF(r), getF(rl) + getF(rr)) + 1, root->val);
        // 也要考虑到不完全覆盖的情况！
        // put on left
        TreeNode* lll = ll == NULL ? NULL : ll->left;
        TreeNode *llr = ll == NULL ? NULL : ll->right;
        TreeNode* lrl = lr == NULL ? NULL : lr->left;
        TreeNode *lrr = lr == NULL ? NULL : lr->right;
        root->val = min(min(getF(ll), getF(lll) + getF(llr)) + min(getF(lr), getF(lrl) + getF(lrr)) + 1 + getF(r), root->val);
        // put on right
        TreeNode* rll = rl == NULL ? NULL : rl->left;
        TreeNode* rlr = rl == NULL ? NULL : rl->right;
        TreeNode* rrl = rr == NULL ? NULL : rr->left;
        TreeNode* rrr = rr == NULL ? NULL : rr->right;
        root->val = min(min(getF(rl), getF(rll) + getF(rlr)) + min(getF(rr), getF(rrl) + getF(rrr)) + 1 + getF(l), root->val);
    }
    
public:
    int minCameraCover(TreeNode* root) {
        dfs(root);
        return root->val;
    }
};
```