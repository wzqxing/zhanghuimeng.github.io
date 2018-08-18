---
title: Leetcode 90. Subsets II（构造）
urlname: leetcode-90-subsets-ii
toc: true
date: 2018-08-19 03:29:38
updated: 2018-08-19 04:22:38
tags: [Leetcode]
---

题目来源：[https://leetcode.com/problems/subsets-ii/description/](https://leetcode.com/problems/subsets-ii/description/)

标记难度：Medium

提交次数：2/2

代码效率：

* 直接回溯：100.00%
* 迭代：23.82%

## 题意

找出一个数组中所有可能的子集。（可能有重复元素）

## 分析

这道题无论怎样都能搞出来，但是有一些做法比其他做法更加巧妙。我开始时的做法是这样的：先统计重复元素的个数，然后直接DFS。但事实上根本没有这样做的必要：把数组排序之后，重复元素会自动聚合在一起；由于所有的结果都是必须的，DFS也不一定必要。事实上我们可以迭代地构造这些子集：`[a_1, a_m]`的所有子集必然分为两种，一种是`[a1, a_{m-1}]`的所有子集，另一种是`[a1, a_{m-1}]`的所有子集中加上`a_m`。（[C++ solution and explanation](https://leetcode.com/problems/subsets-ii/discuss/30168/C++-solution-and-explanation)）

另一种思路甚至更加有趣。显然我们在这个问题中需要避免的是重复的子集；那么何时会出现重复的子集呢？在迭代式的构造方法中，当之前添加过的元素和当前元素相同时，就会出现重复。但我们可以直接在迭代过程中规避这个问题：当前元素和前一元素重复时，我们只对当前集合中添加了前一元素的一半继续进行复制和添加操作，不再重复对前一半进行相同的操作。（[Simple iterative solution](https://leetcode.com/problems/subsets-ii/discuss/30137/Simple-iterative-solution)）

最后，如果数组为空，按照标准答案，我们应该返回一个空集。但实际上我认为应该返回一个含有空集的集合……所以我在题目中[报了个错](https://leetcode.com/problems/subsets-ii/discuss/161181/Possible-Program-and-Test-Data-Bug:-A-Corner-case-for-empty-sets)。

## 代码

### 直接回溯

```cpp
class Solution {
private:
    vector<vector<int>> ans;
    vector<pair<int, int>> diverseNum;

    void dfs(int i, vector<int>& u) {
        if (i >= diverseNum.size()) {
            ans.push_back(u);
            return;
        }
        for (int j = 0; j <= diverseNum[i].second; j++) {
            for (int k = 1; k <= j; k++)
                u.push_back(diverseNum[i].first);
            dfs(i + 1, u);
            for (int k = 1; k <= j; k++)
                u.pop_back();
        }
    }

public:
    vector<vector<int>> subsetsWithDup(vector<int>& nums) {
        // 这道题的复杂度可以说是比较高了
        // 既然可能有duplicate，就先用map统计元素个数，然后枚举subset

        // 空集的子集？
        if (nums.size() == 0)
            return ans;

        map<int, int> freq;
        for (int num: nums)
            freq[num]++;

        for (auto const& i: freq)
            diverseNum.push_back(make_pair(i.first, i.second));

        vector<int> u;
        dfs(0, u);

        return ans;
    }
};
```

### 迭代

```cpp
class Solution {

public:
    vector<vector<int>> subsetsWithDup(vector<int>& nums) {
        vector<vector<int>> subsets;
        if (nums.size() == 0)
            return subsets;
        subsets.push_back(vector<int>());
        sort(nums.begin(), nums.end());
        int lastIndex = 0;
        for (int i = 0; i < nums.size(); i++) {
            if (i == 0 || nums[i] != nums[i - 1])
                lastIndex = 0;
            int size = subsets.size();
            for (int j = lastIndex; j < size; j++) {
                subsets.push_back(subsets[j]);
                subsets.back().push_back(nums[i]);
            }
            lastIndex = size;
        }
        return subsets;
    }
};
```
