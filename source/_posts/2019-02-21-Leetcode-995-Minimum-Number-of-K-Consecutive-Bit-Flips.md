---
title: Leetcode 995. Minimum Number of K Consecutive Bit Flips
urlname: leetcode-995-minimum-number-of-k-consecutive-bit-flips
toc: true
date: 2019-02-21 00:04:12
updated: 2019-02-22 00:30:00
tags: [Leetcode, Leetcode Contest, alg:Greedy]
categories: Leetcode
---

题目来源：[https://leetcode.com/problems/minimum-number-of-k-consecutive-bit-flips/description/](https://leetcode.com/problems/minimum-number-of-k-consecutive-bit-flips/description/)

标记难度：Hard

提交次数：4/5

代码效率：

* 暴力贪心：12.68%（5152ms）
* 线段树：52.11%（288ms）
* O(n)：96.20%（84ms）

## 题意

有一个只包含0和1的数组`A`，每次可以将数组`A`中的恰好连续`k`个元素取反，问能否将`A`变成全1？

## 分析

很容易就能想到一种贪心策略：找到左边第一个0，从它开始翻转连续`k`个元素（因为这个0只能在这里通过翻转而变成1，否则左边就会出现新的0），然后对于右边的0，继续进行这一操作，如果最后能成功则`A`可以变成全1。这个算法是`O(n*k)`的，或者说是`O(n^2)`的，可能太慢了。

很容易就能想到一种优化策略：用线段树来进行区间更新。这样复杂度就会变成`O(n*log(n))`，可以接受。

用线段树实在是overkill了。可以很简单地通过记录区间的开闭事件来确定当前元素是否需要取反。甚至可以不显式地记录开闭事件——如果`k`个元素之前的元素是`0`，那么现在就是一个区间的关闭。[^lee]

[^lee]: [les215's solution - \[Java/C++/Python\] One Pass and O(1) Space](https://leetcode.com/problems/minimum-number-of-k-consecutive-bit-flips/discuss/238609/JavaC++Python-One-Pass-and-O(1)-Space)

## 代码

### 暴力贪心

```cpp
class Solution {
public:
    int minKBitFlips(vector<int>& A, int K) {
        int ans = 0;
        for (int i = 0; i < A.size(); i++) {
            if (A[i] == 0) {
                if (i + K - 1 >= A.size()) return -1;
                for (int j = 0; j < K; j++)
                    A[i + j] = 1 - A[i + j];
                ans++;
            }
        }
        return ans;
    }
};
```

### 线段树

当然，这是一种比较愚蠢的做法。

```cpp
class Solution {
private:
    struct TreeNode {
        int l, r;
        bool toFlip;
        
        TreeNode(int _l = 0, int _r = 0) {
            l = _l;
            r = _r;
            toFlip = false;
        }
    };
    TreeNode tree[120005];
    vector<int> a;
    
    void buildTree(int node, int l, int r) {
        if (l > r) return;
        if (l == r) tree[node].l = tree[node].r = l;
        else {
            int m = (l + r) / 2;
            tree[node].l = l;
            tree[node].r = r;
            buildTree(node * 2, l, m);
            buildTree(node * 2 + 1, m + 1, r);
        }
    }
    
    void update(int node, int l, int r) {
        if (tree[node].r < l || r < tree[node].l) return;
        if (l <= tree[node].l && tree[node].r <= r) {
            tree[node].toFlip = !tree[node].toFlip;
            return;
        }
        if (tree[node].toFlip) {
            tree[node * 2].toFlip = !tree[node * 2].toFlip;
            tree[node * 2 + 1].toFlip = !tree[node * 2 + 1].toFlip;
            tree[node].toFlip = false;
        }
        update(node * 2, l, r);
        update(node * 2 + 1, l, r);
    }
    
    bool query(int node, int l) {
        if (tree[node].l == tree[node].r) {
            if (tree[node].toFlip) return !a[l];
            return a[l];
        }
        if (tree[node].toFlip) {
            tree[node * 2].toFlip = !tree[node * 2].toFlip;
            tree[node * 2 + 1].toFlip = !tree[node * 2 + 1].toFlip;
            tree[node].toFlip = false;
        }
        int m = (tree[node].l + tree[node].r) / 2;
        if (l <= m) return query(node * 2, l);
        else return query(node * 2 + 1, l);
    }
    
public:
    int minKBitFlips(vector<int>& A, int K) {
        a = A;
        int n = A.size();
        int ans = 0;
        buildTree(1, 0, n - 1);
        for (int i = 0; i < n; i++) {
            bool x = query(1, i);
            if (!x && i + K > n) return -1;
            if (!x) {
                update(1, i, i + K - 1);
                ans++;
            }
        }
        return ans;
    }
};
```

### O(n)

```cpp
class Solution {
public:
    int minKBitFlips(vector<int>& A, int K) {
        int n = A.size();
        int event[n + 1];
        memset(event, 0, sizeof(event));
        int cur = 0, ans = 0;
        for (int i = 0; i < n; i++) {
            cur += event[i];
            int x = (A[i] + cur) % 2;
            if (x == 0) {
                if (i + K - 1 >= n) return -1;
                cur++;
                ans++;
                event[i + K]--;
            }
        }
        return ans;
    }
};
```
