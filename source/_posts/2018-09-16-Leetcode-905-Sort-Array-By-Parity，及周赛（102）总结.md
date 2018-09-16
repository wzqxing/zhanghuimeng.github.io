---
title: Leetcode 905. Sort Array By Parity，及周赛（102）总结
urlname: leetcode-905-sort-array-by-parity-and-weekly-contest-102
toc: true
date: 2018-09-16 11:48:00
updated: 2018-09-16 15:27:00
tags: [Leetcode, Leetcode Contest, alg:Two Pointers]
---

题目来源：[https://leetcode.com/problems/sort-array-by-parity/description/](https://leetcode.com/problems/sort-array-by-parity/description/)

标记难度：Easy

提交次数：3/3

代码效率：

* 平凡的遍历：44ms
* 平凡的排序：36ms
* 不平凡的双指针：28ms

## 题意

给定整数数组`A`，要求将`A`中元素重排，使得所有偶数元素在前，奇数元素在后。

## 分析

这次比赛的题目整体又比较简单。所以我的排名又变高了，变成了271 / 4385（最近几次比赛人好像越来越多了，是错觉吗？）。做出来三道题；最后一题我也想出了几乎正确的做法，就是懒得写了。

---

### 一种平凡的做法：两次遍历

比赛的时候我就是这么写的。新开一个`vector`，先遍历`A`一次，把偶数元素都插入到里面；再遍历`A`一次，这次只插入奇数元素。时间复杂度是`O(n)`，额外空间复杂度是`O(n)`。

### 另一种平凡的做法：重写排序比较器

好吧，这种做法倒也没有那么平凡。重写一个排序的比较器，使得偶数都小于奇数，而偶数之间都相等。然后就可以直接把`A`排序了。时间复杂度是`O(n * log(n))`，额外空间复杂度是`O(1)`。[^solution]

[^solution]: [Leetcode 905 Solution](https://leetcode.com/problems/sort-array-by-parity/solution/)

### 不太平凡的做法：双指针

仍然和快排的想法是类似的。不过我的代码里似乎仍然写了太多的特判，还是这份代码比较好。[^lee]

[^lee]: [\[C++/Java\] In Place Swap](https://leetcode.com/problems/sort-array-by-parity/discuss/170734/C++Java-In-Place-Swap)

## 代码

### 遍历

```cpp
class Solution {
public:
    vector<int> sortArrayByParity(vector<int>& A) {
        vector<int> ans;
        for (int x: A)
            if (x % 2 == 0)
                ans.push_back(x);
        for (int x: A)
            if (x % 2 != 0)
                ans.push_back(x);
        return ans;
    }
};
```

### 排序

再一次回到了如何使用自己的比较器进行`std::sort`的这个问题中。此时你不止需要一个函数，还需要一个函数对象，把这个函数放到重载的`()`运算符中。[^sort]

[^sort]: [Cpp Reference: std::sort](https://en.cppreference.com/w/cpp/algorithm/sort)

```cpp
class Solution {
private:
    struct {
        bool operator()(const int& x, const int& y) const
        {
            return x % 2 < y % 2;
        }
    } Cmp;

public:
    vector<int> sortArrayByParity(vector<int>& A) {
        // https://en.cppreference.com/w/cpp/algorithm/sort
        sort(A.begin(), A.end(), Cmp);
        return A;
    }
};
```

### 双指针

```cpp
class Solution {
public:
    vector<int> sortArrayByParity(vector<int>& A) {
        int i = 0, j = 0;
        int n = A.size();

        while (i < n && A[i] % 2 == 0) i++;
        j = i;
        while (j < n && A[j] % 2 != 0) j++;

        while (i < n && j < n) {
            swap(A[i], A[j]);
            while (i < n && A[i] % 2 == 0) i++;
            while (j < n && A[j] % 2 != 0) j++;
        }

        return A;
    }
};
```
