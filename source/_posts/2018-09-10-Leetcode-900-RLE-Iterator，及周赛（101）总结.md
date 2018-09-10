---
title: Leetcode 900. RLE Iterator，及周赛（101）总结
urlname: leetcode-900-rle-iterator-and-weekly-contest-101
toc: true
date: 2018-09-10 16:55:42
updated: 2018-09-10 18:38:00
tags: [Leetcode, Leetcode Contest]
---

题目来源：[https://leetcode.com/problems/rle-iterator/description/](https://leetcode.com/problems/rle-iterator/description/)

标记难度：Easy

提交次数：2/4

代码效率：

* 复杂的做法：4ms
* 简单的做法：4ms

## 题意

给定一个数组的游程编码（run-length encoding），写一个它的迭代器，支持操作：

* `int next(int n)`：删除数组中的前`n`个元素，返回最后一个被删除的元素；如果数组已经被删光了，则返回`-1`。

## 分析

这次比赛出了一点incident，开始时Leetcode自己崩了，进不了比赛，看不了题。所以比赛最后延长了15分钟。而且比赛期间我的心态不是很好，所以一共只做出来一道Easy，剩下的两道Medium都没做出来。不过说实话我觉得这次的题难度稍微高了一点啊……

总之排名是1331 / 4937。（这次第一又是[uwi](https://leetcode.com/uwi/)。）

---

我自己的思路倒是很简单：把原来的游程编码拆成两个数组（`len`和`val`），然后再开一个新的数组`acum`，`acum[i] = len[0] + len[1] + ... + len[i]`；用`m`记录当前已经删除了多少个数。对于每一次`next(n)`的操作，令`m += n`，然后找到这样的`i`，使得`len[i-1] < m <= len[i]`，如果找到则返回`val[i]`，否则返回`-1`。

听起来不是很难，结果我WA了两次。第一次是因为`m`爆`int`了。第二次居然是因为之前都忘了把`curIdx`的初值置零了……

但一个事实是，其实不需要额外的`O(N)`空间。只需用变量`i`记录当前还没被删光的第一个元素的位置，以及`q`记录这个元素已经被消耗了多少个。对于每一次`next(n)`的操作，若`q + n <= A[i+1]`，说明应返回当前元素，且`q += n`；否则消耗所有当前元素（`n -= A[i+1] - q; q = 0; i += 2;`）并继续寻找。这个做法有一个好处，不会爆`int`。[^solution]

[^solution]: [Leetcode 900 Solution](https://leetcode.com/problems/rle-iterator/solution/)

## 代码

### 复杂的做法

```cpp
class RLEIterator {
private:
    vector<long long int> len;
    vector<long long int> val;
    vector<long long int> acum;
    int N;
    long long int m;
    int curIdx;

public:
    RLEIterator(vector<int> A) {
        if (A.size() > 0) {
            for (int i = 0; i < A.size(); i += 2) {
                if (A[i] == 0) continue;
                len.push_back(A[i]);
                val.push_back(A[i+1]);
                if (acum.size() > 0) acum.push_back(acum.back() + A[i]);
                else acum.push_back(A[i]);
            }
            N = len.size();
        }
        else {
            N = 0;
        }
        m = 0;
        curIdx = 0;
    }

    int next(int n) {
        m += n;
        if (curIdx >= N) return -1;
        while (curIdx < N && acum[curIdx] < m) curIdx++;
        if (curIdx >= N) return -1;
        return val[curIdx];
    }
};
```

### 简单的做法

```cpp
class RLEIterator {
private:
    int q;
    int i;
    vector<int> A;

public:
    RLEIterator(vector<int> A) {
        i = q = 0;
        this->A = A;
    }

    int next(int n) {
        if (i >= A.size()) return -1;
        while (i < A.size() && A[i] - q < n) {
            n -= A[i] - q;
            q = 0;
            i += 2;
        }
        if (i >= A.size()) return -1;
        q += n;
        return A[i + 1];
    }
};
```
