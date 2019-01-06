---
title: Leetcode 969. Pancake Sorting（排序）
urlname: leetcode-969-pancake-sorting
toc: true
date: 2019-01-06 19:00:07
updated: 2019-01-06 19:35:00
tags: [Leetcode, Leetcode Contest, alg:Sort, alg:Array]
---

题目来源：[https://leetcode.com/problems/pancake-sorting/description/](https://leetcode.com/problems/pancake-sorting/description/)

标记难度：Medium

提交次数：1/1

代码效率：8ms

## 题意

给定一个数组`A`（其中的元素是`[1, 2, ..., A.length]`的排列），定义pancake flip操作为翻转`A`中前`k`个数，其中`k <= A.length`。要求通过最多`10 * A.length`词操作将`A`排序。求操作序列。

## 分析

比赛的时候我的想法很简单：因为每次操作影响的只有前`k`个数，所以不妨考虑先排好`A`中靠后的数，这样之后的操作就不会影响它们。于是就可以得到一个很显然的思路：需要把`i`放到`i`位置时，就找到`i`，先通过一次操作把它flip到数组开头，再通过一次操作把它flip到`i`位置。这样，操作次数最多为`2 * A.length`，符合要求。

以序列`[3, 1, 5, 2, 4]`为例：

* 将5放到第5位：
  * flip 3：`[5, 1, 3, 2, 4]`
  * flip 5：`[4, 2, 3, 1, 5]`
* 将4放到第4位：
  * flip 1（不需要）
  * flip 4：`[1, 3, 2, 4, 5]`
* 将3放到第3位：
  * flip 2：`[3, 1, 2, 4, 5]`
  * flip 3：`[2, 1, 3, 4, 5]`
* 将2放到第2位：
  * flip 1（不需要）
  * flip 2：`[1, 2, 3, 4, 5]`
* 将1放到第1位（已经在第1位了）

---

这是一种非常straightforward的方法，题解也是这么做的。

而且显然上述方法已经说明，flip操作能够将任意排列变换成其他任意排列。

我的问题是：

1. 直接模拟的复杂度为`O(N^2)`（因为每次翻转操作的复杂度是`O(N)`），能否降低复杂度？
2. 这样得到的是否为最优解？

第二个问题的答案是，显然不是，这种做法只是给出了一个上界（至多`2*N - 3`次翻转必然可以完成排序，至于为什么少了3次，请考虑只剩1和2时的情形）。而且找到最优解是一个NP-难问题。那么我就不再耗费我可怜的脑细胞在这个问题上了。顺便一提，比尔·盖茨证明了这个问题的上界是`5(N+5) / 3`；目前最优的结果是`18N / 11`。[^wiki]

经过一些思考和查找资料，我认为我目前无法回答第一个问题。

[^wiki]: 原来Pancake Sorting这个名字并不是乱起的！[wikipedia - Pancake sorting](https://en.wikipedia.org/wiki/Pancake_sorting)

## 代码

```
class Solution {
private:
    vector<int> ans;
    void flip(int k, vector<int>& A) {
        for (int i = 0; i < k - i - 1; i++)
            swap(A[i], A[k - i - 1]);
        ans.push_back(k);
    }
    
public:
    vector<int> pancakeSort(vector<int>& A) {
        int n = A.size();
        for (int i = n; i >= 1; i--) {
            // put i on place i
            int index = 0;
            while (A[index] != i && index < n) index++;
            if (index == i - 1) continue;
            flip(index + 1, A);
            flip(i, A);
        }
        return ans;
    }
};
```