---
title: Leetcode 888. Fair Candy Swap（数组），及周赛（98）总结
urlname: leetcode-888-fair-candy-swap-and-weekly-contest-98
toc: true
date: 2018-08-19 15:24:34
updated: 2018-08-19 16:28:34
tags: [Leetcode, LeetcodeContest]
---

题目来源：[https://leetcode.com/problems/fair-candy-swap/description/](https://leetcode.com/problems/fair-candy-swap/description/)

标记难度：Easy

提交次数：3/4

代码效率：

* 二分查找：76ms（暂缺数据）
* HashSet：<del>136ms</del>112ms

## 题意

给定两个数组，要求交换两个数，使得两个数组各自的总和相等。

## 分析

第二次参加比赛，名次比上回差了一些（362 / 3552），主要是因为第一题耗时太多了。也罢，我目前大概就是这个Leetcode周赛能稳定做出前三题的水平了，第四题基本做不出来。这对于一个参加过NOIP和省选的人来说的确是太菜了，但是不承认这一点并不能就让我变得更不菜。所以还是承认这个事实，然后尽量在此基础上有所进步比较好。

我在11:27时才提交成功第1题，之前还wa了一次。为什么会wa呢？看到这道题之后，我的第一想法是，很显然，我们可以直接计算出要交换的两个数的差值：`delta = (Sb - Sa) / 2`。然后我们要做的就是寻找具有这样差值的数对。于是我就“很自然地”把B数组排了个序，然后遍历A数组中的每一个数`x`，在B中用二分查找的方法寻找`x + delta`这个数是否存在。这种做法是正确的，复杂度是`O((A.len + B.len) + A.len * log(B.len))`（求和+遍历+二分查找）。我用`std::lower_bound`实现二分查找，但却忘了`lower_bound`的语义**不是**返回指向范围中首个等于`value`的迭代器，**而是**“返回指向范围`[first, last)`中首个不小于（即大于或等于）`value`的元素的迭代器，或若找不到这种元素则返回`last`”[^lowerbound]。用人话说，就是我忘了检查`lower_bound`到底真的找到对应元素没有……

[^lowerbound]: [std::lower_bound](https://zh.cppreference.com/w/cpp/algorithm/lower_bound)

修了bug之后就对了，但是实际上根本就不需要二分查找[^solution]。既然你要找的是一个固定的数，那么直接用HashSet不就好了？！复杂度降低到了`O(A.len + B.len)`。

[^solution]: [Solution](https://leetcode.com/problems/fair-candy-swap/solution/)

所以说Leetcode数据还是弱，卡一下复杂度的话，`n * log(n)`的算法根本不应该过的。

## 在set中查找元素？

但是为什么用了`unordered_set`之后，效率一下子低了那么多？经过查找资料，我认为是`std::count`和`std::find`的性能差距导致的问题。`std::find`只要找到元素就会返回；而`std::count`总是需要遍历所有元素，所以会比较慢[^performance]。把`count`换成`find`之后，快了大约20ms，验证了上述说法。

[^performance]: [Performance differences between std::count and std::find](https://stackoverflow.com/questions/28099506/performance-differences-between-stdcount-and-stdfind)

## 代码

### 二分查找

```cpp
class Solution {
public:
    vector<int> fairCandySwap(vector<int>& A, vector<int>& B) {
        // 那还是先sort一下，然后sum一下，找出A和B的差值
        // 然后对A中的每一个数用这个差值二分查找
        sort(A.begin(), A.end());
        sort(B.begin(), B.end());
        int sumA = 0, sumB = 0;
        for (int x: A)
            sumA += x;
        for (int x: B)
            sumB += x;
        int delta = sumB - sumA;
        delta >>= 1;
        for (int x: A) {
            int toFind = x + delta;
            // cout << x << ' ' << toFind << endl;
            auto it = lower_bound(B.begin(), B.end(), toFind);
            if (it != B.end() && *it == toFind) {  // 需要检查的
                vector<int> ans = {x, toFind};
                return ans;
            }
        }
        return {};
    }
};
```

### HashSet

```cpp
class Solution {
public:
    vector<int> fairCandySwap(vector<int>& A, vector<int>& B) {
        int sumA = 0, sumB = 0;
        unordered_set<int> bset;
        for (int x: A)
            sumA += x;
        for (int x: B) {
            sumB += x;
            bset.insert(x);
        }
        int delta = (sumB - sumA) / 2;
        for (int x: A) {
            if (bset.find(x + delta) != bset.end()) {
                // bset.count(x + delta) > 0
                return {x, x + delta};  // 这么写居然是可以的
            }
        }
        return {};
    }
};
```
