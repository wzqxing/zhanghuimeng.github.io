---
title: Leetcode 896. Monotonic Array，及周赛（100）总结
urlname: leetcode-896-monotonic-array-and-weekly-contest-100
toc: true
date: 2018-09-02 14:16:06
updated: 2018-09-02 14:24:00
tags: [Leetcode, LeetcodeContest, Array]
---

题目来源：[https://leetcode.com/problems/monotonic-array/description/](https://leetcode.com/problems/monotonic-array/description/)

标记难度：Easy

提交次数：1/1

代码效率：96ms

## 题意

判断一个数组是否是递增或递减的。

## 分析

这次比赛的排名继续稳步下降（1393 / 4008），我意识到我可能只能稳定做出Easy来，难一点的Medium就做不出来了。当然事实上有的时候也许Hard还会简单一点。比赛时第二题的题目描述出了问题，导致我心态很崩，并在上面花了很长时间和很多罚时。然后还看错了第三题的题面。最后一题本来有希望推导出来，但是没有时间了。

反正这道题只写了3分钟，还行吧。

---

水题，水到难以分类（Array……？），直接扫描两遍分别判断递增和递减就可以了。当然也可以压缩为一遍，但没有什么本质区别。

## 代码

```cpp
class Solution {
public:
    bool isMonotonic(vector<int>& A) {
        if (A.size() <= 1)
            return true;
        bool mono = true;
        for (int i = 1; i < A.size(); i++) {
            if (A[i] < A[i - 1]) {
                mono = false;
                break;
            }
        }
        if (mono)
            return true;
        mono = true;
        for (int i = 1; i < A.size(); i++) {
            if (A[i] > A[i - 1]) {
                mono = false;
                break;
            }
        }
        return mono;
    }
};
```
