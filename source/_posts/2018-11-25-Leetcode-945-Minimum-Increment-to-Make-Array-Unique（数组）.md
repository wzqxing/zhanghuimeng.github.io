---
title: Leetcode 945. Minimum Increment to Make Array Unique（数组），及周赛（112）总结
urlname: leetcode-945-minimum-increment-to-make-array-unique-and-weekly-contest-112
toc: true
date: 2018-11-25 13:40:38
updated: 2018-11-25 15:07:00
tags: [Leetcode, Leetcode Contest, alg:Array]
---

题目来源：[https://leetcode.com/problems/minimum-increment-to-make-array-unique/description/](https://leetcode.com/problems/minimum-increment-to-make-array-unique/description/)

标记难度：Medium

提交次数：2/2

代码效率：

* 排序：104ms
* 计数：44ms

## 题意

给定一个数组A，可以对其中的数进行+1操作，返回能够使得A中所有元素两两不同的操作数量。

## 分析

这次比赛的题全都很简单，但是第三题我没有想出来……所以最后排名是248 / 3194。基本上彻底是拼手速了。

---

比赛的时候我的思路就是，先把A排序，然后按顺序判断当前的数是否还没被用过……如果没被用过就记录“下一个没被用的数”；否则将它设置为“下一个没被用的数”，并将这个数+1。然后这样想还是有些复杂；事实上直接和上一个数进行比较就可以了……（因为排序了）。[^sort]如果用类似于计数排序可以更快一些。[^solution]我感觉这有一点贪心的思路。

[^sort]: [lee215's solution for Leetcode 945 - \[C++/Java/Python\] Straight Forward](https://leetcode.com/problems/minimum-increment-to-make-array-unique/discuss/197687/C++JavaPython-Straight-Forward)

[^solution]: [official solution for Leetcode 945](https://leetcode.com/problems/minimum-increment-to-make-array-unique/solution/)

## 代码

### 排序

```cpp
class Solution {
public:
    int minIncrementForUnique(vector<int>& A) {
        if (A.size() == 0) return 0;
        sort(A.begin(), A.end());
        int minn = A[0] + 1;
        long long int ans = 0;
        for (int i = 1; i < A.size(); i++) {
            if (A[i] < minn) {
                ans += minn - A[i];
                A[i] = minn++;
            }
            else
                minn = A[i] + 1;
        }
        return ans;
    }
};
```

### 计数

```cpp
class Solution {
public:
    int minIncrementForUnique(vector<int>& A) {
        int cnt[80005];
        memset(cnt, 0, sizeof(cnt));
        for (int x: A)
            cnt[x]++;
        long long int ans = 0;
        for (int i = 0; i < 80003; i++) {
            if (cnt[i] > 1) {
                cnt[i]--;
                ans += cnt[i];
                cnt[i + 1] += cnt[i];
                cnt[i] = 1;
            }
        }
        return ans;
    }
};
```