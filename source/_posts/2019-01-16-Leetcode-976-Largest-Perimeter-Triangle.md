---
title: Leetcode 976. Largest Perimeter Triangle
urlname: leetcode-976-largest-perimeter-triangle
toc: true
date: 2019-01-16 02:21:37
updated: 2019-01-16 22:54:00
tags: [Leetcode, Leetcode Contest, alg:Math]
---

题目来源：[https://leetcode.com/problems/largest-perimeter-triangle/description/](https://leetcode.com/problems/largest-perimeter-triangle/description/)

标记难度：Easy

提交次数：1/1

代码效率：20.00%（64ms）

## 题意

给定`N`个正数（`N<=10000`），求以这些正数作为边长能组成的周长最大的三角形的周长。

## 分析

我第一次看的时候看成了面积，好在及时纠偏了。（问题：如果是面积，那应该怎么做呢？）

显然，用`O(N^3)`的方法（枚举每种组合成三角形的方式）可做，但是肯定会超时。那么不妨考虑只选两条边，然后用某种方法选出最大的第三条边的过程。（虽然`O(N^2)`也会超时，但是姑且先这么做着。）

不妨先将数组`A`排序。如果先尝试选出较小的两条边`A[i]`和`A[j]`（`i < j`），那么第三条边`A[k]`需要满足`A[j]-A[i] < A[k] < A[i]+A[j]`。那么肯定是选择小于`A[i]+A[j]`的最大的`A[k]`。而且我们希望`A[i]+A[j]`尽量大……

用上面这种思路肯定是可以做出来的，因为它本质上和这一种一样，不过我觉得这样更好想一些。先尝试选择较大的两条边`A[j]`和`A[k]`，则`A[i]`需要满足`A[k]-A[j] < A[i] < A[j]+A[k]`。因为`A[i]`是较小的边，因此必然满足`A[i] < A[j]+A[k]`，于是只需考虑`A[i] > A[k]-A[j]`这一条件。在选定`A[k]`之后，选择最大的比它小的`A[j]`（也就是`A[k-1]`）可以同时最大化这条边的边长和`A[i]`，因此只需对于每个`k >= 2`，找出满足`A[i] > A[k]-A[k-1] && i < k-1`的最大的`i`。

这种做法的复杂度是`O(N*log(N))`，排序是`O(N*log(N))`，对于每个`k`求upper-bound也是`O(N*log(N))`。

显然后一个`O(N*log(N))`有点儿蠢。如果满足`A[i] > A[k]-A[k-1]`的最小的`i`小于`k-1`，那么无论如何都应该选择`A[i] = A[k-2]`；否则就找不到满足条件的三角形。所以直接看`A[k]`、`A[k-1]`和`A[k-2]`就可以了！

事实上，按照题解的说法：假设三条边的边长是`a, b, c`，且满足`a <= b <= c`，那么这三条边能组成三角形的充要条件是`a + b > c`。对于每个`c`，考虑最大的`a, b`就足够了！[^sln]

[^sln]: [Leetcode Official Solution for 976](https://leetcode.com/articles/largest-perimeter-triangle/)

## 代码

这里是比较愚蠢的做法……

```cpp
class Solution {
public:
    int largestPerimeter(vector<int>& A) {
        int n = A.size();
        int ans = 0;
        sort(A.begin(), A.end());
        for (int i = n - 2; i > 0; i--) {
            int thes = A[i+1] - A[i];
            int idx = upper_bound(A.begin(), A.end(), thes) - A.begin();
            if (idx < n && idx < i)
                ans = max(ans, A[i+1] + A[i] + A[i - 1]);
        }
        return ans;
    }
};
```