---
title: Leetcode 961. N-Repeated Element in Size 2N Array
urlname: leetcode-961-n-repeated-element-in-size-2n-array
toc: true
date: 2018-12-23 11:58:28
updated: 2018-12-24 01:46:00
tags: [Leetcode, Leetcode Contest, alg:Array, alg:Hash Table]
---

题目来源：[https://leetcode.com/problems/n-repeated-element-in-size-2n-array/description/](https://leetcode.com/problems/n-repeated-element-in-size-2n-array/description/)

标记难度：Easy

提交次数：4/4

代码效率：

* 排序：36ms
* 计数：32ms
* reduce：40ms
* 随机：40ms

## 题意

有一个长度为`2N`的数组，其中有`N+1`个不同元素，且只有一个元素出现了`N`次。返回这个元素。

## 分析

比赛时我用的是最简单的方法，排序：排序之后直接比较前后的数是否相等。

---

排序可以说是最low的方法了（复杂度高达`O(N^2)`），但在比赛时根据KISS原则，这么写也是合理的。

用哈希表显然也可以做，方法十分简单，不多说了。

下一种方法基于一种有趣的观察，我称之为reduce：把一个整个数组的性质（`N`个重复数字，不妨设为`x`；其他`N`个数字都是不重复的）缩小到其中一部分元素的性质。在所有的长度为4的子序列中，其中必然至少有一个出现了两个`x`。原因是这样的：考虑把这个子序列分成两个长度为2的子序列：由鸽巢原理，其中必然至少有一个数字是`x`。但是单看长度为2的子序列是无法判断的（因为`x`只会出现`N`次，无法保证一定会出现包含两个`x`的子序列），所以只能看长度为4的子序列。如果至少有一个出现了两个`x`的长度为2的子序列，则包含这个子序列的长度为4的子序列必然也至少包含两个`x`；如果每个长度为2的子序列都只有一个`x`，那么仍然至少存在一个出现了两个`x`的长度为4的子序列。[^solution]

[^solution]: [LeetCode Official Solution - 961. N-Repeated Element in Size 2N Array](https://leetcode.com/problems/n-repeated-element-in-size-2n-array/discuss/208317/C++-2-lines-O(4)-or-O-(1))

最后一种不太有趣的解法是随机。每次随机取两个元素，然后判断它们是否相等，不相等则继续找。

## 代码

### 排序

```cpp
class Solution {
public:
    int repeatedNTimes(vector<int>& A) {
        sort(A.begin(), A.end());
        for (int i = 1; i < A.size(); i++) {
            if (A[i-1] == A[i])
                return A[i];
        }
        return -1;
    }
};
```

### 计数

```cpp
class Solution {
public:
    int repeatedNTimes(vector<int>& A) {
        unordered_set<int> s;
        for (int x: A) {
            if (s.find(x) != s.end()) return x;
            s.insert(x);
        }
        return -1;
    }
};
```

### reduce

感觉还是写得有点复杂。不需要用这么多判断。

```cpp
class Solution {
public:
    int repeatedNTimes(vector<int>& A) {
        for (int i = 0; i < A.size() - 3; i++) {
            if (A[i] == A[i+1] || A[i] == A[i+2] || A[i] == A[i+3]) return A[i];
            if (A[i+1] == A[i+2] || A[i+1] == A[i+3]) return A[i+1];
            if (A[i+2] == A[i+3]) return A[i+2];
        }
        return -1;
    }
};
```

### 随机

```cpp
class Solution {
public:
    int repeatedNTimes(vector<int>& A) {
        int N = A.size();
        while (true) {
            int i1 = rand() % N, i2 = rand() % N;
            if (i1 == i2) continue;
            if (A[i1] == A[i2]) return A[i1];
        }
        return -1;
    }
};
```