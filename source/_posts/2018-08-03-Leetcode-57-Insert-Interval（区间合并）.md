---
title: Leetcode 57. Insert Interval（区间合并）
urlname: leetcode-57-insert-interval
toc: true
date: 2018-08-03 22:22:32
updated: 2018-08-03 22:54:00
tags: [Leetcode, alg:Array]
---

题目来源：[https://leetcode.com/problems/insert-interval/description/](https://leetcode.com/problems/insert-interval/description/)

标记难度：Hard

提交次数：1/2

代码效率：98.07%

## 题意

在一系列已经排好序的且不重叠的区间中插入一个新的区间，要求进行适当的合并，并且插入到正确的位置。

## 分析

我开始时的错误思路是这样的：遍历所有的旧interval，在能够进行合并时就把旧interval合并到新interval中，否则直接插入到新vector中；最后把更新过的interval插入到新vector中。这种做法显然有一个问题：如果新interval不需要和旧interval合并，那这种算法就找不到正确的插入位置了。我猜我的注意力被**merge if necessary**吸引住了，完全忘了有时候不需要合并这回事。

即使纠正了这个问题，我的代码也不是很优雅，最后我遍历了两次数组，在第二次遍历的时候硬是把新interval插进去了，使用的是[insert](https://zh.cppreference.com/w/cpp/container/vector/insert)：

```
auto it = vec.begin();  // 获得vector起始位置的迭代器
// 第一个参数是插入位置（如果为vec.begin()+i，则事实上的效果是插入到第i个元素前面）
// 第二个参数是插入的具体元素
// std::vector::insert的重载甚多，感觉不能胜记……
it = vec.insert(it, 200);
```

比较好的做法是这样的（灵感来自[Short and straight-forward Java solution](https://leetcode.com/problems/insert-interval/discuss/21602/Short-and-straight-forward-Java-solution)）：遍历所有旧interval，首先将所有严格位于新interval左侧的旧interval保存下来（当然，可能没有这样的interval）；然后将与新interval重叠的旧interval进行适当的合并，合并结束的时候，将新interval保存下来；最后将所有严格位于合并后的新interval右侧的旧interval保存下来（当然，也可能没有这样的interval）。

如果用Python而不是Java或C++，这个问题会稍微更好思考一点，毕竟Python的数组操作更加灵活……（参见[7+ lines, 3 easy solutions](https://leetcode.com/problems/insert-interval/discuss/21622/7+-lines-3-easy-solutions)中精妙的写法）

P.S. 我最近找到了遍历C++ vector的更方便的写法（[Iterate through a C++ Vector using a 'for' loop](https://stackoverflow.com/a/12702744)）：

```
vector<int> vi;
...
for(int i : vi)
  cout << "i = " << i << endl;
```

## 代码

```
/**
 * Definition for an interval.
 * struct Interval {
 *     int start;
 *     int end;
 *     Interval() : start(0), end(0) {}
 *     Interval(int s, int e) : start(s), end(e) {}
 * };
 */
class Solution {
    bool isOverlap(Interval i1, Interval i2) {
        return !(i1.end < i2.start || i2.end < i1.start);
    }

    Interval merge(Interval i1, Interval i2) {
        return Interval(min(i1.start, i2.start), max(i1.end, i2.end));
    }

public:
    vector<Interval> insert(vector<Interval>& intervals, Interval newInterval) {
        // 现在的想法是，维护这个newInterval，然后遍历所有的旧interval，发现重叠时就更新newInterval，直到完全不重叠为止
        // 需要证明的是，即使因为overlap更新了当前的interval，这并不会影响到之前的interval
        // 没有考虑到如何找到正确插入位置的问题。
        vector<Interval> ans;
        int finished = -1;
        for (Interval interval: intervals) {
            if (!isOverlap(interval, newInterval)) {
                if (finished == 0) {
                    ans.push_back(newInterval);
                    finished = 1;
                }
                ans.push_back(interval);
            }
            else {
                if (finished == -1)
                    finished = 0;
                newInterval = merge(newInterval, interval);
            }
        }
        // 找到正确的插入位置
        if (finished != 1) {
            for (int i = 0; i < ans.size(); i++) {
                if (ans[i].start > newInterval.end) {
                    ans.insert(ans.begin() + i, newInterval);
                    finished = 1;
                    break;
                }
            }
            if (finished != 1)
                ans.push_back(newInterval);
        }
        return ans;
    }
};
```
