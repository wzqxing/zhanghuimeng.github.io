---
title: Leetcode 710. Random Pick with Blacklist（二分；STL）
urlname: leetcode-710-random-pick-with-blacklist
toc: true
date: 2018-08-30 12:50:17
updated: 2018-08-30 13:48:17
tags: [Leetcode, alg:Hash Table, alg:Binary Search, alg:Random]
---

题目来源：[https://leetcode.com/problems/random-pick-with-blacklist/description/](https://leetcode.com/problems/random-pick-with-blacklist/description/)

标记难度：Hard

提交次数：3/6

代码效率：

* 二分：10.06%
* 重映射：45.67%

## 题意

给定区间`[0, N)`和一个该区间内的“黑名单”，要求以均匀的概率返回该区间内非黑名单的数字，且调用`rand()`的次数尽量少。

## 分析

### 二分

我感觉这是一道很妙的题。看到题之后，我的第一反应是：假定黑名单的长度为`M`，那么我们只需要随机`[0, N - M)`范围内的数字，然后把它们映射到`[0, N)`区间内就可以了。问题是怎么映射呢？

于是我想出了这样一种做法：随机得到一个数字之后，计算黑名单中有多少个数小于等于这个数，然后把这个数量加到这个数上，返回结果。然后我就把代码交上去了，于是，很快我就发现这个做法错得有点离谱，因为它完全没有保证返回的数不在黑名单里。反例如`N = 4, blacklist = [0, 1]`，随机得到0时，上述做法会返回1。

于是我尝试从另一个角度来思考这个问题。我们显然可以显式地构建一个映射：顺序枚举`[0, N - M)`范围内的数，维护一个指针，指向下一个能够被映射的数，遇到黑名单中的数则跳过。这个做法是正确的，但考虑到`1 <= N <= 1000000000`的数据量，显然不可能把整个映射表都建立出来，然后去查表。

于是这个思路被否决了，我又回头去考虑怎么在线计算出准确的映射这个问题。我想了想：考虑“黑名单中有多少个数小于等于当前的数”这个思路其实是正确的，问题在于，被考虑的不应该是`[0, N - M)`中的数，而应该是`[0, N)`中的数。考察这个例子，`N = 9, balcklist = [2, 3, 5]`，令`blacklist_before(i)`表示黑名单中`<=i`的数的个数：

| `[0, N)`中的数`i` | `blacklist_before(i)` | `i - blacklist_before(i)` | 映射到`[0, N - M)`中 |
| ---- | ---- | ---- | ---- |
| 0     | 0 | 0 | 0 |
| 1     | 0 | 1 | 1 |
| **2** | 1 | 1 | - |
| **3** | 2 | 1 | - |
| 4     | 2 | 2 | 2 |
| **5** | 3 | 2 | - |
| 6     | 3 | 3 | 3 |
| 7     | 3 | 4 | 4 |
| 8     | 3 | 5 | 5 |

可以观察得到一些非常有趣的性质：

* `i - blacklist_before(i)`是非单调递增的，因为它代表的是`[0, i]`区间内不属于黑名单内的数的个数
* 当`i - blacklist_before(i)`没有递增时，表示`i`是一个黑名单内的数字
* `i`在`[0, N - M)`区间中应该映射到的数与`i - blacklist_before(i)`两列很接近（从意义上来说也是）

所以可以从这个逆向映射的角度考虑，用二分的方法解决问题：假定在`[0, N - M)`中随机得到了`y`，我们需要找到满足`x - blacklist_before(x) == y`的`x`的`lower_bound`。然后就可以写了。

---

在讨论区里我看到了类似的做法，但是思考的角度不太一样。我们可以把从区间`[0, N - M)`中生成随机数`r`的情形这样分类：[^binary]

* `r`在`[0, B[0])`区间内，可以直接返回`r`
* `r`在`[B[0], B[1] - 1)`区间内，应返回`r + 1`
* ...
* `r`在`[B[i]-i, B[i+1]-(i+1))`区间内，应返回`r + i + 1`。注意到`r + i + 1`位于`[B[i] + 1, B[i+1])`区间内，因此这样做是安全的。

因此可以在`B[i] - (i+1)`数组中进行二分查找。

这种做法是在经过处理的`blacklist`数组上进行二分查找，而我是在`[0, N)`区间上直接查找，所以比我的做法更压缩一些。

[^binary]: [Simple Java solution with Binary Search](https://leetcode.com/problems/random-pick-with-blacklist/discuss/146545/Simple-Java-solution-with-Binary-Search)

### HashMap重映射法

我之前考虑过建立整个映射表，但因为数据量而放弃了。然而，事实上，完全不需要把整个映射表都建立出来；或者不如说，完全不需要按顺序建立映射，只要保证所有数字都可以被随机取到即可。所以可以采取这样的一种做法：将黑名单中的元素分成两类，一类在`[0, N - M)`区间内，一类在`[N - M, N)`区间内。然后在`[N - M, N)`区间内为`[0, N - M)`区间内的黑名单元素顺序寻找映射（我看到了从前向后[^fronttoend]和从后向前[^endtofront]两种方法，不过本质上是相同的），同时注意跳过那些同样在黑名单内的元素。

[^fronttoend]: [Java O(B) constructor and O(1) pick, HashMap](https://leetcode.com/problems/random-pick-with-blacklist/discuss/144474/Java-O%28B%29-constructor-and-O%281%29-pick-HashMap)

[^endtofront]: [Java O(B) / O(1), HashMap](https://leetcode.com/problems/random-pick-with-blacklist/discuss/144624/Java-O%28B%29-O%281%29-HashMap)

上述从后向前找映射的一个图示例子：`N=10, blacklist=[3, 5, 8, 9]`，将3和5映射为7和6。

![从后向前映射](remapping.png)

## 代码

### 二分

```cpp
class Solution {
private:
    int N;
    vector<int> blacklist;
    unordered_set<int> blackSet;
    random_device rd;
    mt19937 gen;
    uniform_int_distribution<> dist;

    int findMap(int y) {
        int l = 0, r = N - 1;
        // need to find lower_bound
        // 二分真是一件Tricky的事情
        while (l < r) {
            int mid = (l + r) / 2;
            int smaller = upper_bound(blacklist.begin(), blacklist.end(), mid) - blacklist.begin();
            if (mid - smaller < y)
                l = mid + 1;
            else
                r = mid;
        }

        return l;
    }

public:
    Solution(int N, vector<int> blacklist): gen(rd()), dist(0, N - blacklist.size() - 1) {
        this->N = N;
        this->blacklist = blacklist;
        sort(this->blacklist.begin(), this->blacklist.end());
        for (int b: blacklist)
            blackSet.insert(b);
    }

    int pick() {
        int r = dist(gen);
        int m = findMap(r);
        // cout << r << ' ' << m << endl;
        // 事实证明，保证lower_bound之后，找到的数必然不是黑名单中的数
        /* while (blackSet.find(m) != blackSet.end())
            m++; */
        return m;
    }
};

/**
 * Your Solution object will be instantiated and called as such:
 * Solution obj = new Solution(N, blacklist);
 * int param_1 = obj.pick();
 */
// 这个问题很有趣。
// 既然从N个数的区间中屏蔽了B.length个点，那么就是从N - B.length的区间中随机选点
// 然后换算一下。
// 但是换算方法看起来是个很大的问题……
```

### 重映射

```cpp
class Solution {
private:
    int N;
    vector<int> blacklist;
    unordered_set<int> blackSet;
    unordered_map<int, int> blackMap;
    random_device rd;
    mt19937 gen;
    uniform_int_distribution<> dist;

public:
    Solution(int N, vector<int> blacklist): gen(rd()), dist(0, N - blacklist.size() - 1) {
        this->N = N;
        this->blacklist = blacklist;
        sort(this->blacklist.begin(), this->blacklist.end());
        for (int b: blacklist)
            blackSet.insert(b);

        int M = blacklist.size();
        int mapTo = N - M;
        for (int b: this->blacklist) {
            if (b >= N - M)
                break;
            while (blackSet.find(mapTo) != blackSet.end())
                mapTo++;
            blackMap[b] = mapTo++;
        }
    }

    int pick() {
        int r = dist(gen);
        if (blackSet.find(r) != blackSet.end())
            return blackMap[r];
        return r;
    }
};

/**
 * Your Solution object will be instantiated and called as such:
 * Solution obj = new Solution(N, blacklist);
 * int param_1 = obj.pick();
 */
```
