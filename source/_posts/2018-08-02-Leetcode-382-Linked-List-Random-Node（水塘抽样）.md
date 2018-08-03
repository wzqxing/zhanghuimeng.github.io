---
title: Leetcode 382. Linked List Random Node（水塘抽样）
urlname: leetcode-382-linked-list-random-node
toc: true
date: 2018-08-02 01:13:08
updated: 2018-08-03 21:14:00
mathjax: true
tags: [Leetcode]
---

题目来源：[https://leetcode.com/problems/linked-list-random-node/description/](https://leetcode.com/problems/linked-list-random-node/description/)

标记难度：Medium

提交次数：3/3

代码效率：

* 平凡的解法：71.27%
* 平凡但更好的写法：99.11%
* 水塘抽样法：70.97%

## 题意

从一个链表中随机取元素。

## 分析

### 平凡的思路

一种非常直接的想法是把链表当成数组，先遍历一遍链表，得到链表的总长度，然后每次按下标取随机数，在链表中顺序查找对应的数。此时预处理的时间复杂度为`O(N)`，每次获取随机数的平摊时间复杂度为`O(N)`。当然，如果直接新开一个`O(N)`的数组（vector），把链表缓存下来，然后直接按下标寻址，则每次获取随机数的时间复杂度会下降到`O(1)`。

虽然我们平时常用的生成随机数的搭配是`srand(time(0))`和`rand()`，但它们其实不是很符合伪随机数的要求，一部分原因是`rand()`生成的随机数的大小最大为`RAND_MAX`。（参见[RAND_MAX](https://en.cppreference.com/w/cpp/numeric/random/RAND_MAX)）既然我们写的是C++，那么更标准的方法是采用`std::uniform_int_distribution`。[使用方法](https://zh.cppreference.com/w/cpp/numeric/random/uniform_int_distribution)如下：

```
#include <random>
#include <iostream>

int main()
{
    std::random_device rd;  // 将用于为随机数引擎获得种子
    std::mt19937 gen(rd()); // 以播种标准 mersenne_twister_engine
    std::uniform_int_distribution<> dis(1, 6);

    for (int n=0; n<10; ++n)
        // 用 dis 变换 gen 所生成的随机 unsigned int 到 [1, 6] 中的 int
        std::cout << dis(gen) << ' ';
    std::cout << '\n';
}
```

我写了相应的代码，但实在是太麻烦了，平时还是用`rand()`方便啊！

### 水塘抽样法

至于[水塘抽样](https://en.wikipedia.org/wiki/Reservoir_sampling)法——它的优点是不需要提前扫描一遍以获得整个链表的大小，缺点是每次都需要遍历整个链表并取多次随机数来进行抽样，虽然时间复杂度仍然是`O(N)`，但隐含的常数变大了。我认为它适合的场景是一次取多个随机数，而不是这种情况。

我总是记不住水塘抽样法的原理，不如在这里把维基上的最简单的例子复述一遍：假定你想要从网易云音乐的全体曲库（显然原文不是网易云音乐，我就随便举个我熟悉例子）里随机取出10首乐曲，保存在“我喜欢的音乐”歌单中，但是你并不知道网易云音乐的曲库有多大。此时，你可以采用这种方法：

* 顺序浏览所有乐曲。
* 将前10首乐曲保存到歌单中。
* 浏览到第$i$（$i > 10$）首乐曲的时候，以$\frac{10}{i}$的概率保存这首新的乐曲（同时删除一首已保存的旧的曲子；其中每首旧乐曲被删除的概率都是$\frac{1}{10}$），也即以$1 - \frac{10}{i}$的概率删除这首乐曲。
* 不断重复上一过程，直到遍历完所有乐曲。

如果网易云音乐里一共只有10首乐曲，那么显然每首乐曲被保存下来的概率都是1。如果有11首乐曲，则在浏览到第11首乐曲的时候，我们保存这首乐曲的概率为$\frac{10}{11}$；对于之前的旧乐曲，其中某一首被保存下来的概率为

$$P(\text{新乐曲没有被选择保存}) + P(\text{新乐曲被选择保存}) \cdot P(\text{这一首没有被选择替换掉}) \\\\
= \frac{1}{11} + \frac{10}{11} \cdot \frac{9}{10} = \frac{10}{11}$$

也就是说，到目前为止，对于这11首乐曲，每一首仍然处于歌单中的概率都是$\frac{10}{11}$，这是符合我们的要求的。

在浏览到第12首乐曲的时候，我们保存这首乐曲的概率为$\frac{10}{12}$；而对于之前的那11首乐曲，它们仍然处于歌单中的概率为

$$P(\text{在只看到过11首曲子的的时候，它们还在歌单里}) \cdot (P(\text{第12首乐曲被直接丢弃了}) \\\\ + P(\text{第12首乐曲被选择保留}) \cdot P(\text{这一首没有被选择替换掉})) \\\\
= \frac{10}{11} (\frac{2}{12} + \frac{10}{12} \cdot \frac{9}{10})$$

也就是说，现在每一首乐曲仍然处于歌单中的概率都是$\frac{10}{12}$，这是符合常理的。

严谨的证明需要用到数学归纳法，不过形式和上述思考过程非常类似，所以我就懒得再抄一遍了。

我觉得这是一种非常精妙的incremental的算法。

## 代码

### 平凡的解法

```
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
    int length;
    ListNode* head;
public:
    /** @param head The linked list's head.
        Note that the head is guaranteed to be not null, so it contains at least one node. */
    Solution(ListNode* head) {
        length = 0;
        ListNode* p = head;
        this->head = head;
        while (p != NULL) {
            length++;
            p = p->next;
        }
        srand(time(0));
    }

    /** Returns a random node's value. */
    int getRandom() {
        int x = rand() % length;
        int i = 0;
        ListNode* p = head;
        while (i < x) {
            p = p->next;
            i++;
        }
        return p->val;
    }
};

/**
 * Your Solution object will be instantiated and called as such:
 * Solution obj = new Solution(head);
 * int param_1 = obj.getRandom();
 */
```

### 平凡但更好的写法

```
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
    int length;
    ListNode* head;
    random_device rd;  // 将用于为随机数引擎获得种子
    mt19937* gen; // 以播种标准 mersenne_twister_engine
    uniform_int_distribution<>* dis;
public:
    /** @param head The linked list's head.
        Note that the head is guaranteed to be not null, so it contains at least one node. */
    Solution(ListNode* head) {
        length = 0;
        ListNode* p = head;
        this->head = head;
        while (p != NULL) {
            length++;
            p = p->next;
        }
        gen = new mt19937(rd());
        dis = new uniform_int_distribution<>(0, length-1);
    }

    /** Returns a random node's value. */
    int getRandom() {
        int x = (*dis)(*gen);
        int i = 0;
        ListNode* p = head;
        while (i < x) {
            p = p->next;
            i++;
        }
        return p->val;
    }
};

/**
 * Your Solution object will be instantiated and called as such:
 * Solution obj = new Solution(head);
 * int param_1 = obj.getRandom();
 */
```

### 水塘抽样法

```
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
    ListNode* head;
public:
    /** @param head The linked list's head.
        Note that the head is guaranteed to be not null, so it contains at least one node. */
    Solution(ListNode* head) {
        this->head = head;
    }

    /** Returns a random node's value. */
    int getRandom() {
        // 进行水塘抽样
        int ans = head->val;
        int i = 2;
        ListNode* p = head->next;
        while (p != NULL) {
            int x = rand() % i;
            if (x == 0)
                ans = p->val;
            i++;
            p = p->next;
        }
        return ans;
    }
};

/**
 * Your Solution object will be instantiated and called as such:
 * Solution obj = new Solution(head);
 * int param_1 = obj.getRandom();
 */
```
