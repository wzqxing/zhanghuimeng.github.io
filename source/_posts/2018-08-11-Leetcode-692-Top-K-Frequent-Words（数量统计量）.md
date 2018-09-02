---
title: Leetcode 692. Top K Frequent Words（数量统计量）
urlname: leetcode-692-top-k-frequent-words
toc: true
date: 2018-08-11 23:10:23
updated: 2018-08-12 00:03:23
tags: [Leetcode, alg:Hash Table, alg:Heap]
---

题目来源：[https://leetcode.com/problems/top-k-frequent-words/description/](https://leetcode.com/problems/top-k-frequent-words/description/)

标记难度：Medium

提交次数：3/3

代码效率：

* `O(n * log(n))`：36.84%
* `O(n * log(k))`：36.84%

## 题意

从一个数组中选出出现次数前k多的元素。

## 分析

显然很容易想到这种解法：用`map`来统计每种元素出现的数量（`O(n * log(n))`），然后用`priority_queue`来选择出现次数前k多的元素（`O(n * log(n))`），整体时间复杂度为`O(n * log(n))`。我们可以迅速改进这种算法的时间复杂度（虽然实际运行时间并没有多大改进）：用`unordered_map`来统计每种元素出现的数量（`O(n)`），然后在保证`priority_queue`中的元素不超过k个的前提下选择出现次数前k多的元素（`O(n * log(k))`），这样时间复杂度可以降低到`O(n * log(k))`。

我们还可以进行进一步的优化。[这篇题解](https://leetcode.com/problems/top-k-frequent-words/discuss/108399/Java-O%28n%29-solution-using-HashMap-BucketSort-and-Trie-22ms-Beat-81)指出，每种元素出现的数量的值最大为原数组的长度，因此我们并不需要动用优先队列来选择出现次数前k多的元素，用桶排序就可以了。为了保证先输出字典序小的元素，原作者在每个桶里搞了一棵Trie，因此这一处理过程的复杂度为`O(n * ave_len)`。

我想，用Trie处理字符串算是相当聪明的做法了——如果换成其他的有序数据结构或者排序算法，总归会有退化情况的。不过，如果字符串太长，处理时间仍然会变长。（不过我实在懒得去写了。）

### C++ STL代码技巧

通过阅读[这份代码](https://leetcode.com/problems/top-k-frequent-words/discuss/108366/O%28nlog%28k%29%29-Priority-Queue-C++-code)和查找资料，我感觉我又新学会了很多C++ STL的技巧。

#### map的一些特点

* 可以用[std::map::count](http://www.cplusplus.com/reference/map/map/count/)函数统计map中元素的个数。当然，map中的元素不可重复，因此这个函数的输出只可能为0或1。我把这个函数当做`find`的轻量版来用了……
* 使用`[]`操作符访问map中的元素时，如果这个元素不存在，会自动插入该元素并将值置为空。所以代码中直接`mmap[x]++`是可行的。（[What happens if I read a map's value where the key does not exist?](https://stackoverflow.com/questions/10124679/what-happens-if-i-read-a-maps-value-where-the-key-does-not-exist)）

#### 遍历map的几种方法

参考了[C++ Loop through Map](https://stackoverflow.com/questions/26281979/c-loop-through-map)。

1. 普通方法：迭代器

```cpp
map<string, int>::iterator it;

for ( it = symbolTable.begin(); it != symbolTable.end(); it++ )
{
    std::cout << it->first  // string (key)
              << ':'
              << it->second   // string's value
              << std::endl ;
}
```

2. C++ 11：`auto`和冒号（我感觉这种写法很像Java）

```cpp
for (auto const& x : symbolTable)
{
    std::cout << x.first  // string (key)
              << ':'
              << x.second // string's value
              << std::endl ;
}
```

注意上一种方法里ref的方式是`->`，这种方法里ref的方式是`.`。

3. C++ 17：这根本就是Python吧

```cpp
for( auto const& [key, val] : symbolTable )
{
    std::cout << key         // string (key)
              << ':'
              << val        // string's value
              << std::endl ;
}
```

#### 如何创建最小堆

我记得我之前曾经记录过，简单的创建一个最小堆的方法是：

```cpp
std::priority_queue<int, std::vector<int>, std::greater<int> > my_min_heap;
```

但是我没有考虑到如何为自建的类型创建比较方法。对于那些C算法库函数，写个返回`bool`的普通函数就行了，但对于C++ STL，显然没有这么简单——我果然还是不懂STL啊……简单来说，需要新建一个类，然后在里面重载`()`运算符，最后把这个类作为参数构造模板。下列代码来自[How can I create Min stl priority_queue?](https://stackoverflow.com/questions/2439283/how-can-i-create-min-stl-priority-queue)：

```cpp
#include <iostream>
#include <queue>
using namespace std;

struct compare
{
    bool operator()(const int& l, const int& r)
    {
        return l > r;
    }
};

 int main()
 {
     priority_queue<int,vector<int>, compare > pq;

     pq.push(3);
     pq.push(5);
     pq.push(1);
     pq.push(8);
     while ( !pq.empty() )
     {
         cout << pq.top() << endl;
         pq.pop();
     }
     cin.get();
 }
```

## 代码

### O(n * log(n))

```cpp
class Solution {
private:
    struct compare
    {
        bool operator()(const pair<int, string>& p1, const pair<int, string>& p2) {
            if (p1.first != p2.first) return p1.first < p2.first;
            return p1.second > p2.second;
        }
    };

public:
    vector<string> topKFrequent(vector<string>& words, int k) {
        map<string, int> mmap;
        for (string w: words) {
            if (mmap.count(w) == 1)
                mmap[w]++;
            else
                mmap[w] = 1;
        }
        priority_queue<pair<int, string>, std::vector<pair<int, string>>, compare> pq;
        for (auto const& x : mmap)
        {
            pq.push(make_pair(x.second, x.first));
        }

        vector<string> ans;
        for (int i = 0; i < k; i++) {
            ans.push_back(pq.top().second);
            pq.pop();
        }
        return ans;
    }
};
```

### O(n * log(k))

```cpp
class Solution {
private:
    struct compare
    {
        // 改为构造最小堆
        bool operator()(const pair<int, string>& p1, const pair<int, string>& p2) {
            if (p1.first != p2.first) return p1.first > p2.first;
            return p1.second < p2.second;
        }
    };

public:
    vector<string> topKFrequent(vector<string>& words, int k) {
        unordered_map<string, int> mmap;  // 即HashMap
        for (string w: words) {
            mmap[w]++;  // 直接+即可
        }
        priority_queue<pair<int, string>, std::vector<pair<int, string>>, compare> pq;
        for (auto const& x : mmap) {
            pq.push(make_pair(x.second, x.first));
            if (pq.size() > k)  // 需要保持pq的size不超过k，否则就是O(n * log(n))了
                pq.pop();
        }

        vector<string> ans;
        for (int i = 0; i < k; i++) {
            ans.insert(ans.begin(), pq.top().second);
            pq.pop();
        }
        return ans;
    }
};
```
