---
title: Leetcode 980. Unique Paths III（状态DP）
urlname: leetcode-980-unique-paths-iii
toc: true
date: 2019-01-21 11:04:51
updated: 2019-01-21 21:16:00
tags: [Leetcode, Leetcode Contest, alg: Dynamic Programming, alg:Depth-first Search]
---

题目来源：[https://leetcode.com/problems/unique-paths-iii/description/](https://leetcode.com/problems/unique-paths-iii/description/)

标记难度：Hard

提交次数：2/8

代码效率：

* DP：16ms
* DFS：4ms

## 题意

有一个二维的方格图，每个格子里有一个值：

* 1表示起始格子
* 2表示终止格子
* 0表示其他可以走的格子
* -1表示障碍物

问从起始格子走到终止格子，且把其他可以走的格子都不重不漏地走完一遍的方法总数。

## 分析

第一个想法就是状态压缩DP啦……但是这次我写好之后却发现`f[20][1 << 20]`这么大的数组会MLE！！！虽然Leetcode经常只会报RE就是了……然后我稍微试了一下，发现静态数组定义在类私有变量里一定会MLE，定义在函数里则不会MLE，但是也非常靠近MLE的边缘（再来几个栈帧大概也就爆了）；如果动态分配数组，Run Code时可以正常运行，但提交时也会MLE。交了八次主要就是在试验这些，最后也没试出来Memory Limit是多少。

总之Leetcode这一点非常烦。结论：用递归写法，用map当数组。

除此之外，这道题还有一点需要注意，就是障碍格子的处理。其他就没有什么了。复杂度是`O(N * 2^N)`，其中`N`是格子总数。

我之前的非递归写法需要保证已访问格子数少的状态先被计算出来，所以会多出来一个`N`的常数项，不过因为`N`一般很小，所以也不是什么大问题。不过递归写法就不需要专门操心格子的计算顺序问题……而且还可以省空间，唉。

另一种方法是DFS……或者说直接暴力。我本来以为这种方法肯定不能过的，结果发现在适当的剪枝下完全可以。但是这个复杂度我就不会估计了。

## 代码

### DP

```cpp
class Solution {
private:
    int n, m, N;
    map<pair<int, int>, int> f;
    int obsMask;
    int obsCnt;
    int start, end;
    int mx[4] = {0, 0, 1, -1}, my[4] = {1, -1, 0, 0};
    
    // state里是包括自己的
    int calc(int x, int y, int state, vector<vector<int>>& grid) {
        int g = x * m + y;
        pair<int, int> p(g, state);
        if (f.find(p) != f.end()) return f[p];
        // 障碍代表的格子一直都处于visited状态
        if (__builtin_popcount(state) == obsCnt + 1) {
            if (state == (obsMask | (1 << start)))
                f[p] = 1;
            else
                f[p] = 0;
            return f[p];
        }
        int f1 = 0;
        for (int i = 0; i < 4; i++) {
            int nx = x + mx[i], ny = y + my[i];
            if (nx < 0 || nx >= n || ny < 0 || ny >= m) continue;
            if (grid[nx][ny] == -1) continue;
            int ng = nx * m + ny;
            if (!(state & (1 << ng))) continue;
            f1 += calc(nx, ny, state ^ (1 << g), grid);
        }
        f[p] = f1;
        return f1;
    }
    
public:
    int uniquePathsIII(vector<vector<int>>& grid) {
        n = grid.size();
        m = grid[0].size();
        N = m * n;
        obsCnt = 0;
        for (int i = 0; i < N; i++) {
            int x = i / m, y = i % m;
            if (grid[x][y] == 2) end = i;
            else if (grid[x][y] == 1) start = i;
            else if (grid[x][y] == -1) {
                obsMask |= 1 << i;
                obsCnt++;
            }
        }
        return calc(end / m, end % m, (1 << N) - 1, grid);
    }
};
```

### DFS

```cpp
class Solution {
private:
    bool visited[20][20];
    int mx[4] = {0, 0, 1, -1}, my[4] = {1, -1, 0, 0};
    int n, m;
    int tot;
    int ans;
    
    void dfs(int x, int y, int depth, vector<vector<int>>& grid) {
        if (depth == tot - 1) {
            if (grid[x][y] == 2) ans++;
            return;
        }
        if (grid[x][y] == 2) return;  // 一个小的剪枝
        for (int i = 0; i < 4; i++) {
            int nx = x + mx[i], ny = y + my[i];
            if (nx < 0 || nx >= n || ny < 0 || ny >= m) continue;
            if (visited[nx][ny] || grid[nx][ny] == -1) continue;
            visited[nx][ny] = true;
            dfs(nx, ny, depth + 1, grid);
            visited[nx][ny] = false;
        }
    }
    
public:
    int uniquePathsIII(vector<vector<int>>& grid) {
        n = grid.size();
        m = grid[0].size();
        tot = 0;
        int startx, starty;
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                if (grid[i][j] != -1) tot++;
                if (grid[i][j] == 1) startx = i, starty = j;
            }
        }
        ans = 0;
        memset(visited, 0, sizeof(visited));
        visited[startx][starty] = true;
        dfs(startx, starty, 0, grid);
        return ans;
    }
};
```