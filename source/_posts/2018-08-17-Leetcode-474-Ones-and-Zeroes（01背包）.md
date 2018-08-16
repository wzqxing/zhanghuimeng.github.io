---
title: Leetcode 474. Ones and Zeroes（01背包）
urlname: leetcode-474-ones-and-zeroes
toc: true
date: 2018-08-17 04:50:09
updated: 2018-08-17 05:01:09
tags: [Leetcode]
---

题目来源：[https://leetcode.com/problems/ones-and-zeroes/description/](https://leetcode.com/problems/ones-and-zeroes/description/)

标记难度：Medium

提交次数：1/2

代码效率：25.73%

## 题意

有两种资源（0和1）的01背包。（倒是很符合“01”之义。）

## 分析

没什么好说的咯……因为是01背包，所以时间复杂度应该是无法优化的，为`O(m * n * K)`；但是空间复杂度可以通过滚动优化降低到`O(m * n)`。

我有时候很好奇，为什么动态规划的复杂度会这么高，无法进行更好的优化。我想，可能是因为我们要求的是最优解，所以必须遍历整个解空间，动态规划只是用一种聪明的方式进行了剪枝而已。

提交中Runtime Error了一次。经过试验，我发现，在Leetcode这里，在栈上定义`526 * 100 * 100`的int数组是会爆栈的，他们的栈空间到底是开了多大啊？我在网上查了一圈也没有查到。反正后来就换成滚动数组了。

## 代码

```cpp
class Solution {
public:
    int findMaxForm(vector<string>& strs, int m, int n) {
        // f[N][i][j]：在前N个字符串中，用i个0，j个1最多能组成多少个串
        int N = strs.size(), digitCnt[N+1][2];
        memset(digitCnt, 0, sizeof(digitCnt));
        for (int i = 0; i < N; i++) {
            for (char c: strs[i]) {
                if (c == '0')
                    digitCnt[i+1][0]++;
                else
                    digitCnt[i+1][1]++;
            }
        }
        // cout << N << ' ' << m << ' ' << n << endl;
        int f[2][m+1][n+1], ans = 0;
        // seems 526 100 100 is not ok
        memset(f, 0, sizeof(f));
        // 对k进行滚动优化时，必须把k放在最外层循环
        for (int k = 1; k <= N; k++) {
            for (int i = 0; i <= m; i++) {
                for (int j = 0; j <= n; j++) {
                    if (i < digitCnt[k][0] || j < digitCnt[k][1])
                        f[k % 2][i][j] = f[(k-1) % 2][i][j];
                    else
                        f[k % 2][i][j] = max(f[(k-1) % 2][i][j], f[(k-1) % 2][i-digitCnt[k][0]][j-digitCnt[k][1]] + 1);
                    ans = max(ans, f[k % 2][i][j]);

                    // cout << i << ' ' << j << ' ' << k << ' ' << f[k % 2][i][j] << endl;
                }
            }
        }
        return ans;
    }
};
```
