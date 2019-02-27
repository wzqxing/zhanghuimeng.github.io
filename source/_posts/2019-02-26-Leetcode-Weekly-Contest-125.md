---
title: Leetcode Weekly Contest 125总结
urlname: leetcode-weekly-contest-125-summary
toc: true
date: 2019-02-26 20:54:43
updated: 2019-02-27 14:41:00
tags: [Leetcode, Leetcode Contest]
categories: Leetcode
---

比赛太多，题解写不过来，Leetcode题解也变成每周一份了……

## [997. Find the Town Judge](https://leetcode.com/problems/find-the-town-judge/description/)

标记难度：Easy

提交次数：1/2

代码效率：252ms

### 题意

有`N`个人，他们之间有一些相互信任的关系，其中可能有一个town judge，如果有则只有一个，且满足以下要求：

* town judge不信任别人
* 除了town judge之外的其他人都信任他

请找出town judge，或者返回-1。

### 分析

水题，随便怎么做都好。

### 代码

```cpp
class Solution {
public:
    int findJudge(int N, vector<vector<int>>& trust) {
        set<int> tSet[N + 1];
        int trusting[N + 1];
        memset(trusting, 0, sizeof(trusting));
        for (int i = 0; i < trust.size(); i++) {
            tSet[trust[i][1]].insert(trust[i][0]);
            trusting[trust[i][0]]++;
        }
        int ans = -1;
        for (int i = 1; i <= N; i++) {
            if (tSet[i].size() == N - 1 && trusting[i] == 0) {
                if (ans == -1) ans = i;
                else return -1;
            }
        }
        return ans;
    }
};
```

## [998. Maximum Binary Tree II](https://leetcode.com/problems/maximum-binary-tree-ii/description/)

标记难度：Medium

提交次数：1/2

代码效率：

* 暴力：12ms
* 不暴力：8ms

### 题意

定义一种“最大二叉树”：

* 对于一个数组`A`，建一棵这样的树：
  * 找出其中最大的元素`A[i]`
  * 令当前结点的值为`A[i]`，左子结点为数组`A[:i-1]`建出的树，右子结点为数组`A[i+1:]`建出的数

现在给定一棵这样的二叉树，假设你在它对应的数组`A`后面加上一个值`val`，问得到的新树是什么样子的？

### 分析

这道题还稍微有点意思。比赛的时候，鉴于这是在比赛，所以我就直接愚蠢地把整棵树序列化成数组，把`val`加上去，然后重造了一棵树，反正`N`也不大……复杂度最差`O(N^2)`。

不那么愚蠢的方法倒是很容易考虑。首先把`val`和树根比较，如果`val`比它大，那它就是新的树根；否则继续去右子树里面查。[^sln]

[^sln]: [Leetcode 998 Solution - Java clean recursive solution](https://leetcode.com/problems/maximum-binary-tree-ii/discuss/242897/Java-clean-recursive-solution)

### 代码

#### 暴力做法

真的很暴力啊。

```cpp
class Solution {
private:
    vector<int> dfs(TreeNode* root) {
        if (root == NULL) return {};
        vector<int> l = dfs(root->left);
        l.push_back(root->val);
        vector<int> r = dfs(root->right);
        l.insert(l.end(), r.begin(), r.end());
        return l;
    }
    
    TreeNode* dfs(int l, int r, vector<int>& a) {
        if (l > r) return NULL;
        if (l == r) return new TreeNode(a[l]);
        int maxn = -1, argmax = -1;
        for (int i = l; i <= r; i++)
            if (a[i] > maxn) {
                maxn = a[i];
                argmax = i;
            }
        TreeNode* root = new TreeNode(a[argmax]);
        root->left = dfs(l, argmax - 1, a);
        root->right = dfs(argmax + 1, r, a);
        return root;
    }
    
public:
    TreeNode* insertIntoMaxTree(TreeNode* root, int val) {
        vector<int> a(dfs(root));
        a.push_back(val);
        return dfs(0, a.size() - 1, a);
    }
};
```

#### 不暴力的做法

```cpp
class Solution {
public:
    TreeNode* insertIntoMaxTree(TreeNode* root, int val) {
        if (root == NULL) return new TreeNode(val);
        if (val > root->val) {
            TreeNode* p = new TreeNode(val);
            p->left = root;
            return p;
        }
        root->right = insertIntoMaxTree(root->right, val);
        return root;
    }
};
```

## [999. Available Captures for Rook](https://leetcode.com/problems/available-captures-for-rook/description/)

标记难度：Easy

提交次数：1/1

代码效率：4ms

### 题意

在一个国际象棋盘上有一个白车，几个（或没有）白主教，几个（或没有）黑卒，剩下的地方都是空格。白车可以从上下左右中选择一个方向移动，直到遇到棋盘边沿/被主教堵住/吃到黑卒。问白车一共有可能吃到多少个黑卒？

### 分析

这题水到不能再水了。模拟走四次即可……

### 代码

```cpp
class Solution {
public:
    int numRookCaptures(vector<vector<char>>& board) {
        int rx, ry;
        int n = board.size(), m = board[0].size();
        int mx[4] = {0, 0, 1, -1};
        int my[4] = {1, -1, 0, 0};
        for (int i = 0; i < n; i++)
            for (int j = 0; j < m; j++) {
                if (board[i][j] == 'R') {
                    rx = i;
                    ry = j;
                    break;
                }
            }
        int ans = 0;
        for (int d = 0; d < 4; d++) {
            int x = rx, y = ry;
            while (0 <= x && x < n && 0 <= y && y < m) {
                if (board[x][y] == 'p') {
                    ans++;
                    break;
                }
                else if (board[x][y] == 'B') break;
                x += mx[d];
                y += my[d];
            }
        }
        return ans;
    }
};
```

## [1001. Grid Illumination](https://leetcode.com/problems/grid-illumination/description/)

标记难度：Hard

提交次数：1/1

代码效率：764ms

### 题意

嘿，1000题呢？

在一个grid上有一些灯，每个灯会照亮所有和它同行、同列、同对角线的格子。我们query这个grid上的一些格子现在是不是亮的，同时，一旦进行一次query，就把和它八连通的灯都关掉（如果有这样的灯）。问每次query的结果。

### 分析

这道题也挺水的。记录每一行、每一列、每条对角线被多少灯照亮了，然后在query的时候进行必要的删除就行。

### 代码

```cpp
class Solution {
private:
    int mx[8] = {0, 0, 1, -1, 1, 1, -1, -1};
    int my[8] = {1, -1, 0, 0, 1, -1, 1, -1};
public:
    vector<int> gridIllumination(int N, vector<vector<int>>& lamps, vector<vector<int>>& queries) {
        map<int, int> row, col, ldiag, rdiag;
        map<pair<int, int>, int> lampSet;
        for (int i = 0; i < lamps.size(); i++) {
            int x = lamps[i][0], y = lamps[i][1];
            row[x]++;
            col[y]++;
            ldiag[x - y]++;
            rdiag[x + y]++;
            lampSet[{x, y}]++;
        }
        vector<int> ans;
        for (int i = 0; i < queries.size(); i++) {
            int x = queries[i][0], y = queries[i][1];
            if (row[x] > 0 || col[y] > 0 || ldiag[x - y] > 0 || rdiag[x + y] > 0) ans.push_back(1);
            else ans.push_back(0);
            for (int j = 0; j < 8; j++) {
                int nx = x + mx[j], ny = y + my[j];
                if (nx < 0 || nx >= N || ny < 0 || ny >= N) continue;
                // 删掉可能的lamp
                if (lampSet.find({nx, ny}) != lampSet.end() && lampSet[{nx, ny}] > 0) {
                    lampSet[{nx, ny}]--;
                    row[nx]--;
                    col[ny]--;
                    ldiag[nx - ny]--;
                    rdiag[nx + ny]--;
                }
            }
        }
        return ans;
    }
};
```
