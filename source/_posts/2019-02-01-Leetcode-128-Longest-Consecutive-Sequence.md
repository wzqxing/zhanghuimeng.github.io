---
title: Leetcode 128. Longest Consecutive Sequence
urlname: leetcode-128-longest-consecutive-sequence
toc: true
date: 2019-02-01 16:04:06
updated: 2019-02-01 16:04:06
tags: [Leetcode]
---

题目来源：[https://leetcode.com/problems/longest-consecutive-sequence/description/](https://leetcode.com/problems/longest-consecutive-sequence/description/)

标记难度：Hard

提交次数：2/2

代码效率：

* 排序：100.00%（4ms）
* Hash Set：100.00%（4ms）

## 题意

给定一个（未排序的）整数数组，找到最长的连续整数序列的长度。要求复杂度是`O(n)`。

## 分析

虽然说要求复杂度是`O(n)`，但是我想都没想就排了个序……然后直接扫描一遍数组就可以了。

[题解](https://leetcode.com/articles/longest-consecutive-sequence/)里给出了不排序的做法：把每个数都存到一个哈希表里，然后对于每个数，如果它之前没有出现在某个连续序列里（也就是说比它少1的数不存在），则尝试寻找以它为开头的连续整数序列的长度。

## 代码

### 排序

```cpp
class Solution {
public:
    int longestConsecutive(vector<int>& nums) {
        if (nums.size() == 0) return {};  // ……
        sort(nums.begin(), nums.end());
        int ans = -1;
        int cnt = 1;
        for (int i = 1; i < nums.size(); i++) {
            if (nums[i] == nums[i-1]) continue;
            if (nums[i] == nums[i-1] + 1) {
                cnt++;
                ans = max(ans, cnt);
            }
            else
                cnt = 1;
        }
        return max(ans, cnt);
    }
};
```

### 哈希表

```cpp
class Solution {
public:
    int longestConsecutive(vector<int>& nums) {
        if (nums.size() == 0) return {};
        unordered_set<int> s;
        for (int num: nums)
            s.insert(num);
        int maxn = -1;
        for (int num: nums) {
            if (s.find(num - 1) == s.end()) {
                int l = 1;
                while (s.find(num + l) != s.end())
                    l++;
                maxn = max(maxn, l);
            }
        }
        return maxn;
    }
};
```
