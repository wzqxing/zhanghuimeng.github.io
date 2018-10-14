---
title: Leetcode 923. 3Sum With Multiplicity（数组）
urlname: leetcode-923-3sum-with-multiplicity
toc: true
date: 2018-10-14 16:41:00
updated: 2018-10-14 17:24:00
tags: [Leetcode, Leetcode Contest, alg:Array, alg:Two Pointers]
---

题目来源：[https://leetcode.com/problems/3sum-with-multiplicity/description/](https://leetcode.com/problems/3sum-with-multiplicity/description/)

标记难度：Medium

提交次数：3/7

代码效率：

* 计数：8ms
* 2Sum：4ms

## 题意

给定整数数组`A`和`target`，返回满足`i < j < k, A[i] + A[j] + A[k] == target`的`(i, j, k)`数量。其中`3 <= A.length <= 3000`，`0 <= A[i] <= 100`，`0 <= target <= 300`。

## 分析

比赛的时候这道题挂了三次：

* 没有判断`k`是否在`[0, 100]`范围内，所以RE了
* 忘记`A[i]`的下限是0了，所以WA了
* 数量计算的时候爆int了，所以又WA了

……

---

显然最暴力的做法就是直接枚举，时间复杂度是`O(N^3)`。当然，这样太暴力了。有两个主要的优化方向：

* 根据`0 <= A[i] <= 100`进行去重
* 根据2Sum或者[3Sum](https://leetcode.com/problems/3sum/description/)的思路进行指针优化

由于数据规模的原因，去重的思路是很容易想到的。至于指针优化，它的思路大概是这样的：如果已经确定了`(i, j)`，在`i`不变，`j`递增的情况下，如果合法的`k`存在，则`k`必然是递减的。所以只需要枚举`(i, j)`并通过这一方法确定`k`即可。当然，在本题使用去重的情况下，只要在hash表中直接查找`target - i - j`元素是否存在即可；不过这一思路仍然很有趣。[^solution]

[^solution]: [Leetcode 923 - Official Solution](https://leetcode.com/problems/3sum-with-multiplicity/solution/)

## 代码

### 计数

```cpp
class Solution {
public:
    int threeSumMulti(vector<int>& A, int target) {
        int n = A.size();
        long long int cnt[101];
        memset(cnt, 0, sizeof(cnt));
        for (int i: A) cnt[i]++;
        long long int ans = 0, MOD = 1000000007;
        // different: i <= j <= k
        for (int i = 0; i <= min(100, target); i++) {
            if (!cnt[i]) continue;
            for (int j = i; j <= 100 && i + j <= target && j <= target - i - j; j++) {
                int k = target - i - j;
                if (k > 100) continue;
                if (!cnt[j] || !cnt[k]) continue;
                if (i != j && j != k)
                    ans = (ans + cnt[i] * cnt[j] * cnt[k]) % MOD;
                else if (i != j && j == k && cnt[j] >= 2)
                    ans = (ans + cnt[i] * cnt[j] * (cnt[j] - 1) / 2) % MOD;
                else if (i == j && j != k && cnt[i] >= 2)
                    ans = (ans + cnt[i] * (cnt[i] - 1) * cnt[k] / 2) % MOD;
                else if (i == j && j == k && cnt[i] >= 3)
                    ans = (ans + cnt[i] * (cnt[i] - 1) * (cnt[i] - 2) / 6) % MOD;
            }
        }
        return ans;
    }
};
```

### 2Sum

```cpp
class Solution {
public:
    int threeSumMulti(vector<int>& A, int target) {
        int n = A.size();
        int cnt[101];
        long long int value[101];
        int key[101], m;
        memset(cnt, 0, sizeof(cnt));
        for (int i: A) cnt[i]++;

        m = 0;
        for (int i = 0; i <= 100; i++) {
            if (cnt[i]) {
                key[m] = i;
                value[m++] = cnt[i];
            }
        }

        long long int ans = 0, MOD = 1000000007;
        // different: i <= j <= k
        for (int i = 0; i < m; i++) {
            int k = m - 1;
            for (int j = i; j <= k; j++) {
                while (j <= k && key[i] + key[j] + key[k] > target) k--;
                if (j > k) break;
                if (key[i] + key[j] + key[k] < target) continue;
                if (i < j && j < k) ans += value[i] * value[j] * value[k] % MOD;
                else if (i == j && j < k) ans += (value[i] * (value[i] - 1) * value[k] / 2) % MOD;
                else if (i < j && j == k) ans += (value[i] * value[j] * (value[j] - 1) / 2) % MOD;
                else if (i == j && j == k) ans += (value[i] * (value[i] - 1) * (value[i] - 2) / 6) % MOD;
                ans %= MOD;
            }
        }
        return ans;
    }
};
```
