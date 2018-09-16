---
title: Leetcode 902. Numbers At Most N Given Digit Set（DP）
urlname: leetcode-902-numbers-at-most-n-given-digit-set
toc: true
date: 2018-09-16 00:16:06
updated: 2018-09-16 11:44:00
tags: [Leetcode, Leetcode Contest, alg:Recursive, alg:Math]
---

题目来源：[https://leetcode.com/problems/numbers-at-most-n-given-digit-set/description/](https://leetcode.com/problems/numbers-at-most-n-given-digit-set/description/)

标记难度：Hard

提交次数：1/1

代码效率：

* 递推：47.81%
* 数学：100.00%

## 题意

给定数字集合`D`（即`D`是集合`{'1','2','3','4','5','6','7','8','9'}`的子集）和正整数`N`，问用这些数字共能组成多少`<=N`的数。

## 分析

我想，第一个问题是，这到底是一个统计问题，还是一个计算问题。考虑到`N`的范围，显然不能直接遍历所有数进行统计，而是需要进行某种程度的计算。也许会涉及到组合数。（虽然事实是没有）

第二个问题是正着做还是反着做——计算**只**包含这些数字的数的个数，还是计算**不只**包含这些数字的数的个数？显然我比赛的时候完全没有想清楚这一点，但就这道题来说，大概计算**包含**更简单一点。

最后一个问题是怎么算。显然我比赛的时候懵逼了，没想出来。“包含X数字的数”也许不算是一种常见的题型，但用非统计的方法计算在某个范围内符合某条件的数的数量似乎还挺常见的（如[Leetcode 866](https://leetcode.com/problems/prime-palindrome/description/)）。但我觉得这些题目之间的共性并不大。

---

### 递推方法

如何寻找`<=N`的合法的数？显然可以根据数的长度进行分类讨论。[^solution]对于长度小于`len(N)`的那些，显然`D`中数字的任意组合均是合法的（这一点我之前居然没有意识到……）；对于长度和`len(N)`相等的那些，则需要分类讨论：

* 如果该数首位比`N`的首位小，则之后的数字可以任意组合；如果该数首位和`N`的首位相等（当然前提是`N`的首位也是一个合法数字），则需要继续讨论第2位
* 如果该数第2位比`N`的第2位小，则之后的数字可以任意组合；如果该数第2位和`N`的第2位相等（当然前提是`N`的第2位也是一个合法数字），则需要继续讨论第3位
* 以此类推……

于是我们就得到了一个递推算法：令`f[i]`表示`<= N[i:]`的数的数量（`i`从1开始编号），则

```
f[i] = (number of digits < N[i]) * (D.size)^(len - i) + (N[i] in D) * f[i + 1]
```

且最终的结果为

```
f[1] + (D.size)^(len - 1) + (D.size)^(len - 2) + ... + D.size
```

### 数学方法

这是一种很神奇的方法。我们不妨把`D`中的数字看成是一种新的进位表示。比如，当`D = {1, 3, 5}`时，就可以把`[11, 13, 51, 53]`这类的数写成`[11, 12, 31, 32]`。如果能够知道`<= N`的最大的这种进位表示，就可以直接计算合法的数的数量了。所以剩下的问题就是怎么找到这个进位表示。

一种方法是根据`N`的位，从高位到低位分别进行讨论。令`B`表示所需的进位表示，`1 <= B[i] <= D.size, 1 <= i <= len(N)`：

* 如果`N[i] in D && N[i] = D[j]`，则`B[i] = j`
* 如果`N[i] > min(D)`，则`B[i] = lower_bound(D, N[i])`，`B[i+1:] = len(D)`，结束
* 如果`N[i] < min(D)`，则`B[i] = 0`，并对之前的数执行类似于退位的操作，`B[i+1:] = len(D)`，结束

其他情况是比较显然的。一个需要退位的例子：`D = {2, 4, 6, 8}`，`N = 41265`

1. `B[1] = 2, "4"`
2. `B[2] = 0`；执行退位操作后，`B[1] = 1, B[2] = 4, B[3:5] = 4, "28888"`

找到这个进位表示之后就可以直接计算出比它小的数的数量了。事实上这种表示方法类似于[Leetcode 171](/post/leetcode-171-excel-sheet-column-number/)，同样没有0的位置。[^solution]

[^solution]: [Leetcode 902 Solution](https://leetcode.com/problems/numbers-at-most-n-given-digit-set/solution/)

## 代码

### 递推

```cpp
class Solution {
private:
    // 递归快速幂
    long long quickpow(long long a, int x) {
        if (x == 0) return 1;
        long long int b = quickpow(a, x >> 1);
        return x % 2 == 0 ? b * b : b * b * a;
    }

public:
    int atMostNGivenDigitSet(vector<string>& D, int N) {
        bool valid[10];
        memset(valid, 0, sizeof(valid));
        for (string str: D)
            valid[str[0] - '0'] = true;

        int x = 0;
        for (int i = 0; i < 10; i++)
            if (valid[i]) x++;

        // 递推过程
        long long int f = 1;
        int digits = 0;
        while (N > 0) {
            if (!valid[N % 10])
                f = 0;
            int y = 0;
            for (int i = 0; i < N % 10; i++)
                if (valid[i]) y++;
            f += y * quickpow(x, digits);

            N /= 10;
            digits++;
        }

        for (int i = 1; i < digits; i++)
            f += quickpow(x, i);

        return f;
    }
};
```

### 数学

```cpp
class Solution {
public:
    int atMostNGivenDigitSet(vector<string>& D, int N) {
        vector<int> digits;
        while (N > 0) {
            digits.push_back(N % 10);
            N /= 10;
        }
        reverse(digits.begin(), digits.end());
        vector<int> digSet;
        for (string str: D)
            digSet.push_back(stoi(str));

        int n = digits.size();
        int B = D.size();
        vector<int> b(n, 0);
        for (int i = 0; i < digits.size(); i++) {
            int k;
            // 本来用的是set，后来我实在搞不明白lower_bound和distance那套理论了
            // 反正数据量比较小……
            for (k = 0; k < digSet.size(); k++)
                if (digits[i] < digSet[k])
                    break;

            // digit[i] < min(D)
            if (k == 0) {
                b[i] = 0;
                // 退位
                for (int j = i; j > 0; j--) {
                    if (b[j] == 0) {
                        b[j-1]--;
                        b[j] = B;
                    }
                    else break;
                }
                for (int j = i + 1; j < n; j++)
                    b[j] = B;
                break;
            }
            // digit[i] > min(D) with no match
            else if (digSet[k - 1] < digits[i]) {
                b[i] = k;
                for (int j = i + 1; j < n; j++)
                    b[j] = B;
                break;
            }
            // find match
            else if (digSet[k - 1] == digits[i])
                b[i] = k;
        }

        long long int cnt = 0;
        for (int d: b) {
            cnt = cnt * B + d;
        }
        return cnt;
    }
};
```
