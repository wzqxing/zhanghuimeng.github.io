---
title: Leetcode 911. Online Election（BST）
urlname: leetcode-911-online-election
toc: true
date: 2018-09-23 15:47:44
updated: 2018-09-23 16:46:44
tags: [Leetcode, Leetcode Contest, alg:Binary Search Tree, alg:Hash Table]
---

题目来源：[https://leetcode.com/problems/online-election/description/](https://leetcode.com/problems/online-election/description/)

标记难度：Medium

提交次数：2/4

代码效率：

* 使用BST进行离线计算：248ms
* 线性扫描：384ms
* 二维链表：396ms

## 题意

给定`N`个时间点，在`times[i]`时刻，就有一个人投票给`person[i]`。对于若干个时刻`t`，请给出`t`时刻领先的人。

## 分析

这道题的设计好像不是很适合在线方法，所以我觉得离线还是比较合适的。

### BST+二分查找

我比赛的时候用了一种相当之麻烦的方法——仍然是用优先队列的思想，在TreeSet中存储`(person, votes, recency)`的元组，然后在每个有人投票的时间点，更新TreeSet，并找出当前领先的人，保存下来；之后在查询的时候进行二分查找。这种方法的复杂度大约是`O(N * log(N) + M * log(N))`（假定`M`是查询的总次数）。

### 线性扫描+二分查找

事实上找出各个时刻领先的人并不需要这么麻烦。事实上`persons`和`times`数组都是已经排好序的了。因此，可以用一个`map`来维护每个人的票数，然后在每个时刻更新被投票的人的票数，并判断**当前领先的人是否会变成这个人**。如果不变的话，完全可以忽略这个时间点。这种方法的复杂度大约是`O(N + M * log(N))`。[^lee215]

[^lee215]: [\[C++/Java/Python\] Binary Search in Times](https://leetcode.com/problems/online-election/discuss/173382/C++JavaPython-Binary-Search-in-Times)

### 二维链表

我感觉这是一种特别神奇的做法……简单来说，我觉得可以维护一个`vector<vector<pair<int, int>>`，其中外层`vector`的含义是，所有投的是某人的第`i`票的选票；内层`vector`按时间顺序存储了每张选票的`(person, time)`。用题目中的例子来描述一下：

```
[0,1,1,0,0,1,0],[0,5,10,15,20,25,30]

+------+    +---+      +---+      +---+      +---+
| Head |----| 1 |------| 2 |------| 3 |------| 4 |  count
+------+    +-+-+      +-+-+      +-+-+      +---+
              |          |          |          |
            +-+-+     -+-+-+-    -+-+-+-    -+-+-+-          |
            |0,0|     |1, 10|    |0, 20|    |0, 30|          |
            +-+-+     -+-+-+-    -+-+-+-    -+-+-+-          | Most recent
              |          |          |                        |
            +-+-+     -+-+-+-    -+-+-+-                     |
            |1,5|     |0, 15|    |1, 25|                     |
            +-+-+     -+-+-+-    -+-+-+-                     v
```

然后很显然，每个外层`vector`的第一个元素都是递增的，每个内层`vector`自己也是递增的。对于每个时间点，我们可以首先对外层`vector`进行二分查找，然后再在内层进行二分查找。整体复杂度是`O(N + M * log(N)^2)`。[^solution]

[^solution]: [911. Online Election, Approach 2: Precomputed Answer + Binary Search](https://leetcode.com/articles/online-election/#approach-2-precomputed-answer-binary-search)

以及，我觉得这种做法有些像[Leetcode 460](/post/leetcode-460-lfu-cache)中给出的HashMap+List方法。

## 代码

### BST

```cpp
class TopVotedCandidate {
private:
    vector<pair<int, int>> timesOfVote;  // time, vote
    vector<int> times;
    vector<int> leaders;
    int N;

    struct VotedPerson {
        int person;
        int recency;
        int votes;

        VotedPerson(int p, int r, int v) {
            person = p;
            recency = r;
            votes = v;
        }

        friend bool operator < (const VotedPerson& p1, const VotedPerson& p2) {
            if (p1.votes != p2.votes) return p1.votes > p2.votes;
            return p1.recency > p2.recency;
        }

        friend bool operator == (const VotedPerson& p1, const VotedPerson& p2) {
            return p1.person == p2.person && p1.recency == p2.recency && p1.votes == p2.votes;
        }
    };

public:
    TopVotedCandidate(vector<int> persons, vector<int> times) {
        N = persons.size();
        for (int i = 0; i < N; i++) {
            timesOfVote.emplace_back(times[i], persons[i]);
        }
        sort(timesOfVote.begin(), timesOfVote.end());

        unordered_map<int, int> recMap;  // person to recency
        unordered_map<int, int> voteMap;  // person to vote
        set<VotedPerson> treeSet;
        for (int i = 0; i < N; i++) {
            int person = timesOfVote[i].second;
            int time = timesOfVote[i].first;
            int recency = recMap[person];
            int votes = voteMap[person];
            auto it = treeSet.find(VotedPerson(person, recency, votes));
            if (it != treeSet.end()) treeSet.erase(it);
            recMap[person] = recency = i;
            voteMap[person]++;
            votes++;
            treeSet.insert(VotedPerson(person, recency, votes));

            this->times.push_back(time);
            this->leaders.push_back(treeSet.begin()->person);
        }
    }

    int q(int t) {
        auto it = upper_bound(times.begin(), times.end(), t);
        it--;
        return leaders[it - times.begin()];
    }
};
```

### 线性扫描

```cpp
class TopVotedCandidate {
private:
    vector<int> leader;
    vector<int> ctimes;

public:
    TopVotedCandidate(vector<int> persons, vector<int> times) {
        unordered_map<int, int> voteMap;
        int curLeader = -1, curLeadVotes = -1;
        for (int i = 0; i < persons.size(); i++) {
            voteMap[persons[i]]++;
            if (voteMap[persons[i]] >= curLeadVotes) {
                curLeadVotes = voteMap[persons[i]];
                curLeader = persons[i];
                leader.push_back(curLeader);
                ctimes.push_back(times[i]);
            }
        }
    }

    int q(int t) {
        auto it = upper_bound(ctimes.begin(), ctimes.end(), t);
        return leader[it - ctimes.begin() - 1];
    }
};
```

### 二维链表

```cpp
class TopVotedCandidate {
private:
    vector<vector<pair<int, int>>> bucket;  // (time, person)

public:
    TopVotedCandidate(vector<int> persons, vector<int> times) {
        unordered_map<int, int> voteMap;
        for (int i = 0; i < persons.size(); i++) {
            voteMap[persons[i]]++;
            int c = voteMap[persons[i]];
            while (bucket.size() < c) bucket.push_back(vector<pair<int, int>>());
            bucket[c-1].emplace_back(times[i], persons[i]);
        }
    }

    int q(int t) {
        int l = 0, r = bucket.size();
        // upper_bound, then -1
        while (l < r) {
            int mid = l + (r - l) / 2;
            if (bucket[mid][0].first <= t)
                l = mid + 1;
            else
                r = mid;
        }
        int i = l - 1;

        l = 0, r = bucket[i].size();
        while (l < r) {
            int mid = l + (r - l) / 2;
            if (bucket[i][mid].first <= t)
                l = mid + 1;
            else
                r = mid;
        }
        int j = l - 1;
        if (j < 0) j = 0;
        return bucket[i][j].second;
    }
};
```
