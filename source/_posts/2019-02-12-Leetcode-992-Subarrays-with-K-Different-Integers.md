---
title: Leetcode 992. Subarrays with K Different Integers
urlname: Leetcode 992. Subarrays with K Different Integers
toc: true
date: 2019-02-12 17:30:55
updated: 2019-02-14 00:00:05
tags: [Leetcode, Leetcode Contest]
categories: Leetcode
---

题目来源：[https://leetcode.com/problems/subarrays-with-k-different-integers/description/](https://leetcode.com/problems/subarrays-with-k-different-integers/description/)

标记难度：Hard

提交次数：1/1

代码效率：

* 两个滑动窗口：28.89%（340ms）
* 一个滑动窗口：

## 题意

给定数组`A`，求`A`中恰好含有`k`个不同元素的子数组的数量。

## 分析

从数据范围来看，显然`N^2`的算法是过不了的。所以比赛的时候我就想了一种这样的算法：

* 遍历数组元素
* 用指针`i`和`j`分别表示从当前元素开始，含有`k`个不同元素的最短子数组的结尾元素和最长子数组的结尾元素
* 用两个`map`分别维护这两个子数组中的元素
* 需要移动到下一个元素时，从`map`中删除当前元素，并移动指针直到满足要求为止。显然`i`和`j`的位置是递增的

这就相当于维护了两个滑动窗口。显然每个元素最多进出每个集合一次，所以整体复杂度应该是`O(N)`。（虽然常数实在很大……）

这个算法应该和[题解](https://leetcode.com/problems/subarrays-with-k-different-integers/solution/)是很相似的。

---

当然别人还有一些更神奇的做法，比如[lee215的另辟蹊径的方法](https://leetcode.com/problems/subarrays-with-k-different-integers/discuss/234482/JavaC++Python-Sliding-Window-with-Video)。他做了一件这样的事情：

* 对于某个`K`，遍历数组元素
* 对于每个`A[j]`，找到使得`A[i...j]`中恰好有`K`个不同元素的最大的`i`，记`j-i+1`为以`j`结尾的最多有`K`个不同元素的子数组的数量
* 令`f(K)`表示数组中最多有`K`个不同元素的子数组的数量
* 则所求结果为`f(K) - f(K-1)`

还真是一个独到的想法，虽然有些不明白他是怎么想出来的……

---

还有一种更有趣的方法[^vot]，没有显式地维护两个滑动窗口。我觉得这种方法的核心思路其实是这样的：

* 对于每一个数组元素`A[i]`，记`sMin`为使得`A[sMin...i]`中包含`K`个不同元素的最小index，`sMax`为使得`A[sMax...i]`中包含`K`个不同元素的最大index
* 那么显然`A[sMin...i]`和`A[sMax...i]`包含的元素是一样的（虽然可能个数不同）
* 考虑从`A[i]`转移到`A[i+1]`的情况：
  * 如果`A[i+1]`是窗口中已经出现过的元素，则`sMin`不变，`sMax`可能会减小
  * 如果`A[i+1]`是窗口中没有出现过的元素，则`sMin = sMax + 1`，`sMax >= sMin`
* 所以维护`sMax`对应的小窗口就够了。

[^vot]: [votrubac's solution - C++ with picture, 7 lines 56 ms](https://leetcode.com/problems/subarrays-with-k-different-integers/discuss/235235/C++-with-picture-7-lines-56-ms)

---

以及，有一个可以大幅度提速的优化。题目里给了数组元素的范围（`<=N`），因此可以用数组代替`map`。

## 代码

### 两个滑动窗口

```cpp
class Solution {
public:
    int subarraysWithKDistinct(vector<int>& A, int K) {
        map<int, int> beginMap;
        map<int, int> endMap;
        int ans = 0;
        int end1, end2 = A.size() - 1;
        for (int i = 0; i < A.size(); i++) {
            if (beginMap.size() < K) {
                beginMap[A[i]]++;
                endMap[A[i]]++;
                if (beginMap.size() == K) end1 = i;
            }
            else {
                if (endMap.find(A[i]) == endMap.end()) {
                    end2 = i - 1;
                    break;
                }
                endMap[A[i]]++;
            }
        }
        if (beginMap.size() < K) return 0;
        // end1和end2是两个窗口结尾的指针
        ans += end2 - end1 + 1;
        for (int i = 1; i < A.size(); i++) {
            beginMap[A[i - 1]]--;
            endMap[A[i - 1]]--;
            if (beginMap[A[i - 1]] == 0) beginMap.erase(A[i - 1]);
            if (endMap[A[i - 1]] == 0) endMap.erase(A[i - 1]);
            while (end1 < A.size() - 1) {
                if (beginMap.size() == K) {
                    break;
                }
                end1++;
                beginMap[A[end1]]++;
            }
            if (beginMap.size() < K) break;
            while (end2 < A.size() - 1) {
                if (endMap.size() < K || endMap.size() == K && endMap.find(A[end2 + 1]) != endMap.end()) {
                    end2++;
                    endMap[A[end2]]++;
                }
                else break;
            }
            if (endMap.size() < K) break;
            ans += end2 - end1 + 1;
        }
        return ans;
    }
};
```

### 一个滑动窗口

```cpp
class Solution {
public:
    int subarraysWithKDistinct(vector<int>& A, int K) {
        int sMin = 0, sMax = 0;
        unordered_map<int, int> window;
        int ans = 0;
        for (int i = 0; i < A.size(); i++) {
            // sMin不变，只修改sMax
            if (window.size() < K || window.find(A[i]) != window.end()) {
                window[A[i]]++;
                while (window.size() == K && window[A[sMax]] > 1) {
                    window[A[sMax]]--;
                    sMax++;
                }
            }
            // sMin和sMax都改变
            else {
                window[A[i]]++;
                sMin = sMax + 1;
                while (window.size() >= K) {
                    if (window.size() == K && window[A[sMax]] == 1) break;
                    window[A[sMax]]--;
                    if (window[A[sMax]] == 0) window.erase(A[sMax]);
                    sMax++;
                }
            }
            if (window.size() == K) ans += sMax - sMin + 1;
        }
        return ans;
    }
};
```