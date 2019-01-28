---
title: Leetcode 981. Time Based Key-Value Store（二分查找）
urlname: leetcode-981-time-based-key-value-store
toc: true
date: 2019-01-27 21:32:53
updated: 2019-01-28 00:21:00
tags: [Leetcode, Leetcode Contest, alg:Binray Search, alg:Hash Table]
---

题目来源：[https://leetcode.com/problems/time-based-key-value-store/description/](https://leetcode.com/problems/time-based-key-value-store/description/)

标记难度：Medium

提交次数：1/1

代码效率：33.33%（212ms）

## 题意

给定一系列set和get操作，其中：

* 每个set操作包含一个key，一个value和一个timestamp，其中timestamp是严格递增的
* 每个get操作包含一个key和一个timestamp，要求找出与这个key相等且时间戳<=timestamp的set操作中时间戳最大的set对应的value

所有操作总数不超过12万次。

## 分析

这个题目咋一看很唬人，其实完全不是那么回事。解题思路很简单：

* 用一个map维护set操作的key对应的value和timestamp对的列表
* 对于每个get操作，从key对应的列表中通过二分查找，找到最大的符合要求的timestamp对应的value

如果map用的是Hash Table，记总操作次数为`N`，那么set的复杂度是`O(1)`，get的复杂度是`O(log(N))`；如果用的是树结构的话，那set的复杂度就是`O(log(N))`，get的复杂度是`O(log^2(N))`（不过显然可以把它写得更好一些）。

这次我写了二分查找。一般来说，如果二分查找（`m = (l + r) / 2`）之后的转移条件是`l = m + 1`，`r = m`的话，那循环条件就可以写成`l < r`；但是如果转移条件是`l = m`，`r = m - 1`的话，循环条件就需要写成`l < r-1`，然后判断`l`还是`r`是解……

## 代码

```cpp
class TimeMap {
private:
    map<string, vector<pair<int, string>>> mmap;
    
public:
    TimeMap() {
        
    }
    
    void set(string key, string value, int timestamp) {
        mmap[key].emplace_back(timestamp, value);
    }
    
    string get(string key, int timestamp) {
        if (mmap.find(key) == mmap.end()) return "";
        int n = mmap[key].size();
        if (mmap[key][0].first > timestamp) return "";
        // 二分查找
        int l = 0, r = n - 1;
        while (l < r - 1) {
            int m = (l + r) / 2;
            if (mmap[key][m].first <= timestamp) l = m;
            else
                r = m - 1;
        }
        if (mmap[key][r].first <= timestamp) return mmap[key][r].second;
        return mmap[key][l].second;
    }
};
```