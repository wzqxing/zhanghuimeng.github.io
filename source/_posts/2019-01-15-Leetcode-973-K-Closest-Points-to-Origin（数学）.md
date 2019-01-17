---
title: Leetcode 973. K Closest Points to Origin（数学）
urlname: leetcode-973-k-closest-points-to-origin
toc: true
date: 2019-01-15 17:13:52
updated: 2019-01-17 15:51:00
tags: [Leetcode, Leetcode Contest, alg:Math, alg:Sort]
---

题目来源：[https://leetcode.com/problems/k-closest-points-to-origin/description/](https://leetcode.com/problems/k-closest-points-to-origin/description/)

标记难度：Easy

提交次数：2/5

代码效率：

* 普通方法：80.0%（208ms）
* 堆：60.00%（212ms）
* 快排：40.00%（224ms）

## 题意

给定平面上的若干个（<=10000）点，求距离原点最近的`K`个点。保证结果对应的点集唯一，以任意顺序返回都可以。

## 分析

其实我睡过了这次比赛。随后又撞上期末考试和毕设，感觉流年不利。

---

显然可以把距离算出来之后排序，然后取前`K`个点，时间复杂度为`O(N*log(N))`。

如果想做得更好一点的话，可以用一个大小为`K`的堆来维护距离前`K`小的点，时间复杂度为`O(N*log(K))`。

题解里给出了一种神奇的方法，思路是用类似于快排的方法求前`K`小的点，感觉很有趣。不过为了严格起见，大概还是要每次随机选主元才行……[^sln]

[^sln]: [Leetcode Official Solution for 973. K Closest Points to Origin](https://leetcode.com/problems/k-closest-points-to-origin/solution/)

### 快排！快排！

为了写好题解里快排的方法，浪费了一两个小时。我果然又把快排的写法忘光了！姑且先总结一下。

快排的第一种写法不妨称之为交替填坑法。这种做法的核心规律是，先把序列最靠左的元素拿出来作为主元（并空出来一个坑），然后从右向左找到第一个可以填这个坑的元素（也就是第一个比主元小的元素），把它拿出来填到这个坑里。此时右边就空出来一个新坑。然后从左边刚被填完的坑向右扫描，找到第一个能够填右边的坑的元素（也就是第一个比主元大的元素），把它拿出来填到右边的坑里。此时左边又空出来一个新坑……以此类推。最后中间剩下一个坑，满足左边的元素都比主元小，右边的元素都比主元大，再把主元填到这个坑里。

以序列`[3, 6, 5, 4, 2, 1, 0]`为例：

```
| 3 | 6 | 5 | 4 | 2 | 1 | 0 |
  ^
pivot

pivot = 3, remove pivot
| x | 6 | 5 | 4 | 2 | 1 | 0 |
  i                       j

pivot = 3, move A[j] to A[i], i++
| 0 | 6 | 5 | 4 | 2 | 1 | x |
      i                   j

pivot = 3, move A[i] to A[j], j--
| 0 | x | 5 | 4 | 2 | 1 | 6 |
      i               j

pivot = 3, move A[j] to A[i], i++
| 0 | 1 | 5 | 4 | 2 | x | 6 |
          i           j

pivot = 3, move A[i] to A[j], j--
| 0 | 1 | x | 4 | 2 | 5 | 6 |
          i       j

pivot = 3, move A[j] to A[i], i++
| 0 | 1 | 2 | 4 | x | 5 | 6 |
              i   j

pivot = 3, move A[i] to A[j], j--
| 0 | 1 | 2 | x | 4 | 5 | 6 |
             i j

found i==j, put pivot into i (or j)
| 0 | 1 | 2 | 3 | 4 | 5 | 6 |
             i j
```

显然可以看出，每个周期包含两次指针移动和交换，其中`i`和`j`的含义是分阶段而不同的。

此处有坑，明天再填。

2019.1.17 UPDATE：坑应该是填不动了。简单来说，还有一种常用的写法是不移出主元，而是让它在里面跟着交换。这两种写法都解决不了主元重复的问题，此时最好改成CLRS式的写法。

## 代码

### 普通方法

```cpp
class Solution {
public:
    vector<vector<int>> kClosest(vector<vector<int>>& points, int K) {
        vector<pair<int, int>> toSort;
        for (int i = 0; i < points.size(); i++) {
            toSort.emplace_back(points[i][0] * points[i][0] + points[i][1] * points[i][1], i);
        }
        sort(toSort.begin(), toSort.end());
        vector<vector<int>> ans;
        for (int i = 0; i < K; i++)
            ans.push_back(points[toSort[i].second]);
        return ans;
    }
};
```

### 堆

```cpp
class Solution {
public:
    vector<vector<int>> kClosest(vector<vector<int>>& points, int K) {
        priority_queue<pair<int, int>> q;
        for (int i = 0; i < points.size(); i++) {
            q.emplace(points[i][0] * points[i][0] + points[i][1] * points[i][1], i);
            if (q.size() > K) q.pop();
        }
        vector<vector<int>> ans;
        while (!q.empty()) {
            ans.push_back(points[q.top().second]);
            q.pop();
        }
        return ans;
    }
};
```

### 快排

```cpp
class Solution {
private:
    int dist(vector<int>& p) {
        return p[0] * p[0] + p[1] * p[1];
    }
    
    void partition(vector<vector<int>>& points, int l, int r, int& K) {
        if (l >= r) return;
        if (l >= K) return;
        if (r < K) return;
        // int x = rand() % (r - l + 1) + l;
        int x = l;
        int i = l, j = r;
        int pivot = dist(points[x]);
        while (i < j) {
            // why <, not <=?
            while (dist(points[i]) < pivot && i < j) i++;
            while (pivot < dist(points[j]) && i < j) j--;
            if (i < j) swap(points[i], points[j]);
        }
        partition(points, l, i - 1, K);
        partition(points, i + 1, r, K);
    }
    
public:
    vector<vector<int>> kClosest(vector<vector<int>>& points, int K) {
        partition(points, 0, points.size() - 1, K);
        return vector(points.begin(), points.begin() + K);
    }
};
```