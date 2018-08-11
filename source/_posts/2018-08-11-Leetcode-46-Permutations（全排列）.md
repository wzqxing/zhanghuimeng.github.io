---
title: Leetcode 46. Permutations（全排列）
urlname: leetcode-46-permutations
toc: true
date: 2018-08-11 17:32:30
updated: 2018-08-11 18:17:30
tags: [Leetcode]
---

题目来源：[https://leetcode.com/problems/permutations/description/](https://leetcode.com/problems/permutations/description/)

标记难度：Medium

提交次数：3/4

代码效率：

* `next_permutation`版本：99.91%
* DFS版本：99.92%
* Heap算法：99.92%
* Steinhaus–Johnson–Trotter算法：……懒得写了

## 题意

计算若干整数全排序，整数两两不同。

## 分析

### STL版本

直接用STL中提供的[next_permutation](http://www.cplusplus.com/reference/algorithm/next_permutation/)可能是最简单的一种方式了。但是，值得注意的是，在开始使用这个函数之前，需要把数组排序。

### DFS版本

用DFS直接做也是一种思路，而且并不太难。

显然我基本已经把我的C++忘光了（或者说我就没有好好学过STL和模板）。C++ vector的`push_back`函数使用的是相应的[拷贝构造函数](https://stackoverflow.com/questions/6717821/is-vectorpush-back-making-a-shallow-copy-how-to-solve-this)，所以不需要再单独复制一次vector。

### Heap算法

可能是效率最高的排列算法之一。好吧，我一时搞不清楚这个算法了，但是按照[维基百科](https://en.wikipedia.org/wiki/Heap%27s_algorithm)的思路，应该很好写……

### Steinhaus–Johnson–Trotter算法

之前读《组合数学》的时候曾经看到过这个[算法](https://en.wikipedia.org/wiki/Steinhaus%E2%80%93Johnson%E2%80%93Trotter_algorithm)，是一种很有趣的思路——虽然在开始做这道题之前，我已经把具体的算法给忘光了。……算了，现在懒得写了。

## 代码

### STL版本

```cpp
class Solution {
public:
    vector<vector<int>> permute(vector<int>& nums) {
        vector<vector<int>> ans;
        sort(nums.begin(), nums.end());  // 如果使用next_permutation()，是需要排序的
        do {
            vector<int> x(nums);
            ans.push_back(x);
        } while (next_permutation(nums.begin(), nums.end()));
        return ans;
    }
};
```

### DFS版本

```cpp
class Solution {
private:
    vector<vector<int>> ans;

    void dfs(vector<int>& perm, vector<int>& nums, vector<bool> used) {
        if (perm.size() == nums.size()) {
            vector<int> tmp(perm);  // 事实上没有必要
            ans.push_back(tmp);
            return;
        }
        for (int i = 0; i < nums.size(); i++) {
            if (!used[i]) {
                used[i] = true;
                perm.push_back(nums[i]);
                dfs(perm, nums, used);
                perm.pop_back();
                used[i] = false;
            }
        }
    }

public:
    vector<vector<int>> permute(vector<int>& nums) {
        vector<bool> used(nums.size(), false);
        vector<int> perm;
        dfs(perm, nums, used);
        return ans;
    }
};
```

### Heap算法

```cpp
class Solution {
private:
    vector<vector<int>> ans;

    void generate(int n, vector<int>& nums) {
        if (n == 1) {
            ans.push_back(nums);
            return;
        }
        for (int i = 0; i < n - 1; i++) {
            generate(n-1, nums);
            if (n % 2 == 0)
                swap(nums[i], nums[n-1]);
            else
                swap(nums[0], nums[n-1]);
        }
        generate(n - 1, nums);
    }

public:
    vector<vector<int>> permute(vector<int>& nums) {
        generate(nums.size(), nums);
        return ans;
    }
};
```
