---
title: Leetcode 891. Sum of Subsequence Widths（子序列；ad hoc）
urlname: leetcode-891-sum-of-subsequence-widths
toc: true
date: 2018-08-19 17:55:20
updated: 2018-08-19 20:02:00
tags: [Leetcode, LeetcodeContest]
---

题目来源：[https://leetcode.com/problems/sum-of-subsequence-widths/description/](https://leetcode.com/problems/sum-of-subsequence-widths/description/)

标记难度：Hard

提交次数：1/1

代码效率：64ms

## 题意

给定数组`A`，考虑`A`的所有非空子序列。对于每个子序列，定义它的`width`为子序列中最大值-最小值。求`A`的所有非空子序列的`width`之和模`1000000007`。

## 分析

这是周赛的第4题。我比赛的时候花了40分钟也没想出来，但其实我感觉我已经接近正确答案了！我考虑了一会儿怎么用二分之类的方法之后，感觉针对`width`和序列本身去二分是不可取的，不如转而考虑每个元素会在多少个子序列里做最大值，在多少个子序列里做最小值。思路是正确的，但是我开始的时候看错题了，看成是subarray而非subsequence了……所以最后没做完……

对于子序列而言，顺序不是很重要，所以首先可以把数组排序。[^intuition]（这真是一个珍贵的直觉。）这之后，对于元素`A[i]`，有`i`个比它更小的元素，所以在`2 ^ i`个子序列中，它是最大的元素；有`n - i - 1`个比它更大的元素，所以在`2 ^ (n - i - 1)`个子序列中，它是最大的元素。所以，就`A[i]`而言，`result += (2^i - 2^(n-i-1)) * A[i]`。在实际求解过程中，为了避免重复求2的幂次，可以把上述代码按2的幂次进行拆分。[^solution]

[^intuition]: [Solution](https://leetcode.com/problems/sum-of-subsequence-widths/solution/)

[^solution]: [C++/Java/1-line Python Sort and One Pass](https://leetcode.com/problems/sum-of-subsequence-widths/discuss/161267/C++Java1-line-Python-Sort-and-One-Pass?page=2)

之前我对重复元素的情况有些疑虑，但现在想来并不是必要的。在上述方法中，我们对每个子序列都恰好考虑了两遍：一遍是开头元素，一遍是结尾元素。即使子序列中存在重复的元素，也并不会影响其中的最大值和最小值的数值。

以及，因为A中数值的范围是确定的（`[1, 20000]`），我们可以采用计数排序的方法来对A进行排序。

## 代码

```cpp
class Solution {
private:
    void bucketSort(vector<int>& A) {
        int bucket[20005];
        memset(bucket, 0, sizeof(bucket));
        for (int x: A)
            bucket[x]++;
        int i = 0;
        for (int j = 1; j <= 20000; j++) {
            while (bucket[j] > 0) {
                A[i++] = j;
                bucket[j]--;
            }
        }
    }

public:
    int sumSubseqWidths(vector<int>& A) {
        bucketSort(A);
        int n = A.size();
        int modulo = 1000000007;
        long long int pow[20005];
        pow[0] = 1;
        for (int i = 1; i <= n; i++)
            pow[i] = (pow[i - 1] << 1) % modulo;
        long long int sum = 0;
        for (int i = 0; i < n; i++) {
            sum += ((pow[i] - pow[n-i-1] + modulo) % modulo) * A[i];
            sum %= modulo;
        }
        return sum;
    }
};
```
