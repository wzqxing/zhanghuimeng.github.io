---
title: Leetcode 934. Shortest Bridge（BFS）
urlname: leetcode-934-shortest-bridge
toc: true
date: 2018-11-04 16:02:37
updated: 2018-11-04 23:05:00
tags: [Leetcode, Leetcode Contest, alg:Breadth-first Search, alg:Depth-first Search]
---

题目来源：[https://leetcode.com/problems/number-of-recent-calls/description/](https://leetcode.com/problems/number-of-recent-calls/description/)

标记难度：Medium

提交次数：3/6

代码效率：

* treeSet+Dijstra：140ms
* 优先队列+Dijstra：120ms
* BFS：24ms

## 题意

在一个四连通图上有两个互不连通的子图，求这两个子图之间的最短距离。

## 分析

刚看到这道题的时候我很晕，不知道该怎么做。遂上网谷歌一下……找到了正确的做法之后，我却把`set`给写挂了。事实证明，`std::set`中判断相等的方式是`!(a < b) && !(b < a)`而非`!(a == b)`。这一点是值得注意的。[^stlset]

[^stlset]: [stackoverflow - std::set with user defined type, how to ensure no duplicates](https://stackoverflow.com/questions/1114856/stdset-with-user-defined-type-how-to-ensure-no-duplicates)

---

单就一般的图来说，这道题也不算很难：把子图A的所有结点都作为Dijstra算法的起始结点，然后在子图B的所有结点中取距离最短的。如果没有负权边，则直接在找到第一个子图B结点时结束算法即可。

然后这道题并不是一般的图，而是四连通的方格图，所以BFS就可以了。BFS的常数果然还是小啊……

## 代码

用Dijstra的代码太过智障所以就不贴了……

```cpp
class Solution {
private:
    int N;
    int color[105][105];
    int dist[105][105];
    int mx[4] = {-1, 1, 0, 0};
    int my[4] = {0, 0, -1, 1};

    void dfs(int curx, int cury, int c, vector<vector<int>>& A) {
        for (int i = 0; i < 4; i++) {
            int x = curx + mx[i];
            int y = cury + my[i];
            if (x < 0 || x >= N || y < 0 || y >= N || A[x][y] != 1 || color[x][y] != -1) continue;
            color[x][y] = c;
            dfs(x, y, c, A);
        }
    }

public:
    int shortestBridge(vector<vector<int>>& A) {
        N = A.size();
        memset(color, -1, sizeof(color));
        int colorCnt = 0;
        for (int i = 0; i < N; i++)
            for (int j = 0; j < N; j++) {
                if (A[i][j] == 1 && color[i][j] == -1) {
                    color[i][j] = colorCnt;
                    dfs(i, j, colorCnt, A);
                    colorCnt++;
                }
            }

        queue<pair<int, int>> q;
        for (int i = 0; i < N; i++)
            for (int j = 0; j < N; j++) {
                if (color[i][j] == 0) {
                    dist[i][j] = 0;
                    q.push(make_pair(i, j));
                }
                else
                    dist[i][j] = 1e9;
            }
        while (!q.empty()) {
            int x = q.front().first, y = q.front().second;
            q.pop();
            if (color[x][y] == 1) return dist[x][y] - 1;
            for (int i = 0; i < 4; i++) {
                int nx = x + mx[i], ny = y + my[i];
                if (nx < 0 || nx >= N || ny < 0 || ny >= N || dist[nx][ny] <= dist[x][y] +1) continue;
                dist[nx][ny] = dist[x][y] + 1;
                q.push(make_pair(nx, ny));
            }
        }
        return -1;
    }
};
```
