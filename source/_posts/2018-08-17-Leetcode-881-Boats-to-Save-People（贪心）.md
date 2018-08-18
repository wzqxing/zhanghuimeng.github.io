---
title: Leetcode 881. Boats to Save People（贪心）
urlname: leetcode-881-boats-to-save-people
toc: true
date: 2018-08-17 09:55:41
updated: 2018-08-18 17:25:00
tags: [Leetcode]
---

题目来源：[https://leetcode.com/problems/boats-to-save-people/description/](https://leetcode.com/problems/boats-to-save-people/description/)

标记难度：Medium

提交次数：1/1

代码效率：25.56%

## 题意

给定若干个人的重量，以及一条船的载重限制，一条船上最多载两个人，且总重量不能超过载重限制，问至少需要多少条船运载所有人。

## 分析

### 一种证明贪心算法正确的方法

一道非常简单（经典）的贪心问题，但我却被[这样的一个问题](https://leetcode.com/problems/boats-to-save-people/discuss/156748/Python-short-2-pointer-solution-and-some-thoughts)困扰住了：对于当前重量最大的人，为何我们寻找的匹配是当前重量最小的人，而不是所有可能的匹配中重量最大的人？

当然，寻找当前重量最小的匹配方法是正确的。[6 lines Java O(nlogn) code, sorting + greedy, with greedy algorithm proof. ](https://leetcode.com/problems/boats-to-save-people/discuss/156855/6-lines-Java-O%28nlogn%29-code-sorting-+-greedy-with-greedy-algorithm-proof.)中提供了一份形式化的证明，翻译如下（作为贪心算法证明方法的参考）：

令`S`表示一种最优解（显然，由于优化目标只是船的数量，最优解可能不止有一种），`O`表示我们的算法输出的解。

1. 贪心选择性质（greedy choice property）：选择只需依赖于当前情形，不需要依赖于未来的选择和子问题的其他解。

从最沉的人`hi`开始，有2种可能的情况：

(a) 如果`hi`**不能**和其他任何人进入同一艘船，则在`S`和`O`中，`hi`都独自处于一艘船中。显然，在这种情况下，我们的这一选择是最优的，贪心选择性质维持不变。

(b) 如果`hi`**可以**和其他某一个人进入一艘船，则在`O`中，根据我们的算法，`hi`和最轻的人`lo`必然位于同一条船上。

在`S`中，如果他们也在同一条船上，则我们的这一步骤是最优的，贪心选择性质保持不变；如果他们不在同一条船上，则我们可以把`hi`和`lo`在船上的同伴`m`交换一下。显然，`m <= hi`，因此交换是可以进行的。因为交换没有导致船变多，我们得到了一个新的最优解`T`，且在这种解中，`hi`和`lo`在同一条船上。这表明，我们的第一个步骤——把`hi`和`lo`放进同一条船中——是一个最优步骤，贪心选择性质也维持不变。

2. 最优子结构性质（optimal substructure property）：对原问题的最优解必然包含对子问题的最优解

令`P`表示规模为`n`的原问题，其中`n = people.length`。从上述推导中，可以看出，在第一步之后，我们得到了一个子问题`P'`，规模为`n'`（如果`hi`独自占据一艘船，则`n' = n - 1`，否则`n' = n - 2`）。我们可以相似地对`P'`中的`hi'`和`lo'`进行和刚才一样的操作。

由于我们已经证明了，`T`也是一种最优解，且`P'`的解（不妨称之为`O'`）包含在`T`内，也是一种最优解，所以这一问题具有最优子结构性质。

综上，我们的算法符合贪心选择性质和最优子结构性质，证毕。

---

好吧，我之前并不知道[贪心算法的这两种性质](https://en.wikipedia.org/wiki/Greedy_algorithm)。按照算法导论上的说法，这两种性质实际上是用来分析问题的——只有证明问题具有这两种性质时，才能确定可以应用贪心算法。而在此处的证明里，作者把实际的解法也混在证明过程中了。而“证明问题符合贪心的两种性质”和“证明贪心算法的正确性”本质上是两个问题。事实上，此处作者的证明策略应该叫做“exchange arguments”。[Proof methods and greedy algorithms](http://www.idi.ntnu.no/~mlh/algkon/greedy.pdf)这篇文章中包含了对两种策略的详细论述。

### 对于上述贪心算法的另一种理解

在开始查找这些资料之前，我尝试从直觉上去理解这种算法的重要性，并且确实找到了一种理解方法。

由于有一条船上最多坐两个人的限制，在这种算法下，实际上，我们是在为那些`weight > floor(limit/2)`的人寻找配对的，能坐在同一条船里的人；结束寻找之后，那些`weight <= floor(limit/2)`的人之间则可以随意组合。

把所有的人按重量排序，记`mid = floor(limit/2)+1`。

```
| 0 | 1 | ... | mid-1 | mid | ... | n-1 |
```

显然，此时最重的人能够配对的人是最少的，次重的人能够配对的人可能会稍微多一些。记最重的人能够配对的人的位置区间为`[0, r_1]`，次重的人能够配对的人的位置区间为`[0, r_2]`……直到`[0, r_mid]`。显然，`r_1 <= r_2 <= ... <= r_mid`。

现在我们实际上是在做这样一件事：从上述整数区间中选择一些点，使得它们具有这样的性质：

* 每个人只能在自己对应的区间中选点
* 点不能重复选择

我们的目的是最大化能选到点的人的数量。

此时，将最重的人与最轻的人匹配（假如他们能够匹配）的做法就很好理解了：我们实际上是为他选择了对应的线段中（还没被选择过的）最靠前的一个点。而“所有可能的匹配中重量最大的人”相当于是选择了线段上（还没被选择过的）最靠后的一个点。

对于这两种算法，不能找到匹配的情况都满足`r_i < i`。所以这两种算法本质上是一样的。

---

我感觉我的证明能力不是很强。以后再遇到贪心问题的时候，我也会尝试去做一下问题的贪心性质和贪心算法的正确性这两种证明的。

## 代码

```cpp
class Solution {
public:
    int numRescueBoats(vector<int>& people, int limit) {
        sort(people.begin(), people.end());
        int i = 0, j = people.size() - 1, sum = 0;
        while (i < j) {
            while (i < j && people[i] + people[j] > limit) {
                j--;
                sum++;
            }
            i++;
            j--;
            sum++;
        }
        if (i == j)
            sum++;
        return sum;
    }
};
```
