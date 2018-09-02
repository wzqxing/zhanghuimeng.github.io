---
title: Leetcode 295. Find Median from Data Stream（堆）
urlname: leetcode-295-find-median-from-data-stream
toc: true
date: 2018-08-05 19:20:47
updated: 2018-08-05 19:20:47
tags: [Leetcode, alg:Heap]
---

题目来源：[https://leetcode.com/problems/find-median-from-data-stream/description/](https://leetcode.com/problems/find-median-from-data-stream/description/)

标记难度：Hard

提交次数：2/2

代码效率：

* vector版本：17.75%
* 优先队列版本：17.75%

## 题意

写一个读入数据流并随时按要求输出中位数的工具类。

## 分析

我最开始的想法就是直接维护一个有序vector。这样做显然是可以的，但是每次插入的代价都是`O(N)`，看起来比较高。

交上去之后，我立刻想到了一种优化方法：维护一个二叉搜索树，这样插入和查询的代价就基本上都是`O(log(N))`了。不过我并没有写这个方法，因为我在[题解](https://leetcode.com/problems/find-median-from-data-stream/discuss/74062/Short-simple-JavaC++Python-O%28log-n%29-+-O%281%29)里看到了一种很聪明的方法——维护两个堆。大根堆中存储较小的一半元素，小根堆中存储较大的一半元素，维护两个堆的大小大致相等，这样就可以直接通过这两个堆的堆顶计算中位数了。

## 代码

### vector版本

```cpp
class MedianFinder {
private:
    vector<int> nums;

public:
    /** initialize your data structure here. */
    MedianFinder() {
        // 最简单的方法就是直接把所有数缓存下来，排序，每次返回中间的值
        // 但其实我感觉这样是不行的。
        // 也许能用int做些文章？
        // 或许应该保存一棵二叉搜索树，找median比较方便？
        // 我感觉把所有数都存下来可能是必要的。
    }

    void addNum(int num) {
        auto i = lower_bound(nums.begin(), nums.end(), num);
        nums.insert(i, num);

        /*for (int x: nums)
            cout << x << ' ';
        cout << endl;*/
    }

    double findMedian() {
        if (nums.size() % 2 != 0)
            return nums[nums.size() / 2];
        return (nums[nums.size() / 2 - 1] + nums[nums.size() / 2]) / 2.0;
    }
};

/**
 * Your MedianFinder object will be instantiated and called as such:
 * MedianFinder obj = new MedianFinder();
 * obj.addNum(num);
 * double param_2 = obj.findMedian();
 */
```

### 优先队列版本

```cpp
class MedianFinder {
    // maxHeap: 较小的一半元素
    // minHeap：较大的一半元素
    priority_queue<int> maxHeap;
    priority_queue<int, vector<int>, greater<int>> minHeap;
public:
    /** initialize your data structure here. */
    MedianFinder() {

    }

    void addNum(int num) {
        // cout << num << ' ' << maxHeap.size() << ' ' << minHeap.size() << endl;
        if (minHeap.size() == 0 || num > minHeap.top())
            minHeap.push(num);
        else
            maxHeap.push(num);
        // 维护maxHeap.size==minHeap.size || maxHeap.size==minHeap.size-1
        while (maxHeap.size() > minHeap.size()) {
            minHeap.push(maxHeap.top());
            maxHeap.pop();
        }
        while (minHeap.size() > maxHeap.size() + 1) {
            maxHeap.push(minHeap.top());
            minHeap.pop();
        }
    }

    double findMedian() {
        double ans;
        if (minHeap.size() == 0)
            return 0;
        if (maxHeap.size() == minHeap.size())
            ans = (maxHeap.top() + minHeap.top()) / 2.0;
        else
            ans = minHeap.top();
        return ans;
    }
};

/**
 * Your MedianFinder object will be instantiated and called as such:
 * MedianFinder obj = new MedianFinder();
 * obj.addNum(num);
 * double param_2 = obj.findMedian();
 */
```
