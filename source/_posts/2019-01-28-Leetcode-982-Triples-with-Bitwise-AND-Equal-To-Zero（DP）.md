---
title: Leetcode 982. Triples with Bitwise AND Equal To Zero（DP）
urlname: leetcode-982-triples-with-bitwise-and-equal-to-zero
toc: true
date: 2019-01-28 16:16:26
updated: 2019-01-28 16:16:26
tags: [Leetcode, Leetcode Contest, alg:Dynamic Programming]
---

题目来源：[https://leetcode.com/problems/triples-with-bitwise-and-equal-to-zero/description/](https://leetcode.com/problems/triples-with-bitwise-and-equal-to-zero/description/)

标记难度：Hard

提交次数：1/1

代码效率：

* 我的做法：64.71%（876ms）
* DP法：82.35%（184ms）

## 题意

给定数组`A`（每个数在`[0, 2^16)`范围内），问所有满足`A[i] & A[j] & A[k] == 0`的三元组`(i, j, k)`的个数。

`0 <= A.length <= 1000`。

## 分析

比赛的时候我觉得这道题很难，所以没去做它。后来我就想了这样的一种做法：

* 用`O(N^2)`的时间复杂度枚举所有的二元组`(i, j)`与的结果，存入map中，进行计数
* 对于map中的每种可能结果，找到其中中每个为1的位都为0的`A[i]`的个数

总的时间复杂度是`O(min(N^2, 65536) * 16 * N)`。

事实上我觉得这种做法没什么道理，只是因为题目整体的复杂度实在很低才过的……

“与”这个操作让我想起了之前做的一道利用“或”的题目：[Leetcode 898. Bitwise ORs of Subarrays](/post/leetcode-898-bitwise-ors-of-subarrays)。这道题的核心思路之一是，将一个数或上另一个数不会使其中为1的位的数量减少，只会增加，所以不论或上多少个数，不同的数的最大的数量也只有32个。如果把这个思路搬到这道题中，就是将一个数与上另一个数，0的数量不会减少吧。不过这道题一共最多也只有3个数与在一起，这么想好像也没什么意思。

---

随便看了看题解区，感觉大部分解法都透着一股别扭劲，没什么特别值得一看的。不过这个动态规划解法还挺有趣的：

* 令`f[i][state]`表示从`A`中总共选出`i`个数，这`i`个数的与是`state`的情况的数量
* 初始化：`f[1][A[i]] = 1`
* 更新：`f[i][state & A[j]] += f[i-1][state]`

总复杂度为`O(3 * 2^16 * N)`。

[wangzi]: [Java DP O(3 * 2^16 * n) time O(n) space](https://leetcode.com/problems/triples-with-bitwise-and-equal-to-zero/discuss/226721/Java-DP-O%283-*-216-*-n%29-time-O%28n%29-space)

## 代码

### 我的方法

```cpp
class Solution {
public:
    int countTriplets(vector<int>& A) {
        bool isZero[16][1000];
        int N = A.size();
        map<int, int> cnt2;
        
        for (int i = 0; i < N; i++) {
            int x = A[i];
            for (int j = 0; j < 16; j++)
                isZero[j][i] = (x & (1 << j)) == 0;
        }
        for (int i = 0; i < N; i++)
            for (int j = 0; j < N; j++)
                cnt2[A[i] & A[j]]++;
        
        int ans = 0;
        for (const auto& i: cnt2) {
            int x = i.first;
            bool andZero[1000];
            memset(andZero, 1, sizeof(andZero));
            for (int j = 0; j < 16; j++) {
                if (x & (1 << j)) {
                    for (int k = 0; k < N; k++)
                        andZero[k] &= isZero[j][k];
                }
            }
            int cnt = 0;
            for (int j = 0; j < N; j++)
                if (andZero[j])
                    cnt++;
            ans += cnt * i.second;
        }
        return ans;
    }
};
```

### DP法

```cpp
class Solution {
public:
    int countTriplets(vector<int>& A) {
        int f[4][1 << 16];
        int N = A.size();
        memset(f, 0, sizeof(f));
        for (int i = 0; i < N; i++)
            f[1][A[i]]++;
        // 注意更新方式
        for (int i = 1; i < 3; i++) {
            for (int state = 0; state < (1 << 16); state++) {
                if (f[i][state] == 0) continue;
                for (int j = 0; j < N; j++)
                    f[i + 1][state & A[j]] += f[i][state];
            }
        }
        return f[3][0];
    }
};
```