---
title: Leetcode 283. Move Zeroes（数组指针）
urlname: leetcode-283-move-zeroes
toc: true
date: 2018-08-05 11:19:59
updated: 2018-08-05 11:43:00
tags: [Leetcode, alg:Array, alg:Two Pointers]
---

题目来源：[https://leetcode.com/problems/move-zeroes/description/](https://leetcode.com/problems/move-zeroes/description/)

标记难度：Easy

提交次数：1/1

代码效率：32.62%

## 题意

把数组中的0都移动到数组末尾，其他数的相对顺序保持不变。

## 分析

总的来说是道水题：需要移动的不是0，而是其他的数，把它们直接前移就行。这算是一种什么思路呢？巧妙地利用数组中的指针吗？也许这只是表象而已。[Discuss](https://leetcode.com/problems/move-zeroes/discuss/72005/My-simple-C++-solution)里有人说这种思路类似于快速排序，怎么说呢，还真有点像，数组同样被划分成了几个区域，中间区域也是滚动向前的，只是这里被移动到后面的元素都是0。

![数组中元素的性质](array.jpg)

## 代码

```
class Solution {
public:
    void moveZeroes(vector<int>& nums) {
        // 我感觉只要把所有的数都往前挪，最后在后面补0就可以了。
        int validCnt = 0;
        for (int i = 0; i < nums.size(); i++) {
            if (nums[i] != 0)
                nums[validCnt++] = nums[i];
        }
        for (int i = validCnt; i < nums.size(); i++)
            nums[i] = 0;
    }
};
```
