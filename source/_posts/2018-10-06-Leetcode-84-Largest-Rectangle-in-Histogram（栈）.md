---
title: Leetcode 84. Largest Rectangle in Histogram（栈）
urlname: leetcode-84-largest-rectangle-in-histogram
toc: true
date: 2018-10-06 01:36:34
updated: 2018-10-06 01:36:34
tags: [Leetcode, alg:Stack]
---

题目来源：[https://leetcode.com/problems/largest-rectangle-in-histogram/description/](https://leetcode.com/problems/largest-rectangle-in-histogram/description/)

标记难度：Hard

提交次数：1/3

代码效率：

* O(N^2)递推：超时
* 单栈：98.96%

## 题意

给定一个直方图的高度（或者说给定一排宽度都为1的矩形柱子的高度），找出图中包含的面积最大的矩形。

## 分析

显然可以有这样的直觉：所能找到的面积最大的矩形必定与某个矩形柱子的高度是相等的，否则它的高度必然小于它覆盖的所有柱子，因此高度还可以再升高，直到到达某个柱子的高度为止。因此问题可以转化为，对于每个柱子，它左边和右边各有多少个连续的柱子的高度不小于它？不妨记这两个值为`left[i]`和`right[i]`，则以第`i`个柱子为最大高度的矩形的面积就是`(left[i] + right[i] + 1) * heights[i]`。

我的第一个解法是错误的：试图直接通过前一个柱子的`left[i-1]`推导出下一个柱子的`left[i]`。显然这两者并不存在一个直接的推导关系，所以我WA了。

于是我立刻想出了第二种解法——用`O(N^2)`的复杂度直接计算`left`和`right`数组。题目里没给`N`的范围，不过我还是TLE了。

这时候我才想起来，我已经见过三道类似的题目了，它们分别是：

* [Leetcode 901](/post/leetcode-901-online-stock-span/)
* [Leetcode 739](/post/leetcode-739-daily-temperatures/)
* [Leetcode 907](/post/leetcode-907-sum-of-subarray-minimums/)

它们所需要完成的任务是相似的：给定一个数组，为其中的每个数求出它左边/右边/both的第一个比它大/小的数的位置。这种算法我已经详细分析过了，所以现在不妨来比较一下这几道题的区别，特别是有重复数字时的边界问题的处理。下面用`A`表示数组：

* [Leetcode 901](/post/leetcode-901-online-stock-span/)：求左侧第一个`> A[i]`的值。
* [Leetcode 739](/post/leetcode-739-daily-temperatures/)：求右侧第一个`> A[i]`的值。
* [Leetcode 907](/post/leetcode-907-sum-of-subarray-minimums/)：在问题的转换过程中需要自行确定边界：
  * 计算“以`A[i]`为最小值的子序列的数量”之前，就需要考虑一个子序列中有多个最小值的情况怎么办。显然一个子序列只能被作为其中一个最小值所对应的子序列计算一次。所以问题是应该选择哪一个最小值作为“最小值”。显然比较方便的办法是取最左边或最右边的最小值。
  * 如果取的是最左边的最小值，则对于每个值，需要求的是左侧第一个`<= A[i]`的值，和右侧第一个`< A[i]`的值；
  * 反之，则需要求左侧第一个`< A[i]`的值，和右侧第一个`<= A[i]`的值。
* [Leetcode 84](/post/leetcode-84-largest-rectangle-in-histogram)：求左侧第一个`< A[i]`的值，和右侧第一个`< A[i]`的值。

在[Leetcode 907](/post/leetcode-907-sum-of-subarray-minimums/)的分析中已经提到，有两种做法，一种是用两次迭代分别求左侧和右侧的值；另一种是直接用一次迭代。这种方法很聪明，但面临着一个问题：它无法实现左右两侧都严格不等或都不严格不等的情况。考虑用这一算法实现的[907题](/post/leetcode-907-sum-of-subarray-minimums/)，假如在栈中，`A[i]`将要被`A[j]`弹出了：

* 如果条件是`A[i] >= A[j]`，则对于`A[j]`，它将找到左侧第一个`< A[j]`的值，但对于`A[i]`，它找到的是右侧第一个`<= A[i]`的值
* 如果条件是`A[i] > A[j]`，则`A[j]`找到的是左侧第一个`<= A[j]`的值，但`A[i]`找到的是右侧第一个`< A[i]`的值

我想这是算法本身的限制。（也许可以改进，但我现在不太想思考了）所以严格来说，单栈算法对本题之前的模型是不太合适的。

但是我还是用了单栈的算法，因为可以把模型改一下。假如最大的矩形碰到了至少两个柱子的边界，说明对于这些柱子，它们都能计算得到对应的矩形面积（按照之前的算法）。那么只要保证其中至少一根柱子在计算的时候得到正确结果即可（其他的可以不管）。于是问题就转化成类似于[907题](/post/leetcode-907-sum-of-subarray-minimums/)的情况了。

## 代码

### O(N^2)递推

```cpp
class Solution {
public:
    int largestRectangleArea(vector<int>& heights) {
        int n = heights.size();
        if (n == 0) return 0;
        if (n == 1) return heights[0];

        vector<int> sorted(heights);
        sort(sorted.begin(), sorted.end());
        long long int f[n];
        memset(f, 0, sizeof(f));
        long long int ans = 0;
        // for each rectangle
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                if (sorted[j] > heights[i])
                    f[j] = 0;
                else {
                    f[j]++;
                    ans = max(ans, f[j] * sorted[j]);
                }
            }
        }

        return ans;
    }
};
```

### 单栈

```cpp
class Solution {
public:
    int largestRectangleArea(vector<int>& heights) {
        int n = heights.size();
        if (n == 0) return 0;
        if (n == 1) return heights[0];

        stack<pair<int, int>> s; // height, pos
        // Note: the left and right arrays are defined slightly differently
        // - They denote positions, not length (not a big deal though)
        long long int left[n], right[n], ans = 0;
        for (int i = 0; i < n; i++) {
            while (!s.empty() && s.top().first > heights[i]) {
                right[s.top().second] = i;
                s.pop();
            }
            if (s.empty()) left[i] = -1;
            else left[i] = s.top().second;

            s.emplace(heights[i], i);
        }
        while (!s.empty()) {
            right[s.top().second] = n;
            s.pop();
        }

        for (int i = 0; i < n; i++)
            ans = max(ans, (right[i] - left[i] - 1) * heights[i]);

        return ans;
    }
};
```
