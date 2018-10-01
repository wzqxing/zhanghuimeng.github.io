---
title: 2018-09-30-Leetcode 913. Cat and Mouse（极大极小问题）
urlname: 2018-09-30-Leetcode 913. Cat and Mouse（极大极小问题）
toc: true
date: 2018-09-30 23:20:43
updated: 2018-10-01 21:45:00
tags: [Leetcode, Leetcode Contest, alg:Minimax]
---

题目来源：[https://leetcode.com/problems/cat-and-mouse/description/](https://leetcode.com/problems/cat-and-mouse/description/)

标记难度：Hard

提交次数：1/4

代码效率：12ms

## 题意

有一张无向图，包含最多50个结点。有两个玩家（Mouse和Cat）在图上，Mouse的起始位置是1，Cat的起始位置是2，0处有一个洞，Cat不能移动到0。Mouse和Cat在图上轮流移动，每次必须移动到与当前结点相邻的一个结点。

游戏有三种结束的可能：

* Cat和Mouse进入同一结点，则Cat胜利
* Mouse进入0结点，则Mouse胜利
* Cat和Mouse的位置和回合发生了重复，则平局

问：如果Cat和Mouse都以最优策略行动，最后的结果是什么？

## 分析

这大概是我目前为止在Leetcode Contest里见过的最难的一道题……

---

很显然，这道题是一个极大极小问题，可以把状态写成`(mPos, cPos, turn)`，然后建立一张状态之间的转移图，并给状态染色（胜、负、平局）。但问题是这个图里可能有圈（所以显然这不是一棵树），而且也因此存在平局的问题。题解里给出的解决方法类似于反过来的拓扑排序：

* 首先将那些颜色能够确定的状态（同时出度=0）入队，比如`mPos = 0`和`mPos == cPos`的那些
* 从队里取出一个状态，尝试对它的所有祖先状态进行染色。对于某个祖先状态：
  * 如果已经被染色，则跳出
  * 如果能够立即染色（比如祖先状态的`turn == MOUSE`且当前状态的颜色为`MOUSE`），则直接进行染色，并将该状态入队
  * 如果不能立即染色，则记录祖先状态的出度-1；如果发现祖先状态的出度为0，说明该状态下的先手必然失败，也可以进行相应的染色并将该状态入队
* 最后输出`(1, 2, MOUSE)`的颜色。没有被染过色说明是平局。

以上的内容只是**类似**于拓扑排序，并不**就是**拓扑排序，因为我们可以在尚未访问完一个状态的所有子状态（即出度还不是0）时把它取出来，继续更新其他的结点。（我似乎因为这个原因踩了一些坑）[^solution]

[^solution]: [Leetcode 913 Solution](https://leetcode.com/problems/cat-and-mouse/solution/)

但是，我觉得没有被染过色说明是平局不太好理解。虽然理论上是这样的。

除此之外，还有DP的方法，但是我暂时还不太理解。

## 代码

```cpp
class Solution {
    struct Tuple {
        int mPos, cPos;
        int turn;

        Tuple(int m, int c, int t) {
            mPos = m;
            cPos = c;
            turn = t;
        }
    };

public:
    int catMouseGame(vector<vector<int>>& graph) {
        const int MOUSE = 1, CAT = 2;

        int N = graph.size() - 1;
        int states[55][55][3];  // 1: mouse, 2: cat
        memset(states, 0, sizeof(states));

        queue<Tuple> q;
        int outDegree[55][55][3];
        memset(outDegree, 0, sizeof(outDegree));
        // put the definite status to queue
        for (int i = 0; i <= N; i++) {
            for (int j = 0; j <= N; j++) {
                if (j == 0) continue;
                if (i == 0) {
                    states[i][j][MOUSE] = states[i][j][CAT] = MOUSE;
                    q.emplace(i, j, MOUSE);
                    q.emplace(i, j, CAT);
                    continue;
                }
                if (i == j) {
                    states[i][j][MOUSE] = states[i][j][CAT] = CAT;
                    q.emplace(i, j, MOUSE);
                    q.emplace(i, j, CAT);
                    continue;
                }
                outDegree[i][j][MOUSE] = graph[i].size();
                // Note: Cat cannot go to 0
                for (int x: graph[j])
                    if (x != 0)
                        outDegree[i][j][CAT]++;
            }
        }

        while (!q.empty()) {
            Tuple tuple = q.front();
            q.pop();

            int mPos = tuple.mPos, cPos = tuple.cPos, turn = tuple.turn;
            if (states[mPos][cPos][turn] == 0) continue;

            // find its parent node
            if (turn == MOUSE) {
                for (int x: graph[cPos]) {
                    if (x == 0) continue;
                    // Need this line
                    if (states[mPos][x][CAT] != 0) continue;
                    // When its color is defined, immediately add it to the queue
                    if (states[mPos][cPos][turn] == CAT) {
                        states[mPos][x][CAT] = CAT;
                        q.emplace(mPos, x, CAT);
                    }
                    else {
                        outDegree[mPos][x][CAT]--;
                        if (outDegree[mPos][x][CAT] == 0) {
                            states[mPos][x][CAT] = MOUSE;
                            q.emplace(mPos, x, CAT);
                        }
                    }
                }
            }
            else {
                for (int x: graph[mPos]) {
                    if (states[x][cPos][MOUSE] != 0) continue;
                    if (states[mPos][cPos][turn] == MOUSE) {
                        states[x][cPos][MOUSE] = MOUSE;
                        q.emplace(x, cPos, MOUSE);
                    }
                    else {
                        outDegree[x][cPos][MOUSE]--;
                        if (outDegree[x][cPos][MOUSE] == 0) {
                            states[x][cPos][MOUSE] = CAT;
                            q.emplace(x, cPos, MOUSE);
                        }
                    }
                }
            }
        }

        return states[1][2][MOUSE];
    }
};
```
