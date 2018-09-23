---
title: Leetcode 909. Snakes and Ladders（BFS）
urlname: leetcode-909-snakes-and-ladders
toc: true
date: 2018-09-23 14:10:33
updated: 2018-09-23 14:10:33
tags: [Leetcode, Leetcode Contest, alg:Breadth-first Search]
---


题目来源：[https://leetcode.com/problems/snakes-and-ladders/description/](https://leetcode.com/problems/snakes-and-ladders/description/)

标记难度：Medium

提交次数：2/3

代码效率：

* 比赛时的BFS：32ms
* 优化过的BFS：16ms

## 题意

一个变种的BFS题。简单来说，就是可以以某种方式在棋盘上向前走1~6个格子；如果走到的格子中的值不是-1，则可以跳到该值对应的格子。

## 分析

数据量很小，所以不论怎么处理坐标问题都能做。但是在这个问题中，`visited`数组就不能那么简单地定义了。走到一个格子之后继续尝试向前走，和中途跳过这个格子显然是不一样的。所以我定义了两种`visited`数组，分别处理两种情况。不过题解里好像就没有管中途跳过这种情况了，只记录了实际走过的格子。

我中间因为没有处理好`visited`数组的问题MLE了一次，说明必须要处理这个问题，否则会出现循环跳的情况。

仔细想了一下，我觉得处理中途跳过的情形的数组确实没什么用，因为下一个要跳到的格子是确定的。以及实时计算格子对应的坐标其实也是比较简单的，不过我更喜欢不想那么多，直接全部存起来……[^solution]

[^solution]: [Leetcode 909 Solution](https://leetcode.com/problems/snakes-and-ladders/solution/)

## 代码

### 比赛时的BFS

```cpp
class Solution {
    struct Pair {
        int x, y;  // 事实上没什么必要
        int boardNum;
        int steps;

        Pair(int num, int x1, int y1, int s) {
            boardNum = num;
            x = x1;
            y = y1;
            steps = s;
        }
    };

public:
    int snakesAndLadders(vector<vector<int>>& board) {
        int N = board.size();
        map<int, pair<int, int>> numToCoordMap;
        map<pair<int, int>, int> coordToNumMap;
        bool startFrom = true;
        int cnt = 1;
        for (int i = N - 1; i >= 0; i--) {
            if (startFrom) {
                for (int j = 0; j < N; j++) {
                    numToCoordMap[cnt] = make_pair(i, j);
                    coordToNumMap[make_pair(i, j)] = cnt;
                    cnt++;
                }
            }
            else
                for (int j = N - 1; j >= 0; j--) {
                    numToCoordMap[cnt] = make_pair(i, j);
                    coordToNumMap[make_pair(i, j)] = cnt;
                    cnt++;
                }
            startFrom = !startFrom;
        }
        queue<Pair> q;
        bool visited[N*N + 1], jumped[N*N + 1];
        memset(visited, 0, sizeof(visited));
        memset(jumped, 0, sizeof(jumped));

        q.emplace(1, N-1, 0, 0);
        visited[1] = true;
        while (!q.empty()) {
            Pair p = q.front();
            q.pop();
            if (p.boardNum == N * N)
                return p.steps;
            for (int i = 1; i <= 6; i++) {
                if (p.boardNum + i > N * N) break;

                // jumped[x]: this point has been jumped through
                // visited[x]: this point has been x+1...x+6
                int next = p.boardNum + i;
                if (jumped[next]) continue;
                pair<int, int> nc = numToCoordMap[next];
                if (board[nc.first][nc.second] != -1) {
                    jumped[next] = true;

                    next = board[nc.first][nc.second];
                    if (visited[next]) continue;
                    nc = numToCoordMap[next];
                    visited[next] = true;
                }
                else
                    jumped[next] = true;
                q.emplace(next, nc.first, nc.second, p.steps + 1);
            }
        }
        return -1;
    }
};
```

### 优化过的BFS

```cpp
class Solution {
    struct Pair {
        int x, y;
        int boardNum;
        int steps;

        Pair(int num, int x1, int y1, int s) {
            boardNum = num;
            x = x1;
            y = y1;
            steps = s;
        }
    };

public:
    int snakesAndLadders(vector<vector<int>>& board) {
        int N = board.size();
        // 分别存储number->坐标和坐标->number的映射
        // 虽然可以即时计算，但我觉得这样比较直观好写……
        vector<pair<int, int>> numberToCord;
        vector<vector<int>> cordToNumber(N, vector<int>(N, 0));
        bool startFrom = true;
        numberToCord.emplace_back(-1, -1);  // board number starts from 1
        int cnt = 1;
        for (int i = N - 1; i >= 0; i--) {
            if (startFrom)
                for (int j = 0; j < N; j++) {
                    numberToCord.emplace_back(i, j);
                    cordToNumber[i][j] = cnt++;
                }
            else
                for (int j = N - 1; j >= 0; j--) {
                    numberToCord.emplace_back(i, j);
                    cordToNumber[i][j] = cnt++;
                }
            startFrom = !startFrom;
        }

        // number和坐标表示存储一种即可
        queue<pair<int, int>> q;  // (boardNum, steps)
        bool visited[N*N + 1], jumped[N*N + 1];
        memset(visited, 0, sizeof(visited));
        memset(jumped, 0, sizeof(jumped));

        q.emplace(1, 0);
        visited[1] = true;
        while (!q.empty()) {
            pair<int, int> p = q.front();
            q.pop();
            int curNum = p.first, curStep = p.second;
            if (curNum == N * N) return curStep;

            for (int i = 1; i <= 6 && curNum + i <= N*N; i++) {
                // jumped[x]: this point has been jumped through
                // visited[x]: this point has been x+1...x+6
                int nextNum = curNum + i;
                // 事实上jumped没什么用。即使不维护这个访问数组，从当前nextNum
                // 所能跳到的格子是确定的，所以下一步就可以确定是否已经访问过那个格子了。
                if (jumped[nextNum]) continue;
                int nx = numberToCord[nextNum].first, ny = numberToCord[nextNum].second;
                if (board[nx][ny] != -1) {
                    jumped[nextNum] = true;
                    nextNum = board[nx][ny];
                    if (visited[nextNum]) continue;
                    visited[nextNum] = true;
                }
                else
                    jumped[nextNum] = true;

                q.emplace(nextNum, curStep + 1);
            }
        }
        return -1;
    }
};
```
