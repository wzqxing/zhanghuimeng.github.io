---
title: Leetcode 187. Repeated DNA Sequences（Hash）
urlname: leetcode-187-repeated-dna-qequences
toc: true
date: 2018-10-04 00:55:57
updated: 2018-10-04 01:33:57
tags: [Leetcode, alg:Hash Table, alg:Bit Manipulation]
---

题目来源：[https://leetcode.com/problems/repeated-dna-sequences/description/](https://leetcode.com/problems/repeated-dna-sequences/description/)

标记难度：Medium

提交次数：2/3

代码效率：

* 普通的map：84.31%
* bitset+Rolling hash：99.74%

## 题意

有一个只包含ATGC的字符串序列，求其中全部长度为10的且出现了不止一次的序列。

## 分析

### 初步的想法

只要使用`unordered_map`作为Hash表，那就很简单了：只要依次遍历所有长度为10的字符串，然后把它们加入到`unordered_map`中，最后统计`unordered_map`中出现长度多于1次的序列即可。显然这个方法是可行的。

交上去之后我居然还写错了一次长度为10的子串的边界条件，所以wa了。

### 如何加速？

上述做法存在几个显而易见的问题：

* 最后需要多遍历一次`unordered_map`，能否节省这一次遍历？
* `unordered_map<string, int>`的速度是否太慢？
* 是否存在对`string`的更好的hash方法？

所以可以进行如下改进：

* 用两个Hash表来存储序列，一个存储的是所有出现过的序列，另一个只存储至少出现了两次的序列，这样就可以节省最后的一次遍历了。
* 改为采用[Rolling Hash](https://en.wikipedia.org/wiki/Rolling_hash)的方法滚动计算Hash值，并利用位运算进一步减少计算所需的时间。
* 由于对Hash值的范围进行了限定，因此可以改用其他的数据结构（如`bitset`）作为Hash表。

至于Rolling Hash如何和位运算结合起来，我见到的一种比较好的平衡了编写难度和运行时间的写法[^sample]是这样的：

* 把4个字符分别映射为0、1、2、3
* 对每一个新的序列计算Hash值时，先令`hash = (hash << 2) | charMap[ch]`
* 显然`hash`的长度固定为20bit，其最大值为`(1 << 20) - 1`（全为1），因此可以通过`hash &= (1 << 20) - 1`的方法来去掉最前面的项

此处使用的应该是Polynomial rolling hash，很类似于四进制的一种想法（也因此能保证hash函数是双射的）。

[^sample]: [sample 4 ms submission](https://leetcode.com/submissions/detail/180202097/)

---

从中至少可以学到几点：

1. 位运算是个很好用的东西（前提是没有写错）
2. `bitset`在适当的时候可以拿来代替Hash表或者大数组，但一般来说并不是一个关键数据结构

## 代码

### 普通的map

```cpp
class Solution {
public:
    vector<string> findRepeatedDnaSequences(string s) {
        int n = s.length();
        unordered_map<string, int> mmap;
        vector<string> ans;

        for (int i = 0; i <= n - 10; i++) {
            string str = s.substr(i, 10);
            mmap[str]++;
        }
        for (const auto& k: mmap) {
            if (k.second > 1)
                ans.push_back(k.first);
        }
        return ans;
    }
};
```

### bitset+Rolling hash

```cpp
// 参考了https://leetcode.com/submissions/detail/180202097/
class Solution {
public:
    vector<string> findRepeatedDnaSequences(string s) {
        int n = s.length();
        if (n < 11) return {};

        // cheap mapping
        int charToInt[200];
        charToInt['A'] = 0;
        charToInt['C'] = 1;
        charToInt['G'] = 2;
        charToInt['T'] = 3;

        const int MASK = (1 << 20) - 1;
        int rhash = 0;
        for (int i = 0; i < 9; i++)
            rhash = (rhash << 2) | charToInt[s[i]];

        // use bitset as hash table
        bitset<(1 << 20)> onceSet;
        bitset<(1 << 20)> twiceSet;
        vector<string> ans;
        for (int i = 9; i < n; i++) {
            rhash = (rhash << 2) | charToInt[s[i]];
            rhash &= MASK;
            // 简化代码逻辑
            if (twiceSet[rhash]) continue;
            if (onceSet[rhash]) {
                ans.push_back(s.substr(i - 9, 10));
                twiceSet.set(rhash);
            }
            else
                onceSet.set(rhash);
        }
        return ans;
    }
};
```
