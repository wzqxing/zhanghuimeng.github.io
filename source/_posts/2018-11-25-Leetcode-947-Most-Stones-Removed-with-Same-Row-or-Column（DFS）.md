---
title: Leetcode 947. Most Stones Removed with Same Row or Column（DFS）
urlname: leetcode-947-most-stones-removed-with-same-row-or-column
toc: true
date: 2018-11-25 15:48:42
updated: 2018-11-25 17:01:00
tags: [Leetcode, Leetcode Contest, alg:Depth-first Search, alg:Union-find Forest]
---

题目来源：[https://leetcode.com/problems/most-stones-removed-with-same-row-or-column/description/](https://leetcode.com/problems/most-stones-removed-with-same-row-or-column/description/)

标记难度：Medium

提交次数：1/1

代码效率：40ms

## 题意

给定一个2D平面和一些位于自然数坐标上的小石子（坐标不会重复）；如果某个石子和其他石子位于同一行同一列上，则可以把它移除；问最多能移走多少个小石子。

## 分析

比赛的时候我想了很久也没想出来……（然后就去吃饭了）。我的确想到了把在同一行同一列上的石子都连起来然后找连通子图数量这种做法，但我竟然觉得它有反例……（现在我已经知道我当初是怎么想错的了）

---

第一个问题是怎么证明一个大的连通子图一定能reduce成一颗石子。我感觉这并不是一件显然的事情。

（嗯，确实没有**那么**显然……）

一种方法是，构造一颗这个子树的生成树。（从一个无向连通图能够生成一颗生成树这件事应该够显然的了）然后不断移除这棵树的叶子结点，直到最后只剩下根结点。[^solution]

当然，事实上不需要真的把这些都构造出来，只要知道就可以了。

然后我感觉直接DFS就可以了……并查集当然也可以。

[^solution]: [Leetcode Official Solution for 947. Most Stones Removed with Same Row or Column](https://leetcode.com/articles/most-stones-removed-with-same-row-or-column/)

## 代码

懒得写并查集了。

```cpp
class Solution {
private:
    int n;
    vector<pair<int, int>> stoneCoors;
    map<int, vector<int>> rows, cols;
    set<int> rowsVisited, colsVisited;
    
    void dfs(int x, int r, int c) {
        // cout << x <<' ' << r << ' ' << c << endl;
        if (rowsVisited.find(r) == rowsVisited.end()) {
            rowsVisited.insert(r);
            for (int y: rows[r])
                if (y != x && colsVisited.find(stoneCoors[y].second) == colsVisited.end())
                    dfs(y, stoneCoors[y].first, stoneCoors[y].second); 
        }
        if (colsVisited.find(c) == colsVisited.end()) {
            colsVisited.insert(c);
            for (int y: cols[c])
                if (y != x && rowsVisited.find(stoneCoors[y].first) == rowsVisited.end())
                    dfs(y, stoneCoors[y].first, stoneCoors[y].second); 
        }
    }
    
public:
    int removeStones(vector<vector<int>>& stones) {
        n = stones.size();
        for (int i = 0; i < n; i++) {
            stoneCoors.emplace_back(stones[i][0], stones[i][1]);
        }
        sort(stoneCoors.begin(), stoneCoors.end());
        for (int i = 0; i < n; i++) {
            rows[stoneCoors[i].first].push_back(i);
            cols[stoneCoors[i].second].push_back(i);
        }
        
        int ans = 0;
        for (int i = 0; i < n; i++) {
            if (rowsVisited.find(stoneCoors[i].first) == rowsVisited.end() || 
               colsVisited.find(stoneCoors[i].second) == colsVisited.end()) {
                // cout << i << endl;
                ans++;
                dfs(i, stoneCoors[i].first, stoneCoors[i].second);
            }
        }
        
        return n - ans;
    }
};
```