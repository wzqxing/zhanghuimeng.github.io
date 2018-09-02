---
title: Leetcode 493. Reverse Pairs（数组子问题）
urlname: leetcode-493-reverse-pairs
toc: true
date: 2018-08-11 16:34:53
updated: 2018-08-11 17:21:00
tags: [Leetcode, alg:Divide and Conquer, alg:Binary Search Tree, alg:Binary Indexed Tree]
---

题目来源：[https://leetcode.com/problems/reverse-pairs/description/](https://leetcode.com/problems/reverse-pairs/description/)

标记难度：Hard

提交次数：1/4

代码效率：93.15%

## 题意

计算数组中**重要逆序对**的数量。称`(i, j)`为重要逆序对，当且仅当`i < j`且`nums[i] > 2 * nums[j]`。

## 分析

其实思路并不是很难。归并排序求逆序对的思路已经很平常了，这道题中直接把计数的条件改一改，只统计另一半数组中至少为当前的数的两倍的数的数量（而不是单纯“大于”）就可以了。

但是我提交了4次。第一次是数组边界写错了，后面几次都是被卡在这个`*2`上了——我还是第一次见到卡`2 * (int) 2147483647`这个位置的题，也可能是我孤陋寡闻吧。当然，我承认这么做有点道理，这和单纯比大小是不一样的。反正我最后手动在比较的时候把`int`cast成`long long`了。

### 一种合并的方法

我之前从来没有听说过[inplace_merge](https://zh.cppreference.com/w/cpp/algorithm/inplace_merge)这个函数。又一次在[StephanPochmann的题解](https://leetcode.com/problems/reverse-pairs/discuss/97287/C++-with-iterators)中看到了一些闻所未闻的东西之后，我去查了一下，发现：原来STL算法库里为我们提供了一个原址合并算法啊！

```cpp
template<class Iter>
void merge_sort(Iter first, Iter last)
{
    if (last - first > 1) {
        Iter middle = first + (last - first) / 2;
        merge_sort(first, middle);
        merge_sort(middle, last);
        std::inplace_merge(first, middle, last);
    }
}
```

利用这个算法来实现归并排序无疑是非常简单明了的。

另一个问题是，`inplace_merge`是如何实现的？我之前从未想过归并算法可以是原址的。事实是，确实可以。请参见[Practical In-Place Merging](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.88.1155&rep=rep1&type=pdf)这篇论文和相应的实现（[InplaceMerge.hh](http://keithschwarz.com/interesting/code/?dir=inplace-merge)）。不过这个算法的确有点复杂。

### 类似于逆序对的题目

讨论区里有一篇非常好的[文章](https://leetcode.com/problems/reverse-pairs/discuss/97268/General-principles-behind-problems-similar-to-%22Reverse-Pairs%22)，对相似类型的题目进行了一个整体的讨论。简单来说，此类题目的共同点是：将数组分解，然后解决子问题。

通常我们有两种分解数组（子问题）的方法：

1. 顺序分解：`T(i, j) = T(i, j - 1) + C`
2. 二分分解：`T(i, j) = T(i, m) + T(m + 1, j) + C`，其中`m = (i + j) / 2`。

（是的，我之前没有想过，其实顺序分解也是可以做到`O(n * log(n))`的。）

顺序分解对于本题来说，也就意味着，顺序扫描数组，寻找以当前的数为逆序对的第二个元素的逆序对的数量。显然直接扫描是可以的，但是复杂度会上升到`O(n^2)`。为了提高搜索的效率，我们注意到，子数组内部的顺序不重要，因此我们可以把它转换成一棵二叉搜索树（或者任何类似的东西）。这样效率就可以提高到`O(n * log(n))`了。

二分分解就是我的做法：分别寻找元素都来自左半数组和右半数组的逆序对，然后寻找元素各自来自一侧的逆序对，最后加在一起。

## 代码

```cpp
class Solution {
private:
    int* tmp;
    int split(int l, int r, vector<int>& nums) {
        if (l == r)
            return 0;
        int m = (l + r) >> 1;
        int sum = 0;
        sum += split(l, m, nums);
        sum += split(m+1, r, nums);

        // 统计important逆序对的数目
        int i = l, j = m+1;
        while (i <= m && j <= r) {
            // 第一次遇到卡这里的题目……
            while (i <= m && (long long)nums[i] <= (long long)(nums[j]) * 2)
                i++;
            if (i > m)
                break;
            sum += m - i + 1;
            j++;
        }
        // 合并
        i = l;
        j = m+1;
        int k = 0;
        while (i <= m && j <= r) {
            if (nums[i] <= nums[j])
                tmp[k++] = nums[i++];
            else
                tmp[k++] = nums[j++];
        }
        while (i <= m)
            tmp[k++] = nums[i++];
        while (j <= r)
            tmp[k++] = nums[j++];
        for (i = l; i <= r; i++)
            nums[i] = tmp[i - l];

        return sum;
    }

public:
    int reversePairs(vector<int>& nums) {
        // 只是普通的求逆序对算法的一个变形
        if (nums.size() == 0)
            return 0;

        tmp = new int[nums.size() + 1];
        int sum = split(0, nums.size() - 1, nums);
        delete[] tmp;
        return sum;
    }
};
```
