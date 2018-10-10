---
title: Leetcode 918. Maximum Sum Circular Subarray（动态规划）
urlname: leetcode-918-maximum-sum-circular-subarray
toc: true
date: 2018-10-07 20:23:26
updated: 2018-10-11 00:31:26
tags: [Leetcode, Leetcode Contest, alg:Array]
---

题目来源：[https://leetcode.com/problems/maximum-sum-circular-subarray/description/](https://leetcode.com/problems/maximum-sum-circular-subarray/description/)

标记难度：Medium

提交次数：4/6

代码效率：

* 前缀和+堆：356ms
* 半限定的拆分：32.68%
* 前缀和+双端队列：104ms
* 最大和+最小和：38.06%

## 题意

有一个循环数组，求非空子序列的最大可能值（子序列不能和自己重叠）。

## 分析

比赛的时候我没找到正解，WA了两次。然后匆匆想了一个我觉得能过的`O(N * log(N))`的算法交上去了。

算法是这样的，思路挺简单：把数组复制一份到后面（循环），得到`B = A + A`，然后计算`B`的前缀和数组`P`。遍历`P`，用一个堆（实际上是`map`，因为可能有重复元素，而且还要求最小值）维护`P[max(i - N, 0)] ~ P[i-1]`的前缀和，从中找出最小的（记为`P[j]`），就可以求出以`i`结尾的子序列的最大的和，为`P[i] - P[j]`（以及需要考虑`i < N`时取整个前缀的情况）。最后取所有子序列的和的最大值即可。

### 前缀和+双端队列

我这个思路没什么太大的问题，但是用堆对前缀和进行维护实际上是不必要的——

事实上，又遇到一道[用栈求最大值](/post/leetcode-84-largest-rectangle-in-histogram)一类的问题了，真是惊人……

这次我们需要做的是用一个`deque`保存一系列`index`，它们对应的前缀和是从小到大递增的。当`index = i`时，如果`deque.front() < i - N`，则将队头弹出。然后队头就是目前范围内的最小前缀和了。之后在队列非空且`deque.back() >= B[i]`时不断弹出队尾。最后将`i`插入到队尾。[^solution]

[^solution]: [Leetcode Solution - 918. Maximum Sum Circular Subarray](https://leetcode.com/problems/maximum-sum-circular-subarray/solution/)

我的第一个问题是将队头弹出之后队列是否会变空？答案是不会，因为`i-1`此时刚被插入队列，它是不会被弹出来的，所以队列不会为空。

第二个问题是这个算法为何正确。它和之前使用栈的想法很类似，差别是之前求的是每个元素左（右）侧比它大（小）的元素，现在求的是一个连续范围内的最小值。所以这个算法维护了一个以该连续范围的右界为结尾的且在连续范围内的严格递增的序列，序列开头处的值就是符合要求的最小值。但是怎么说明这样做的道理呢……

还是题解里说得好：若`i < j`且`P[i] >= P[j]`，则`i`在此时无论如何都没有用处了。

### 半限定的拆分

在非循环的情况下，显然我们都会做这道题：令`f[i]`表示以`i`结尾的子序列的最大和，则`f[i] = max(0, f[i-1]) + A[i]`。在循环的情况下，仍然可以尝试这么做。令`g[i]`表示以`i`结尾的**循环**子序列的最大和（只计算循环的）；此时，`g[i] = max(A[j: A.length-1] + A[0, i])`，其中`j < i`。由于我们规定了`g[i]`必须是一个循环子序列的和，因此可以把上式改写为`g[i] = max(A[j: A.length-1]) + (A[0] + ... + A[i]), j < i`。

则此时可以开始递推了。令`T[j] = A[j] + A[j+1] + ... + A[A.length-1]`，`R[j] = max(T[k]) (k >= j)`，则`g[i] = R[i+1] + (A[0] + ... + A[i])`。[^solution]

一个细节是，题解里写的实际上是`g[i] = R[i+2] + (A[0] + ... + A[i])`。我觉得这是因为使用`R[i+1]`时可能会得到整个数组（`A[i+1] + ... + A[A.length-1] + A[0] + ... + A[i]`），这是没有必要的；不过这好像也不会增加什么复杂度。

### 最大和+最小和

这是一种很妙的想法。题解里写了两个贪心算法的变种：sign变种和min变种，但我觉得它们本质上是差不多的。对于循环子序列`A[0] + ... + A[i] + A[j] + ... + A[A.length-1]`，可以把它写成`sum(A) - (A[i+1] + ... + A[j-1])`，也就意味着我们现在只需要求出一个和最小的子序列，然后用整个数列的和减去这个最小的和，就可以得到循环子序列的最大和了。（虽然其中大概包含了一些边界情况）

需要注意一种特殊情况：和最小的子序列是整个数组。这种情况只会发生在整个数组都是负数的时候，因此需要特判。题解里的做法要稍微更麻烦一些，我觉得没有必要。[^solution]

## 代码

### 前缀和+堆

```cpp
class Solution {
public:
    int maxSubarraySumCircular(vector<int>& A) {
        int n = A.size();
        long long int p[2 * n];
        long long int a[2 * n];
        for (int i = 0; i < n; i++)
            a[i] = a[i + n] = A[i];
        p[0] = a[0];
        for (int i = 1; i < 2 * n; i++)
            p[i] = p[i - 1] + a[i];

        map<long long int, int> m;
        long long int ans = a[0];
        m[p[0]]++;
        for (int i = 1; i < 2 * n; i++) {
            if (i >= n) {
                m[p[i - n]]--;
                if (m[p[i - n]] == 0) m.erase(p[i - n]);
            }
            long long int minVal = m.begin()->first;
            ans = max(ans, p[i] - minVal);
            if (i < n) ans = max(ans, p[i]);
            m[p[i]]++;
        }
        return ans;
    }
};
```

### 前缀和+双端队列

```cpp
class Solution {
public:
    int maxSubarraySumCircular(vector<int>& A) {
        int n = A.size();
        long long int p[2 * n + 1];
        long long int a[2 * n + 1];
        for (int i = 1; i <= n; i++)
            a[i] = a[i + n] = A[i - 1];
        p[0] = 0;
        for (int i = 1; i <= 2 * n; i++)
            p[i] = p[i - 1] + a[i];

        long long int ans = a[1];
        deque<long long int> q;
        q.push_back(0);
        for (int i = 1; i <= 2 * n; i++) {
            while (q.front() < i - n) q.pop_front();
            ans = max(ans, p[i] - p[q.front()]);
            while (!q.empty() && p[q.back()] >= p[i]) q.pop_back();
            q.push_back(i);
        }
        return ans;
    }
};
```

### 半限定的拆分

```cpp
class Solution {
public:
    int maxSubarraySumCircular(vector<int>& A) {
        int n = A.size();
        if (n == 1) return A[0];

        // Don't forget the uncircular intervals!
        long long int p[n], t[n], f[n], ans = A[0];
        f[0] = A[0];
        for (int i = 1; i < n; i++) {
            f[i] = max(f[i-1], (long long int) 0) + A[i];
            ans = max(ans, f[i]);
        }

        // The uncircular intervals.
        p[0] = A[0];
        for (int i = 1; i < n; i++)
            p[i] = p[i - 1] + A[i];
        t[n - 1] = A[n - 1];
        long long int r = t[n - 1];
        for (int i = n - 2; i >= 0; i--) {
            ans = max(ans, p[i] + r);
            t[i] = t[i + 1] + A[i];
            r = max(r, t[i]);
        }
        return ans;
    }
};
```

### 最大和+最小和

```cpp
class Solution {
public:
    int maxSubarraySumCircular(vector<int>& A) {
        int n = A.size();
        if (n == 1) return A[0];

        // f: max sum; g: min sum; minSum: min of g
        long long int f[n], g[n], ans = A[0], minSum = A[0], sum = A[0];
        f[0] = g[0] = A[0];
        // not necessary - can directly test if ans < 0 in the end.
        bool hasPos = A[0] >= 0;
        for (int i = 1; i < n; i++) {
            sum += A[i];
            hasPos = hasPos ? hasPos : A[i] >= 0;
            f[i] = max(f[i-1], (long long int) 0) + A[i];
            g[i] = min(g[i-1], (long long int) 0) + A[i];
            ans = max(ans, f[i]);
            minSum = min(minSum, g[i]);
        }

        if (!hasPos) return ans;
        return max(ans, sum - minSum);
    }
};
```
