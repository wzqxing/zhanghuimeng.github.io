---
title: Leetcode 739. Daily Temperatures（栈）
urlname: leetcode-739-daily-temperatures
toc: true
date: 2018-09-15 20:09:27
updated: 2018-09-17 02:09:00
tags: [Leetcode, alg:Stack]
---

题目来源：[https://leetcode.com/problems/daily-temperatures/description/](https://leetcode.com/problems/daily-temperatures/description/)

标记难度：Medium

提交次数：3/6

代码效率：

* 排序+BST：14.41%
* 栈：58.54%
* 暴力：48.92%

## 题意

和[Leetcode 901](/post/leetcode-901-online-stock-span)差不多，不过顺序正好相反。

## 分析

因为是几乎一样的所以没什么好分析的。但是在这道题中，因为数据范围比较小（`[30, 100]`），所以可以从右往左扫描，维护每个温度出现的最左侧的位置，然后对当前的温度，遍历比它高的所有温度，找出其中位于最左侧的。这大概是一种暴力算法吧。[^solution]

[^solution]: [Leetcode 739 Solution](https://leetcode.com/problems/daily-temperatures/solution/)

## 代码

### 离线算法：排序+BST

虽然写了计数排序，但这个算法仍然超时了。一般来说，`30000`规模的数据用`O(n * log(n))`的算法没有什么太大的问题，此处不知道是因为用的STL太多了，Leetcode太面向对象了，还是Leetcode的评测机太渣了。既然如此，纯递归的算法更没有写的必要……

2018.9.16 UPDATE：我错怪Leetcode了，实际上是我自己没用对`set.lower_bound`，结果变成了`O(n^2)`的算法。现在这个算法是能过的了……

```cpp
class Solution {
private:
    struct Pair {
        int val;
        int index;

        Pair(int v, int i) {
            val = v;
            index = i;
        }

        friend bool operator < (const Pair& x, const Pair& y) {
            if (x.val != y.val) return x.val < y.val;
            return x.index > y.index;
        }
    };

    void bucketSort(vector<Pair>& a) {
        list<Pair> bucket[105];
        for (int i = 0; i < a.size(); i++) {
            bucket[a[i].val].push_front(a[i]);
        }
        int n = 0;
        for (int i = 30; i <= 100; i++) {
            for (auto j = bucket[i].begin(); j != bucket[i].end(); j++){
                a[n++] = *j;
            }
        }
    }

public:
    vector<int> dailyTemperatures(vector<int>& temperatures) {
        vector<Pair> loc;
        int n = temperatures.size();
        vector<int> ans(n);
        for (int i = 0; i < n; i++) {
            loc.emplace_back(temperatures[i], i);
        }
        // sort(loc.begin(), loc.end());
        bucketSort(loc);
        set<int> indSet;

        for (int i = n - 1; i >= 0; i--) {
            // auto it = lower_bound(indSet.begin(), indSet.end(), loc[i].index);
            // 上面那个写法是错误的，因为std::lower_bound函数并不知道set的内部结构
            // 下面使用std::set::lower_bound才是正确的
            auto it = indSet.lower_bound(loc[i].index);
            if (it == indSet.end())
                ans[loc[i].index] = 0;
            else
                ans[loc[i].index] = *it - loc[i].index;
            indSet.insert(loc[i].index);
        }
        return ans;
    }
};
```

### 在线算法：栈

```cpp
class Solution {
public:
    vector<int> dailyTemperatures(vector<int>& temperatures) {
        stack<pair<int, int>> s;
        int n = temperatures.size();
        vector<int> ans(n);
        s.emplace(1000, -1);  // 一个哨兵
        for (int i = n - 1; i >= 0; i--) {
            while (!s.empty() && s.top().first <= temperatures[i])
                s.pop();
            if (s.top().second == -1) ans[i] = 0;
            else ans[i] = s.top().second - i;
            s.emplace(temperatures[i], i);
        }
        return ans;
    }
};
```

### 在线算法：暴力

如果要求在线，数据从右向左提供，则显然暴力算法是可以在线的。

```cpp
class Solution {
public:
    vector<int> dailyTemperatures(vector<int>& temperatures) {
        int n = temperatures.size();
        int lastIdx[105];
        vector<int> ans(n, 0);
        for (int i = 0; i < 105; i++)
            lastIdx[i] = n;
        for (int i = n - 1; i >= 0; i--) {
            int minn = n;
            for (int j = temperatures[i] + 1; j <= 100; j++)
                minn = min(minn, lastIdx[j]);
            if (minn == n)
                ans[i] = 0;
            else
                ans[i] = minn - i;
            lastIdx[temperatures[i]] = i;
        }
        return ans;
    }
};
```
