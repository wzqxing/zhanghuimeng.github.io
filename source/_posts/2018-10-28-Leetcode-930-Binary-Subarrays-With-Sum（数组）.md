---
title: Leetcode 930. Binary Subarrays With Sum（数组）
urlname: 2018-10-28-930. Binary Subarrays With Sum（数组）
toc: true
date: 2018-10-28 15:03:17
updated: 2018-10-28 15:41:00
tags: [Leetcode, Leetcode Contest, alg:Array]
---

题目来源：[https://leetcode.com/problems/binary-subarrays-with-sum/description/](https://leetcode.com/problems/binary-subarrays-with-sum/description/)

标记难度：Medium

提交次数：2/2

代码效率：

* 场上乱写的：108ms
* 连续0：20ms
* 前缀和：32ms

## 题意

给定一个只包含0和1的数组A，问A中有多少个和为`S`的非空子序列？

## 分析

Leetcode最近放题解的速度明显变慢了。这道题并不是很好实现（如果没有想到前缀和的方法的话），所以场上WA了一次。

---

### 连续0

这种方法是我在现场想到的：对于一个和为S的非空子序列，它的两侧可能有连续的0，也可能没有。我们可以从两侧都是1的合法的子序列出发，生成对应的所有两侧可能多出来一些0的子序列的数量。这种方法的一个缺点是需要特判`S = 0`的情形。

我看到的一种比较巧妙的方法是，记录所有1之间的间隔里的0的数量，然后就可以直接枚举和为S的子序列了。[^evilnearby]

例如：对于序列`1 0 0 1 0 1`，记录数组会变成`zeros = [0, 2, 1, 0]`；当`S = 2`时，可以直接通过`(zeros[0] + 1) * (zeros[2] + 1)`和`(zeros[1] + 1) * (zeros[3] + 1)`计算出所有合法子序列的数量。

[^evilnearby]: [Leetcode 930 Solution by Evilnearby - Java O(N) Time O(N) Space, counting contiguous zeros](https://leetcode.com/problems/binary-subarrays-with-sum/discuss/186693/Java-O%28N%29-Time-O%28N%29-Space-counting-contiguous-zeros)

### 前缀和

直接用一个map存储前缀和，然后在里面查找对应的值的数量。看起来是十分容易想到的做法，不过我并没有想到。[^lee215]

[^lee215]: [Leetcode 930 Solution by lee215 - \[C++/Java/Python\] Straight Forward](https://leetcode.com/problems/binary-subarrays-with-sum/discuss/186683/C++JavaPython-Straight-Forward)

## 代码

（场上乱写的就不放了）

### 连续0

```cpp
class Solution {
public:
    int numSubarraysWithSum(vector<int>& A, int S) {
        vector<int> zeros;
        int cnt = 0;
        for (int x: A) {
            if (x == 1) {
                zeros.push_back(cnt);
                cnt = 0;
            }
            else
                cnt++;
        }
        zeros.push_back(cnt);

        int ans = 0;
        if (S == 0) {
            for (int x: zeros)
                ans += x * (x + 1) / 2;
            return ans;
        }
        for (int i = 0; i + S < zeros.size(); i++)
            ans += (zeros[i] + 1) * (zeros[i + S] + 1);
        return ans;
    }
};
```

### 前缀和

```cpp
class Solution {
public:
    int numSubarraysWithSum(vector<int>& A, int S) {
        unordered_map<int, int> prefixSum;
        int sum = 0;
        int ans = 0;
        prefixSum[0]++;
        for (int x: A) {
            sum += x;
            ans += prefixSum[sum - S];
            prefixSum[sum]++;
        }
        return ans;
    }
};
```
