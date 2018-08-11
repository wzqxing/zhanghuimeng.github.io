---
title: Leetcode 347. Top K Frequent Elements（数量统计量）
urlname: leetcode-347-top-k-frequent-elements
toc: true
date: 2018-08-12 01:02:31
updated: 2018-08-12 01:02:31
tags: [Leetcode]
---

题目来源：[https://leetcode.com/problems/top-k-frequent-words/description/](https://leetcode.com/problems/top-k-frequent-words/description/)

标记难度：Medium

提交次数：1/2

代码效率：98.50%

## 题意

从一个数组中选出出现次数前k多的元素。

## 分析

这道题和上一道Top K（[Leetcode 692](/;ost/leetcode-692-top-k-frequent-words)）基本是完全一样的——事实上，我是从那边的题解区过来的。区别可能是，这里完全没有对break tie的要求，所以我就直接用HashMap+桶排序做了（其他的做法参见[3 ways to solve this problem](https://leetcode.com/problems/top-k-frequent-elements/discuss/81631/3-ways-to-solve-this-problem)），然后就过了。

中间错了一次是因为桶的数量开少了一个。

## 代码

```cpp
class Solution {
public:
    vector<int> topKFrequent(vector<int>& nums, int k) {
        unordered_map<int, int> freq;
        for (int x: nums)
            freq[x]++;

        // 这道题的题干中没有提到break tie的问题啊……
        // 如果按照没有tie的方法去做，那桶排序将变得非常简单
        vector<vector<int>> bucket;
        for (int i = 0; i <= nums.size(); i++)
            bucket.push_back(vector<int>());
        for (auto const& i: freq)
            bucket[i.second].push_back(i.first);

        vector<int> topk;
        while (topk.size() < k) {
            for (int j = bucket.size() - 1; j >= 0; j--) {
                for (int x: bucket[j])
                    if (topk.size() < k)
                        topk.push_back(x);
                    else
                        break;
                if (topk.size() >= k)
                    break;
            }
        }

        return topk;
    }
};
```
