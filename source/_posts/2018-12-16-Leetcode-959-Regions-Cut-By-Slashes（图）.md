---
title: Leetcode 959. Regions Cut By Slashes（图）
urlname: leetcode-959-regions-cut-by-slashes
toc: true
date: 2018-12-16 21:50:08
updated: 2018-12-16 22:22:00
tags: [Leetcode, Leetcode Contest, alg:Depth-first Search, alg:Union-find Forest, alg:Graph]
---

题目来源：[https://leetcode.com/problems/regions-cut-by-slashes/description/](https://leetcode.com/problems/regions-cut-by-slashes/description/)

标记难度：Medium

提交次数：2/5

代码效率：

* 并查集：12ms
* DFS：28ms

## 题意

用字符画的形式给定一个N*N的格子里的分割线，如：

```
\/
/\
```

问图中共有多少个连通区域。

## 分析

比赛的时候我就想到了把每个格子分成四个三角形的方法。但是我当时尝试在类里开一个3600*3600的静态数组，所以RE了。不过它只会RE而不会报MLE，这令人很难调试……反正直接用邻接矩阵肯定是不行的。

---

一种方法是直接绕开邻接矩阵和邻接表，用并查集来解决。

另一种看起来很神奇的方法[^vortrubac]是，直接用upscale的思路把每个格子分成9块，然后令线的方格为障碍物，其他方格为可通过，这样也可以做DFS（或者说floodfill）。事实上这种建模的思路是隐式地建了每个小方格和周围的4个方格之间的图。

[^vortrubac]: [vortrubac's Solution for Leetcode 959 - C++ with picture, DFS on upscaled grid](https://leetcode.com/problems/regions-cut-by-slashes/discuss/205674/C++-with-picture-DFS-on-upscaled-grid)

最后，当然用邻接表+DFS也可以啦。

## 代码

### 并查集

```cpp
class Solution {
private:
    int _fa[3600];
    int N, M;
    void init() {
        for (int i = 0; i < M; i++)
            _fa[i] = i;
    }
    
    int fa(int x) {
        return x == _fa[x] ? x : _fa[x] = fa(_fa[x]);
    }
    
    void merge(int x, int y) {
        x = fa(x);
        y = fa(y);
        _fa[x] = y;
    }
    
public:
    int regionsBySlashes(vector<string>& grid) {
        // 4N^2 small triangles
        N = grid.size();
        M = 4*N*N;
        init();
        // initial merge
        for (int i = 0; i < N; i++)
            for (int j = 0; j < N; j++) {
                int n = 4*(i*N + j);
                if (i < N - 1) {
                    int n1 = 4*((i+1)*N + j);
                    merge(n+2, n1);
                }
                if (j < N - 1) {
                    int n1 = 4*(i*N + j+1);
                    merge(n+1, n1+3);
                }
            }
        for (int i = 0; i < N; i++) {
            for (int j = 0; j < N; j++) {
                int n = 4*(i*N + j);
                if (grid[i][j] != '/') {
                    merge(n, n+1);
                    merge(n+2, n+3);
                }
                if (grid[i][j] != '\\') {
                    merge(n, n+3);
                    merge(n+1, n+2);
                }
            }
        }
        int ans = 0;
        for (int i = 0; i < M; i++) {
            if (fa(i) == i)
                ans++;
        }
        return ans;
    }
};
```

### DFS

```cpp
class Solution {
private:
    int N;
    vector<int> G[3600];
    void connect(int x, int y) {
        G[x].push_back(y);
        G[y].push_back(x);
    }
    bool visited[3600];
    
    void dfs(int x) {
        for (auto const& y: G[x])
            if (!visited[y]) {
                visited[y] = true;
                dfs(y);
            }
    }
    
public:
    int regionsBySlashes(vector<string>& grid) {
        N = grid.size();
        // initial links
        for (int i = 0; i < N; i++)
            for (int j = 0; j < N; j++) {
                int n = 4*(i*N + j);
                if (i < N - 1) {
                    int n1 = 4*((i+1)*N + j);
                    connect(n+1, n1+3);
                }
                if (j < N - 1) {
                    int n1 = 4*(i*N + j+1);
                    connect(n+2, n1);
                }
            }
        // grid picture
        for (int i = 0; i < N; i++) {
            for (int j = 0; j < N; j++) {
                int n = 4*(i*N + j);
                if (grid[i][j] != '\\') {
                    connect(n, n+3);
                    connect(n+1, n+2);
                }
                if (grid[i][j] != '/') {
                    connect(n, n+1);
                    connect(n+2, n+3);
                }
            }
        }
        
        int ans = 0;
        memset(visited, 0, sizeof(visited));
        for (int i = 0; i < 4*N*N; i++) {
            if (!visited[i]) {
                visited[i] = true;
                ans++;
                dfs(i);
            }
        }
        return ans;
    }
};
```