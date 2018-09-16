---
title: Leetcode 904. Fruit Into Baskets
urlname: leetcode-904-fruit-into-baskets
toc: true
date: 2018-09-16 15:37:53
updated: 2018-09-16 16:19:00
tags: [Leetcode, Leetcode Contest, alg:Greedy, alg:Two Pointers]
---

题目来源：[https://leetcode.com/problems/fruit-into-baskets/description/](https://leetcode.com/problems/fruit-into-baskets/description/)

标记难度：Medium

提交次数：2/2

代码效率：

* 分块扫描：148ms
* 滑动窗口：192ms

## 题意

给定数组`tree`，求其中最长的只包含两种元素的连续子序列的长度。

## 分析

### 分块扫描

比赛的时候我就是这么写的。

---

首先显然可以把这个数组按连续相同的元素分块。这样，相邻的两块所表示的元素必然就是不同的了。然后再考虑怎么求这种子序列。

以样例中的序列`[3,3,3,1,2,1,1,2,3,3,4]`为例，分块之后，即可得到

```
[(3, 3), (1, 1), (2, 1), (1, 2), (2, 1), (3, 2), (4, 1)]
```

首先尝试从`(3, 3)`开始寻找最长子序列。显然我们能够找到的是`[(3, 3), (1, 1)]`。此时下一个块对应的元素必然既不是3也不是1（或者也可能没有下一个块）。一个事实是，如果从这个最长子序列中的任意非最后一个元素开始寻找最长子序列，都必然会结束在当前的最后一个元素。所以下一次搜寻只需从当前最长子序列的最后一个元素开始。

于是之后从`(1, 1)`开始寻找，找到了`[(1, 1), (2, 1), (1, 2), (2, 1)]`；再从`(2, 1)`开始寻找，找到了`[(2, 1), (3, 2)]`；从`(3, 2)`开始寻找，找到了`[(3, 2), (4, 1)]`；最后找到了`[(4, 1)]`。其中最长的子序列是从`(1, 1)`开始的。

### 滑动窗口

看了题解之后，我反而觉得这种想法更容易想。简单来说，令`opt(j)`表示以`j`结尾的最长合法子序列的首位坐标，则显然`opt(j)`是随`j`单调递增的。也就是说，`[opt(j), j]`形成了一个滑动窗口。

因此我们可以在遍历`j`的同时维护滑动窗口中各元素的数量，当元素种类超过2时，则将窗口下沿向前滑，并相应地删除元素，直到元素种类回到2。[^solution]

[^solution]: [Leetcode 904 Solution](https://leetcode.com/problems/fruit-into-baskets/solution/)

## 代码

### 分块扫描

```cpp
class Solution {
public:
    int totalFruit(vector<int>& tree) {
        // 分块
        vector<pair<int, int>> subs;  // cnt, ele
        int cnt = 0;
        for (int i = 0; i < tree.size(); i++) {
            if (i != 0 && tree[i - 1] != tree[i]) {
                subs.emplace_back(cnt, tree[i - 1]);
                cnt = 0;
            }
            cnt++;
        }
        if (cnt != 0)
            subs.emplace_back(cnt, tree.back());

        // 进行扫描
        // eleSet中存储的是当前找到的元素
        unordered_set<int> eleSet;
        int i = 0;
        int curSum = 0;
        int ans = -1;
        while (i < subs.size()) {
            while (i < subs.size() && eleSet.size() <= 2) {
                int cnt = subs[i].first;
                int ele = subs[i].second;

                if (eleSet.find(ele) == eleSet.end() && eleSet.size() == 2)
                    break;
                if (eleSet.find(ele) == eleSet.end())
                    eleSet.insert(ele);
                curSum += cnt;
                i++;
            }
            ans = max(ans, curSum);
            curSum = 0;
            eleSet.clear();
            // 如果扫描已经完成则不再回退
            if (i >= subs.size())
                break;
            // 回退1，重新开始扫描
            i--;
        }
        return ans;
    }
};
```

### 滑动窗口

```cpp
class Solution {
public:
    int totalFruit(vector<int>& tree) {
        int start = 0;  // 窗口前沿
        unordered_map<int, int> cntMap;
        int maxn = -1;
        // 遍历窗口后沿
        for (int i = 0; i < tree.size(); i++) {
            cntMap[tree[i]]++;
            // 维护窗口中元素种类
            while (cntMap.size() > 2) {
                cntMap[tree[start]]--;
                if (cntMap[tree[start]] == 0)
                    cntMap.erase(tree[start]);
                start++;
            }
            maxn = max(maxn, i - start + 1);
        }
        return maxn;
    }
};
```
