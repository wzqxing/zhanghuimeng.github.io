---
title: Leetcode 955. Delete Columns to Make Sorted II（贪心）
urlname: leetcode-955-delete-columns-to-make-sorted-ii
toc: true
date: 2018-12-09 16:54:51
updated: 2018-12-09 17:07:51
tags: [Leetcode, Leetcode Contest, alg:Greedy]
---

题目来源：[https://leetcode.com/problems/delete-columns-to-make-sorted-ii/description/](https://leetcode.com/problems/delete-columns-to-make-sorted-ii/description/)

标记难度：Medium

提交次数：1/2

代码效率：8ms

## 题意

给定`N`个长度相同的字符串，问从中最少删掉多少列，才能使得删除后的字符串是按字典序排列的。

## 分析

这道题还挺有趣的。题解里给了一种很明显是stay ahead思路的正确性证明[^solution]：

* 如果当前列会导致之前留下的列加上这一列不满足字典序，则这一列必须删除
* 否则可以说明，留下当前列必然不比删除它更差
* 由上述说明可以得知，已经留下的列必然是满足字典序的，其中有一些满足严格字典序（大于），有一些则可能不是严格字典序（相等）
* 对于已经满足严格字典序的字符串，后面加什么都没问题
* 对于相等的字符串，后面加的第一列必须满足字典序
* 因此，加上当前这一列之后，相等的字符串可能会变成严格字典序，或者还是相等
* 也就是说，相等的字符串不会增加，添加新列的难度也不会增加
* 所以应该尽可能留下当前列

[^solution]: [LeetCode Official Solution for 955. Delete Columns to Make Sorted II](https://leetcode.com/problems/delete-columns-to-make-sorted-ii/solution/)

（我写的不好，还是看题解的例子吧，那个例子举得很好。）

## 代码

但是我觉得我的代码写得还挺巧妙的。

```cpp
class Solution {
public:
    int minDeletionSize(vector<string>& A) {
        int n = A.size(), m = A[0].size();
        int ans = 0;
        vector<string> B;
        for (int i = 0; i < n; i++) {
            B.push_back("");
        }
        for (int i = 0; i < m; i++) {
            bool isOk = true;
            for (int j = 1; j < n; j++) {
                if (B[j] == B[j-1] && A[j][i] < A[j-1][i]) {
                    isOk = false;
                    ans++;
                    break;
                }
            }
            if (isOk) {
                for (int j = 0; j < n; j++)
                    B[j] += A[j][i];
            }
        }
        return ans;
    }
};
```