---
title: Leetcode 978. Longest Turbulent Subarray
urlname: leetcode-978-longest-turbulent-subarray
toc: true
date: 2019-01-20 17:05:26
updated: 2019-01-20 21:19:00
tags: [Leetcode, Leetcode Contest, alg:Array]
---

题目来源：[https://leetcode.com/problems/longest-turbulent-subarray/description/](https://leetcode.com/problems/longest-turbulent-subarray/description/)

标记难度：Easy

提交次数：1/1

代码效率：124ms

## 题意

给定一个正整数数组`A`，求其中满足turbulent性质的最长子数组的长度：

* 满足`A[i] > A[i+1] < A[i+2] > ... A[j]`
* 或`A[i] < A[i+1] > A[i+2] < ... A[j]`

（其实就是要上下波动。）

## 分析

我比赛时的想法很简单：维护两个数组`up`和`down`，`up[i]`表示以`i`为结尾且`A[i-1]<A[i]`的turbulent子数组最大长度，`down[i]`表示以`i`为结尾且`A[i-1]>A[i]`的turbulent子数组最大长度，然后交替进行递推。

其实这种做法有一些冗余，因为显然只有`A[i-1]<A[i]`时`up[i]`才有意义（否则肯定只能取1）；`down[i]`同理。题解里给出了一种维护这些关系的方法：把原来的数组丢掉，只留下两个数之间的相对关系。对于数组`A = [9,4,2,10,7,8,8,1,9]`，令`B`为比较的结果，记大于号为1，小于号为-1，等号为0。因为`A[0]=9 > 4=A[1]`，因此`B[0]=1`；因为`A[1]=4 > 2=A[2]`，因此`B[1]=1`；因为`A[2]=2 < 10=A[3]`，因此`B[2]=-1`；……[^sln]

[^sln]: [Leetcode Offical Solution for 978. Longest Turbulent Subarray](https://leetcode.com/problems/longest-turbulent-subarray/solution/)

以此类推，得到`B = [1, 1, -1, 1, -1, 0, -1, 1]`。此时我们只需要贪心地尝试把`B`分成若干个最长的1和-1交替的子序列即可，如`[1], [1, -1, 1, -1], [0], [-1, 1]`。因为此时`B`中的每个数表示的是`A`中两个数的关系，所以边缘处（重复）的数得到了比较好的处理：如`[1]`代表`[9, 4]`，`[1, -1, 1, -1]`代表`[4, 2, 10, 7, 8]`，`[0]`代表`[8, 8]`，`[-1, 1]`代表`[8, 1, 9]`。

## 代码

```cpp
class Solution {
public:
    int maxTurbulenceSize(vector<int>& A) {
        int n = A.size();
        int up[n], down[n];
        int ans = 0;
        up[0] = down[0] = 1;
        for (int i = 1; i < n; i++) {
            up[i] = down[i] = 1;
            if (A[i] > A[i-1])
                up[i] = down[i-1] + 1;
            if (A[i] < A[i-1])
                down[i] = up[i-1] + 1;
        }
        for (int i = 0; i < n; i++) {
            ans = max(ans, up[i]);
            ans = max(ans, down[i]);
        }
        return ans;
    }
};
```
