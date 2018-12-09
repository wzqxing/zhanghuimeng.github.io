---
title: Leetcode 956. Tallest Billboard（DP）
urlname: leetcode-956-tallest-billboard
toc: true
date: 2018-12-09 19:44:33
updated: 2018-12-09 20:36:00
tags: [Leetcode, Leetcode Contest, alg:Dynamic Programming, alg:Meet in the Middle]
---

题目来源：[https://leetcode.com/problems/tallest-billboard/description/](https://leetcode.com/problems/tallest-billboard/description/)

标记难度：Hard

提交次数：1/2

代码效率：

* 动态规划：136ms
* 中间相遇：184ms

## 题意

给定<=20根棍子，要求从中拿出一些，分成两组，使得每组的棍子长度之和相等。保证所有棍子的长度之和最大是5000。

## 分析

在比赛的时候我能想到的最好的解法是这样的：

* 枚举第一组都有哪些棍子（`2^20`）
* 对于剩下的棍子，用动态规划的方法判断能否组成第一组已有的长度（`5000*20`）

显然这个算法是会超时的。但是也许它已经接近正解了。

---

### 解法1：-1,0,1背包

把这个问题看成是扩展版的01背包问题：对于每根棍子，我们可以把它加入背包中，不加入背包中，还可以把它从背包中减去。

令`f[i][j]`表示用前`i`根棍子能否组成和为`j`的长度（`j`有可能是负的）。则`f[i+1][j] = f[i][j] || f[i][j-rods[i+1]] || f[i][j+rods[i+1]]`。为了找到可能的最大长度，用辅助数组`g[i][j]`记录`f[i][j]`为真时，最大可能的正长度之和。算法的复杂度为`O(10000*N)`。[^wangzi]

[^wangzi]: [wangzi6147's Solution for LeetCode 956 - Java knapsack O(N*sum)](https://leetcode.com/problems/tallest-billboard/discuss/203261/Java-knapsack-O%28N*sum%29)

### 解法2：中间相遇法

把`rods`数组分成大致相等的两半，然后对每一半都枚举每根棍子是+，-还是0。然后对于左边的一半得到的和，在右边寻找这个和的负值是否存在。最后取最大值。算法复杂度为`O(3^(N/2))`。

这个算法也需要记录最大可能的正长度之和。

## 代码

### 解法1：扩展01背包

```cpp
class Solution {
public:
    int tallestBillboard(vector<int>& rods) {
        bool f[20][10001];  // f[i][j+5000]（index负数的一种方法）
        int g[20][10001];
        memset(f, 0, sizeof(f));
        memset(g, 0, sizeof(g));
        int ans = 0;
        for (int i = 0; i < rods.size(); i++) {
            f[i][5000] = true;
            for (int j1 = 0; j1 <= 10000; j1++) {
                int j = j1 - 5000;  // j才是真正的j，j1是j+5000
                if (i == 0) {
                    if (j == rods[i]) {
                        f[i][j1] = true;
                        g[i][j1] = rods[i];
                    }
                    if (j == -rods[i]) f[i][j1] = true;
                }
                else {
                    // 三种情况：分别更新g，找到最大值
                    if (f[i-1][j1]) {
                        f[i][j1] = true;
                        g[i][j1] = max(g[i][j1], g[i-1][j1]);
                    }
                    if (j1 >= rods[i] && f[i-1][j1-rods[i]]) {
                        f[i][j1] = true;
                        g[i][j1] = max(g[i][j1], g[i-1][j1-rods[i]] + rods[i]);
                    }
                    if (j1 + rods[i] <= 10000 && f[i-1][j1+rods[i]]) {
                        f[i][j1] = true;
                        g[i][j1] = max(g[i][j1], g[i-1][j1+rods[i]]);
                    }
                }
            }
            ans = max(ans, g[i][5000]);
        }
        return ans;
    }
};
```

### 解法2：中间相遇法

```cpp
class Solution {
private:
    unordered_map<int, int> results;  // 和 - 最大正值和
    int M, N;
    int ans;
    
    // 枚举棍子状态：sum是和，p是正值和
    void dfs(int x, int sum, int p, vector<int>& rods, bool check) {
        if (x >= rods.size()) {
            if (!check) results[sum] = max(results[sum], p);
            else {
                if (results.find(-sum) != results.end())
                    ans = max(ans, results[-sum] + p);
            }
            return;
        }
        dfs(x+1, sum, p, rods, check);
        dfs(x+1, sum+rods[x], p+rods[x], rods, check);
        dfs(x+1, sum-rods[x], p, rods, check);
    }
    
public:
    int tallestBillboard(vector<int>& rods) {
        N = rods.size();
        if (N == 0) return 0;
        M = N / 2;
        vector<int> rod1, rod2;
        for (int i = 0; i < M; i++)
            rod1.push_back(rods[i]);
        for (int i = M; i < N; i++)
            rod2.push_back(rods[i]);
        ans = 0;
        dfs(0, 0, 0, rod1, false);
        dfs(0, 0, 0, rod2, true);
        return ans;
    }
};
```