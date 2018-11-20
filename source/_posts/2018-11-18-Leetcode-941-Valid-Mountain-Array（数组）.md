---
title: Leetcode 941. Valid Mountain Array（数组），及周赛（111）总结
urlname: leetcode-941-valid-mountain-array-and-weekly-contest-111
toc: true
date: 2018-11-18 15:28:04
updated: 2018-11-18 21:46:04
tags: [Leetcode, Leetcode Contest, alg:Array]
---

题目来源：[https://leetcode.com/problems/valid-mountain-array/description/](https://leetcode.com/problems/valid-mountain-array/description/)

标记难度：Easy

提交次数：2/4

代码效率：

* two-pass：40ms
* one-pass：24ms

## 题意

给定数组`A`，判断`A`是否是一个合法的mountain array：

* `A.length >= 3`
* 存在`0 < i < A.length - 1`使得：
  * `A[0] < A[1] < ... < A[i-1] < A[i]`
  * `A[i] > A[i + 1] > ... > A[A.length - 1]`

## 分析

这次比赛的排名是410 / 3585。前三题都意想不到地简单；第四题的DP有点难想，但我觉得也许给我更多的时间我就能做出来。

然后这道题因为误以为“题目保证`A.length>=3`”而RE了一次（实际上应该是`A.length>=3`的输入才需要考虑）。

---

比赛的时候我整了一个比较复杂的做法：用`increasing[i]`数组表示是否满足`A[0] < A[1] < ... < A[i-1] < A[i]`，`decreasing[i]`数组表示是否满足`A[i] > A[i + 1] > ... > A[A.length - 1]`，然后寻找是否有同时满足两个条件的`i`。

上述做法显然太复杂了。实际上直接找到最大的满足`A[0] < A[1] < ... < A[i-1] < A[i]`的`i`然后判断`A[i] > A[i + 1] > ... > A[A.length - 1]`是否成立即可。题解中也是这种做法。[^solution]

[^solution]: [Leetcode official solution for 941. Valid Mountain Array](https://leetcode.com/problems/valid-mountain-array/solution/)

## 代码

One-pass版本，因为忘记判断`0 < i < n - 1`，WA了一次。

```cpp
class Solution {
public:
    bool validMountainArray(vector<int>& A) {
        int n = A.size();
        if (n < 3) return false;
        int i = 0;
        while (A[i] < A[i + 1] && i < n - 1) i++;
        if (i == 0 || i == n - 1) return false;
        while (i < n - 1) {
            if (A[i] <= A[i + 1]) return false;
            i++;
        }
        return true;
    }
};
```