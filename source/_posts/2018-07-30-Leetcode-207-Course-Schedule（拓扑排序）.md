---
title: Leetcode 207. Course Schedule（拓扑排序）
urlname: leetcode-207-course-schedule
toc: true
date: 2018-07-30 22:31:58
updated: 2018-07-30 22:42:00
tags: [Leetcode]
---

题目来源：[https://leetcode.com/problems/course-schedule/description/](https://leetcode.com/problems/course-schedule/description/)

标记难度：Medium

提交次数：1/2

代码效率：99.48%

## 题意

给定一些课程和每门课对应的若干先修课程，要求必须修完对应先修课程才能修这门课，问是否存在一种修课顺序，能够修完所有的课。

## 分析

这本质上就是一个在有向图中寻找拓扑序的问题，直接套用模型就可以了。

P.S. 我们都知道拓扑排序一般的做法是记录每个结点的入度，然后在删除结点的同时更新其他点的入度。也就是说，我们用一个数字统计量来代替了集合，而这样做是十分合理的，因为在这一问题中，只有入度的**累计**才有意义，其具体内容没有意义。这很有趣。

## 代码

```cpp
class Solution {
public:
    // 我感觉这个只是问一个有向图里有没有圈。
    // 所以感觉简单的拓扑排序就可以了。
    bool canFinish(int numCourses, vector<pair<int, int>>& prerequisites) {
        if (numCourses <= 1)
            return true;

        vector<vector<int>> graph;
        int in[numCourses];  // 入度
        for (int i = 0; i < numCourses; i++) {
            in[i] = 0;
            vector<int> t;
            graph.push_back(t);
        }

        // [0, 1]: 1 -> 0
        for (int i = 0; i < prerequisites.size(); i++) {
            int x = prerequisites[i].first;
            int y = prerequisites[i].second;
            in[x]++;
            graph[y].push_back(x);
        }

        // 暂时不做堆优化，直接暴力
        int finished = 0;
        while (finished < numCourses) {
            int found = -1;
            for (int i = 0; i < numCourses; i++)
                if (in[i] == 0) {
                    found = i;
                    break;
                }
            if (found == -1)
                return false;
            // 上这门课
            for (int j = 0; j < graph[found].size(); j++)
                in[graph[found][j]]--;
            in[found] = -1; // 将这门课从已上列表里去掉……刚才忘了
            finished++;
        }
        return true;
    }
};
```

---

## 一些废话

因为要准备保研的机试，所以还是要刷题。但是我实在不知道该刷什么比较好。POJ上充满了经典题，CodeForces上每周都有比赛，UVa和《算法竞赛入门经典》是配套的。结果我最后还是来刷炙手可热的Leetcode了，因为最方便，可以勉强维持一点手感。我本来一直觉得用水题刷自己博客的屏很不合适，但是如果不这样，则实在没有办法逼自己继续做下去了。
