---
title: Leetcode 962. Maximum Width Ramp
urlname: leetcode-962-maximum-width-ramp
toc: true
date: 2018-12-23 16:59:56
updated: 2018-12-24 02:16:00
tags: [Leetcode, Leetcode Contest, alg:Stack]
---

题目来源：[https://leetcode.com/problems/maximum-width-ramp/description/](https://leetcode.com/problems/maximum-width-ramp/description/)

标记难度：Medium

提交次数：2/3

代码效率：

* 排序：124ms
* 栈：68ms

## 题意

给定数组`A`，找到满足`i < j`且`A[i] <= A[j]`的`(i, j)`中`j - i`的最大值。如果没有则返回0。

## 分析

这道题我的做法是这样的：将数组排序（并记录原来的`index`），然后遍历数组，记录并不断更新已经出现过的`index`的最小值，将当前的`index[i]`减去该最小值即可求出所有可能的`(x, i)`中`i - x`的最大值。

做的时候因为没有判断好不存在的情况而错了一次。我好像经常因为这种问题犯错。

---

其实做的时候我隐约感觉到了也许可以用到栈（我之前在[Leetcode 84. Largest Rectangle in Histogram](/post/leetcode-84-largest-rectangle-in-histogram)）中详细总结了相关内容），但是比赛的时候没仔细思考。

如果需要用到栈，那么不妨把问题化归成这样：对于每个`j`，找到满足`A[i] <= A[j]`的最小的`i <= j`（如果这个`i`是存在的）。比如说，我们现在有一个`A[i]`，那么它右边的`A[i+1], A[i+2], ...`如果`>= A[i]`，则它们对结果是毫无意义的，因为我们需要尽量选择靠左的且值比较小的数，`A[i]`必然会是一个更好的选择。所以可以维护一个值递减的栈（这次不需要弹出了），然后对于每个元素，用二分查找的方式找到最好的candidate，并且在它比栈顶还小的情况下入栈。[^lee215]

[^lee215]: [lee215's solution for Leetcode 962 - \[Java/C++/Python\] O(N) Using Stack](https://leetcode.com/problems/maximum-width-ramp/discuss/208348/JavaC++Python-O%28N%29-Using-Stack)

很有趣的一点是，之前的问题都是“找到距离最近的元素”，所以直接用栈就可以维护局部性。这次要找到“距离尽量远的元素”，单用栈大概就不够了，需要加上二分查找。

## 代码

```cpp
class Solution {
public:
    int maxWidthRamp(vector<int>& A) {
        // 对于每个数，找到左边比它小的最靠左的数
        // 将index排序，找到左边最小的index
        vector<pair<int, int>> a;
        for (int i = 0; i < A.size(); i++)
            a.emplace_back(A[i], i);
        sort(a.begin(), a.end());
        int N = a.size();
        // 我好像经常犯这一类边界条件的错误。
        int minn = N + 1, ans = 0;
        for (int i = 0; i < N; i++) {
            ans = max(ans, a[i].second - minn);
            minn = min(minn, a[i].second);
        }
        return ans;
    }
};
```

### 栈

```cpp
class Solution {
public:
    int maxWidthRamp(vector<int>& A) {
        vector<pair<int, int>> s;
        int ans = 0;
        for (int i = 0; i < A.size(); i++) {
            if (s.empty() || s.back().first > A[i]) s.emplace_back(A[i], i);
            else {
                int l = 0, r = s.size();
                while (l < r) {
                    int mid = (l + r) / 2;
                    if (s[mid].first > A[i])
                        l = mid + 1;
                    else
                        r = mid;
                }
                if (l < s.size()) ans = max(ans, i - s[l].second);
            }
        }
        return ans;
    }
};
```
