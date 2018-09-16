---
title: Leetcode 907. Sum of Subarray Minimums（栈）
urlname: leetcode-907-sum-of-subarray-minimums
toc: true
date: 2018-09-16 16:35:21
updated: 2018-09-16 18:11:00
tags: [Leetcode, Leetcode Contest, alg:Stack]
---

题目来源：[https://leetcode.com/problems/sum-of-subarray-minimums/description/](https://leetcode.com/problems/sum-of-subarray-minimums/description/)

标记难度：Medium

提交次数：3/5

代码效率：

* BST：304ms
* 2 Stacks：60ms
* 1 Stack：84ms

## 题意

给定正整数数组`A`，求出`A`中所有连续子序列的最小值之和`mod (10^9 + 7)`。

## 分析

比赛后再看到题解时，我的内心是震惊的。没错，这道题又是一道和[Leetcode 901](/post/leetcode-901-online-stock-span)一样的题。比赛期间我使用了原来想到的那种`O(n * log(n))`的BST方法，完全没有意识到这道题和前一晚上学习的题解的相似性。于是我对自己的学习情况表示了深切的怀疑。

---

显然一种较好的思路是，对于`A`中的每一个值`A[i]`，寻找以它为最小值的子序列的数量。也就是说，我们需要寻找`A[i]`左侧第一个比它大的值的位置，以及右侧第一个比它大的值的位置。然后就可以计算对应的子序列的数量了。

显然这个寻找的过程和刚才说到的题是一样的，因此就不多写了。[^lee]

[^lee]: [\[C++/Java/Python\] Stack Solution](https://leetcode.com/problems/sum-of-subarray-minimums/discuss/170750/C++JavaPython-Stack-Solution)

这个过程可以简化为使用一个栈。对于被某个数从栈中弹出的数而言，它右侧第一个比它小的数就是这个数。所以我们可以对所有被弹出的数得到左侧的区间范围和右侧的区间范围。我觉得这是一种非常聪明的做法。[^onestack]

[^onestack]: [One stack solution](https://leetcode.com/problems/sum-of-subarray-minimums/discuss/170857/One-stack-solution)

## 代码

### BST

```cpp
class Solution {
public:
    int sumSubarrayMins(vector<int>& A) {
        vector<pair<int, int>> loc;
        int n = A.size();
        for (int i = 0; i < n; i++)
            loc.emplace_back(A[i], i);
        sort(loc.begin(), loc.end());
        set<int> locSet;

        long long int sum = 0;
        for (int i = 0; i < n; i++) {
            int index = loc[i].second;
            long long int val = loc[i].first;
            int left = 0;
            int right = 0;

            auto it = locSet.lower_bound(index);
            if (it != locSet.begin()) {
                --it;
                left = index - *it - 1;
            }
            else
                left = index;

            it = locSet.upper_bound(index);
            if (it != locSet.end()) {
                right = *it - index - 1;
            }
            else
                right = n - index - 1;

            sum += val * (left + 1) * (right + 1) % 1000000007;
            sum %= 1000000007;
            locSet.insert(index);
        }

        return sum;
    }
};
```

### 2 Stacks

```cpp
class Solution {
public:
    int sumSubarrayMins(vector<int>& A) {
        stack<pair<int, int>> leftStack, rightStack;
        int n = A.size();
        int left[n], right[n];

        leftStack.emplace(-1, -1);  // val, idx
        for (int i = 0; i < n; i++) {
            while (!leftStack.empty() && leftStack.top().first >= A[i])
                leftStack.pop();
            if (leftStack.top().second == -1)
                left[i] = i;
            else
                left[i] = i - leftStack.top().second - 1;

            leftStack.emplace(A[i], i);
        }

        rightStack.emplace(-1, -1);
        for (int i = n - 1; i >= 0; i--) {
            // 对于leftStack，此处写的是>=
            // 这意味着对于同一子序列中有多个最小元素的情况，
            // 我们选择把这种情况的值最右侧的最小元素中计算。
            // （防止重复计算）
            while (!rightStack.empty() && rightStack.top().first > A[i])
                rightStack.pop();
            if (rightStack.top().second == -1)
                right[i] = n - i - 1;
            else
                right[i] = rightStack.top().second - i - 1;

            rightStack.emplace(A[i], i);
        }

        long long int sum = 0;
        for (int i = 0; i < n; i++) {
            sum += (left[i] + 1) * (right[i] + 1) * (long long int) A[i] % 1000000007;
            sum %= 1000000007;
        }
        return sum;
    }
};
```

### 1 Stack

```cpp
class Solution {
public:
    int sumSubarrayMins(vector<int>& A) {
        stack<pair<int, int>> s;  // (ele, idx)
        long long int sum = 0;
        int n = A.size();
        for (int i = 0; i <= n; i++) {
            // 我经常想不清楚这个地方的符号。不过，当A[i] > 已有元素时，
            // 并不会影响那些元素作为最小值。这样想比较容易。
            if (s.empty() || i < n && s.top().first <= A[i])
                s.emplace(A[i], i);
            else {
                while (!s.empty() && (i == n || s.top().first > A[i])) {
                    int val = s.top().first;
                    int idx = s.top().second;
                    s.pop();
                    int left, right;
                    if (s.empty()) left = idx;
                    else left = idx - s.top().second - 1;
                    right = i - idx - 1;

                    sum += (long long) (left + 1) * (right + 1) * val % 1000000007;
                    sum %= 1000000007;
                }
                s.emplace(A[i], i);
            }
        }
        return sum;
    }
};
```
