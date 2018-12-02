---
title: Leetcode 950. Reveal Cards In Increasing Order（数组）
urlname: leetcode-950-reveal-cards-in-increasing-order
toc: true
date: 2018-12-02 12:48:25
updated: 2018-12-02 13:13:00
tags: [Leetcode, Leetcode Contest, alg:Math, alg:Array]
---

题目来源：[https://leetcode.com/problems/reveal-cards-in-increasing-order/description/](https://leetcode.com/problems/reveal-cards-in-increasing-order/description/)

标记难度：Medium

提交次数：2/2

代码效率：8ms

## 题意

有一叠卡片，每张卡片上写着各不相同的数字。对卡片叠重复进行如下操作：

* 拿出叠顶卡片，记录上面的值
* （如果卡片叠非空）把下一张叠顶的卡片放到叠底

问把卡片以什么顺序叠放，才能使得记录的值是顺序递增的？

## 分析

其实这道题只要仔细想并不难。用映射的方法来解释比较好。不妨先假设我们拿的是一摞从上到下是1, 2, 3, 4, 5, 6, 7的卡片。进行操作（操作过程略）后，得到的值为1, 3, 5, 7, 4, 2, 6。因此，我们可以得到一张卡片在原来的叠中的index -> 卡片在记录的值中的index的映射表$\sigma$：

| 1   | 2   | 3   | 4   | 5   | 6   | 7   |
| --- | --- | --- | --- | --- | --- | --- |
| 1   | 3   | 5   | 7   | 4   | 2   | 6   |

不妨假设我们的卡片开始时已经排好序了，也就是说，我们期望所有卡片在记录的值中的index顺序是1, 2, ..., 7。此时只需应用上述映射表的逆映射$\sigma^{-1}$，即可得到满足这种性质的卡片在原来的叠中应有的index顺序：

| 1   | 2   | 3   | 4   | 5   | 6   | 7   |
| --- | --- | --- | --- | --- | --- | --- |
| 1   | 6   | 2   | 5   | 3   | 7   | 4   |

以及，我本来以为是要返回卡片index的一个order的——也就是说，我们还需要记录卡片排序前后的index映射表——吓死我了。原来直接返回卡片的值就好了。不过，这也不过是多加一个映射而已……

## 代码

```cpp
class Solution {
public:
    vector<int> deckRevealedIncreasing(vector<int>& deck) {
        // get the mapping
        queue<int> q;
        int n = deck.size();
        int ord2rev[n];
        for (int i = 0; i < n; i++)
            q.push(i);
        for (int i = 0; i < n; i++) {
            int x = q.front();
            q.pop();
            ord2rev[x] = i;
            if (!q.empty()) {
                x = q.front();
                q.pop();
                q.push(x);
            }
        }
        // sort
        sort(deck.begin(), deck.end());
        // apply reversed mapping
        vector<int> ans;
        for (int i = 0; i < n; i++)
            ans.push_back(deck[ord2rev[i]]);
        return ans;
    }
};
```