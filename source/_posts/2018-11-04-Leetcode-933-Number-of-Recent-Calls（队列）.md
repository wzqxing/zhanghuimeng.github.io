---
title: Leetcode 933. Number of Recent Calls（队列），及周赛（108）总结
urlname: leetcode-933-number-of-recent-calls-and-weekly-contest-108
toc: true
date: 2018-11-04 11:18:53
updated: 2018-11-04 14:29:00
tags: [Leetcode, Leetcode Contest, alg:queue]
---

题目来源：[https://leetcode.com/problems/number-of-recent-calls/description/](https://leetcode.com/problems/number-of-recent-calls/description/)

标记难度：Easy

提交次数：1/1

代码效率：148ms

## 题意

给定一系列严格递增的ping时间戳，问每个时间戳3000毫秒前共有几次ping。

## 分析

今天比赛的时候发生了一点incident。实验室没开门，于是我坐在外面的桌子上又冷又没电地打比赛，然后第三题的Dijstra居然还写错了调不出来，最后紧张地在没电和比赛结束之前换成了用`priority_queue`的做法……（事实证明不加堆优化还是会超时的）然后当然没时间做第四题了。我一眼望去以为第四题要用后缀树之类的做法，结果好像比我想象的要简单。

最后排名是466 / 2948。今天北京下雨了。

---

直接用队列维护一个时间窗口就可以了。（不知道为什么我比赛的时候用的是deque）

## 代码

```cpp
class RecentCounter {
private:
    deque<int> pings;

public:
    RecentCounter() {

    }

    int ping(int t) {
        while (!pings.empty() && pings.front() < t - 3000)
            pings.pop_front();
        pings.push_back(t);
        return pings.size();
    }
};
```
