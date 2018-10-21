---
title: Leetcode 926. Flip String to Monotone Increasing（字符串）
urlname: leetcode-926-flip-string-to-monotone-increasing
toc: true
date: 2018-10-21 15:23:04
updated: 2018-10-21 15:48:00
tags: [Leetcode, Leetcode Contest, alg:Array, alg:Greedy]
---

题目来源：[https://leetcode.com/problems/flip-string-to-monotone-increasing/description/](https://leetcode.com/problems/flip-string-to-monotone-increasing/description/)

标记难度：Medium

提交次数：2/2

代码效率：

* 普通的做法：12ms
* 神奇的做法：8ms

## 题意

有一个只包含0和1的串，希望把里面的一些位翻转，使得这个串变成连续的若干个0（可以没有）后跟着连续的若干个1（也可以没有）。问翻转次数最少是多少。

## 分析

一个立即可以想到的思路是枚举。枚举每一位作为1的起始点，然后根据这一位左边的1的个数和右边的0的个数算出总的翻转次数，然后取最小值。前缀中1的个数和后缀中0的个数都很好维护。

另一个相当神奇的思路[^vortubac]应用了贪心法。在从左向右扫描的过程中，我们动态维护这个分割点，分割点左侧1的数量，分割点右侧1的数量和0的数量；一旦发现分割点右侧0的数量超过了1的数量，说明此时把分割点直接右移到当前扫描点可以减小翻转次数。举个例子：

`S = "00011000"`

| `i` | virtual split point | left 1 cnt | right 0 cnt | right 1 cnt |
| --- | ---- | ---- | ---- | ---- |
| 0 | 0 (`| 00011000`) | 0 | 0 | 0 |
| 1 | 1 (`0| 0011000`) | 0 | 0 | 0 |
| 2 | 2 (`00| 011000`) | 0 | 0 | 0 |
| 3 | 3 (`000| 11000`) | 0 | 0 | 0 |
| 4 | 3 (`000|1 1000`) | 0 | 0 | 1 |
| 5 | 3 (`000|11 000`) | 0 | 0 | 2 |
| 6 | 3 (`000|110 00`) | 0 | 1 | 2 |
| 7 | 3 (`000|1100 0`) | 0 | 2 | 2 |
| 8 | 8 (`00011000| `) | 2 | 0 | 0 |

这个算法的正确性怎么证明……好吧，懒得去想了。大概应该用反证法。

[^vortubac]: [Leetcode 926 Solution by vortubac - C++ 4 lines O(n) | O(1), DP](https://leetcode.com/problems/flip-string-to-monotone-increasing/discuss/183851/C++-4-lines-O%28n%29-or-O%281%29-DP)

## 代码

### 普通的做法

```cpp
class Solution {
public:
    int minFlipsMonoIncr(string S) {
        int n = S.size();
        if (n == 0) return 0;
        int totZeros = 0, totOnes = 0;
        // actually, ones array can be omitted.
        int ones[n];  // num of ones <= i
        for (int i = 0; i < n; i++) {
            if (S[i] == '0') {
                totZeros++;
                ones[i] = i == 0 ? 0 : ones[i - 1];
            }
            else {
                totOnes++;
                ones[i] = i == 0 ? 1 : ones[i - 1] + 1;
            }
        }
        int ans = n;
        for (int i = 0; i < n; i++) { // flip [i, ) to be 1
            int leftOnes = i == 0 ? 0 : ones[i - 1];
            int leftFlip = leftOnes;
            int rightFlip = n - i - (totOnes - leftOnes);

            // cout << i << ' ' << leftOnes << ' ' << leftFlip << ' ' << rightFlip << endl;
            ans = min(ans, leftFlip + rightFlip);
        }

        // flip all to be zero
        ans = min(totOnes, ans);

        return ans;
    }
};
```

### 神奇的做法

```cpp
class Solution {
public:
    int minFlipsMonoIncr(string S) {
        int leftOnes = 0, rightOnes = 0, rightZeros = 0;
        for (int i = 0; i < S.size(); i++) {
            if (S[i] == '0') rightZeros++;
            else rightOnes++;
            if (rightZeros > rightOnes) {  // move split point to i
                leftOnes += rightOnes;
                rightOnes = rightZeros = 0;
            }
        }
        return leftOnes + rightZeros;
    }
};
```
