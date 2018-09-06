---
title: Leetcode 460. LFU Cache（STL）
urlname: leetcode-460-lfu-cache
toc: true
date: 2018-09-05 18:57:45
updated: 2018-09-06 15:37:00
mathjax: true
tags: [Leetcode, alg:Priority Queue, alg:Linked List, alg:Hash Table, alg:Binary Search Tree]
---

题目来源：[https://leetcode.com/problems/merge-two-sorted-lists/description/](https://leetcode.com/problems/merge-two-sorted-lists/description/)

标记难度：Easy

提交次数：4/9

代码效率：

* map O(n)：3.13%
* HashMap + list O(1)：45.28%
* TreeSet + map O(log(n))：23.78%
* 简单版map + list O(1)：11.36%

## 题意

实现LFU缓存。

## 分析

### map O(n)

这是我想出来的第一种做法：用三个`map`分别记录每个`key`对应的值（`valueMap`）、访问次数（`freqMap`）和最近访问时间（`recentMap`）。

* `get(int key)`：在`valueMap`中查找`key`对应的值，然后更新`freqMap`和`recentMap`。
* `put(int key, int value)`：
  * 如果不需要换出，则直接更新三个`map`中维护的值
  * 如果需要换出，则遍历当前的所有`key`，从中找出访问频率最小的最不频繁访问的元素，把它换出，再插入新元素

这种做法显然不是很聪明。

### HashMap + list O(1)

这是一种相对比较复杂的方法，原理是这样的[^bucket]：

```
Increasing frequencies
---------------------------------->

+------+    +---+    +---+    +---+
| Head |----| 1 |----| 5 |----| 9 |  Frequencies
+------+    +-+-+    +-+-+    +-+-+
              |        |        |
            +-+-+    +-+-+    +-+-+     |
            |2,3|    |4,3|    |6,2|     |
            +-+-+    +-+-+    +-+-+     | Most recent
                       |        |       |
                     +-+-+    +-+-+     |
 key,value pairs     |1,2|    |7,9|     |
                     +---+    +---+     v
```

[^bucket]: [C++ list with hashmap with explanation](https://leetcode.com/problems/lfu-cache/discuss/94602/C%2B%2B-list-with-hashmap-with-explanation/172522)

作者在代码中维护了两个数据结构：

* `unordered_map<int, pair<list<LRUNode>::iterator, list<pair<int, int>>::iterator>> kv_`：存放每个`key`对应的桶的头结点，以及`key`在桶中对应的具体结点
* `list<LRUNode> cache_`：所有桶组成的链表；其中`LRUNode = (int freq, list<pair<int, int>> vals)`，表示频率`freq`的`key`组成的链表，其中`pair`的第一项表示`key`，第二项表示`value`。

具体实现的核心是三个成员函数：

* `promote(int key, int val)`：找到`key`对应的桶的头结点`i`和`key`在桶中对应的结点`j`，以及下一个频率对应的桶`k`，在有必要的情况下取出`j->second`；然后从`i`中删除`j`，把`i`插入到`k`的最末端。
* `evict()`：找到频率最小的桶`i`，以及位于桶头部的访问最不频繁的结点`j`，然后从`i`和`kv_`中删除`j`
* `insert(int key, int val)`：找到频率最小的桶`i`，如果`i`对应的频率不是`1`，则在`cahce_`头部插入一个新的桶，对应的频率为`1`；最后在该桶的末端插入`(key, val)`。

利用上述函数很容易实现`get`和`put`，我就不写了。

---

我的实现方法则比较复杂。维护4个`map`和一个`list`：

* `unordered_map<int, int> valMap`：存放`key`和`value`的对应关系
* `unordered_map<int, int> freqMap`：存放`key`和访问频率的对应关系
* `unordered_map<int, list<list<int>>::iterator> headMap`：存放访问频率和桶的对应关系
* `unordered_map<int, list<int>::iterator> nodeMap`：存放`key`和桶中结点的对应关系
* `list<list<int>> buckets`：桶

实现了一个更新函数`touch(int key)`，主要功能是将`key`的访问频率+1，即把`key`从`headMap[freqMap[key]]`对应的桶中删除，放入`headMap[freqMap[key] + 1]`对应的桶中，并更新各`map`之间的关系。

* `get(int key)`：在`valMap`中查找`key`，如果`key`不存在则返回`-1`；如果存在则执行`touch(key)`，然后返回`valMap[key]`。
* `put(int key, int value)`：
  * 如果`key`已经存在，则执行`touch(key)`，并更新`valMap`。
  * 如果`key`不存在且需要换出，则在`buckets`中找到当前对应频率值最低的桶，将桶中最末一个元素删除，并更新其他`map`。
  * 最后将（还不存在的）`key`插入访问频率1对应的桶中，更新其他`map`。

### TreeSet + map O(log(n))

上一种方法实在是太繁琐了，于是我又发现了另一种利用`TreeSet`（或者说优先队列）的方法。[^treeset]

* 维护一个集合`treeSet`（对C++来说，用`priority_queue`有一些缺陷，所以此处实际上用的是`set`），其中的元素是`(frequency, recency, key)`
* `unordered_map<int, int> valMap`，用于存储`key`到`value`的对应关系。
* `unordered_map<int, (frequency, recency, key)> keyMap`，用于存储`key`到`freq`和`recency`的对应关系。

实现一种`touch(int key)`操作：在`treeSet`中删除当前的`keyMap[key]`，更新`key`的访问频率（`keyMap[key].frequency++`），并将更新后的`keyMap[key]`重新插入到`treeSet`中。

* `get(int key)`：在`valMap`中查找`key`，如果找不到则返回`-1`；否则执行`touch(key)`。
* `put(int key, int value)`：
  * 如果`key`已经存在，则执行`touch(key)`。
  * 如果`key`不存在且需要换出，则通过`treeSet->begin()`找到当前的LFU`key`，从`treeSet`和其他`map`中删除该`key`对应的元素。
  * 最后，令`keyMap[key] = (1, curTime, key)`，`valMap[key] = value`，将`keyMap[key]`插入到`treeSet`中。

由于`treeSet`中按值查找、删除和插入的复杂度都是`O(log(n))`，所以这种方法的时间复杂度是`O(log(n))`，比之前的方法稍微高一点，不过写起来方便多了。

以及，随便说一下C++中鸡肋一般的`priority_queue`。事实上，像这道题中这样的需求——插入元素、寻找最小元素、删除最小元素、元素下滤（显然对元素的操作只会使`frequency`和`recency`增加）——是很适合堆的。然而C++中并没有为`priority_queue`提供修改其中元素的接口，如果想要实现，大概只能自己继承这个类然后重写。Java中的`PriorityQueue`倒是有`remove`接口，但时间复杂度是`O(n)`。那还不如用`set`（Java中的`TreeSet`）和运算符重载算了。

不过我之前从未想过，`set`可以实现和`priority_queue`类似的功能这回事，而且时间复杂度差不多。嗯，只是差不多而已，此处又想起了邓公的话：

>由上可见，上滤调整过程中交换操作的累计次数，不致超过全堆的高度$\lfloor \log_2{(n)} \rfloor$。 而在向量中， 每次交换操作只需常数时间，故上滤调整乃至整个词条插入算法整体的时间复杂度，均为$O(logn)$。这也是从一个方面，兑现了10.1节末尾就优先级队列性能所做的承诺。
>当然，不难通过构造实例说明，新词条有时的确需要一直上滤至堆顶。然而实际上，此类最坏情况通常极为罕见。以常规的随机分布而言，新词条平均需要爬升的高度，要远远低于直觉的估计（习题\[10-6\]）。[^dsapp]

[^dsapp]: 《数据结构(C++语言版)》（第三版） P290，清华大学出版社，2013.9

不过，我觉得在这道题的语境下，优先队列的上滤过程恐怕没有那么大的优势（因为插入的新元素的`frequency`必然为`1`，因此很可能会上升到堆中比较高的位置），下滤过程倒可能有类似的效果（因为下滤时作为排序的第一关键字的`frequency`只增加了`1`，因此很可能不需要下降很多层）。

[^treeset]: [Java solution using PriorityQueue, with detailed explanation](https://leetcode.com/problems/lfu-cache/discuss/94536/Java-solution-using-PriorityQueue-with-detailed-explanation)

### 简单版map + list O(1)

这是最简单的一种方法了。本质上也是利用桶排序，但是这种方法利用了一个我们之前都没有注意到的事实：插入新`key`时，它的`frequency`必然是1。因此我们并不需要一个有序集或一个链表来维护最小的访问频率，因为在删除一个访问频率最小的结点之后，插入的新结点的频率（1）必然是现在的最小频率。这样，我们就可以用一个`unordered_map<int, list<int>>`来存储访问频率和桶的对应的关系了。

其余的想法和第一种桶排序是非常类似的。具体内容看代码和注释好了。

---

有一件很有趣的事情。在这种方法中，我仍然需要维护一个`unordered_map<int, list<int>::iterator> iterMap`，用于从桶中删除元素；但是Java里有一个比较神奇的东西，叫[LinkedHashSet](http://docs.oracle.com/javase/7/docs/api/java/util/LinkedHashSet.html)，同时实现了链表和哈希表的特性，所以Java代码会看起来好看一点。[^javaset]不过也许它内部也就是把一个`HashSet<Type, ListNode>`和一个`ListNode<Type>`这样拼起来的。

[^javaset]: [JAVA O(1) very easy solution using 3 HashMaps and LinkedHashSet](https://leetcode.com/problems/lfu-cache/discuss/94521/JAVA-O%281%29-very-easy-solution-using-3-HashMaps-and-LinkedHashSet)

## 代码

### map O(n)

```cpp
class LFUCache {
private:
    unordered_map<int, int> valueMap;
    unordered_map<int, int> freqMap;
    unordered_map<int, int> recentMap;

    int time;
    int cap;
    int num;

public:
    LFUCache(int capacity) {
        time = 0;
        num = 0;
        cap = capacity;
    }

    int get(int key) {
        auto i = valueMap.find(key);
        if (i == valueMap.end())
            return -1;

        recentMap[key] = time;
        freqMap[key]++;

        time++;
        return i->second;
    }

    void put(int key, int value) {
        // 这是一个不容易注意到的corner case
        // 虽然毫无意义。
        if (cap == 0)
            return;

        if (valueMap.find(key) != valueMap.end()) {
            valueMap[key] = value;
            freqMap[key]++;
            recentMap[key] = time;

            time++;
            return;
        }
        if (num >= cap) {
            // find LFU
            int minKey = -1, minFreq, minRec;
            // 遍历所有的key，找到其中freq和recent最小的
            for (auto const& i: freqMap) {
                int freq = i.second;
                int recent = recentMap[i.first];
                if (minKey == -1 || freq < minFreq || (freq == minFreq && recent < minRec)) {
                    minKey = i.first;
                    minFreq = freq;
                    minRec = recent;
                }
            }
            // cout << minKey << ' ' << minFreq << ' ' << minRec << endl;
            valueMap.erase(minKey);
            freqMap.erase(minKey);
            recentMap.erase(minKey);
            num--;
        }

        valueMap[key] = value;
        freqMap[key]++;
        recentMap[key] = time;
        num++;

        time++;
        return;
    }
};
```

### HashMap + list O(1)

```cpp
class LFUCache {
private:
    unordered_map<int, int> valMap;  // key --> value
    unordered_map<int, int> freqMap;  // key --> frequence
    unordered_map<int, list<list<int>>::iterator> headMap;  // frequence --> bucket head
    unordered_map<int, list<int>::iterator> nodeMap;  // key --> node in list
    list<list<int>> buckets;  // buckets

    int cap;
    int num;

    /* increase the frequence of @key */
    void touch(int key) {
        int freq = freqMap[key];
        auto head = headMap[freq];  // head of the bucket containing key
        auto node = nodeMap[key];  // the list node containing key
        head->erase(node);  // delete the key from this bucket
        // find new bucket head
        auto newHead = head;
        if (headMap.find(freq + 1) != headMap.end())
            newHead = headMap[freq + 1];
        else {
            // Note: that's "insert before" for C++
            buckets.insert(head, list<int>());
            newHead--;
        }
        // insert the key into another bucket
        // put the most recent values in the front
        newHead->push_front(key);
        // delete empty bucket (for convenience of finding LFU)
        if (head->empty()) {
            buckets.erase(head);
            headMap.erase(freq);
        }
        // update the maps
        freqMap[key]++;
        headMap[freq + 1] = newHead;
        nodeMap[key] = newHead->begin();
    }

public:
    LFUCache(int capacity) {
        cap = capacity;
        num = 0;
    }

    int get(int key) {
        if (valMap.find(key) == valMap.end()) return -1;
        touch(key);  // update frequence
        return valMap[key];
    }

    void put(int key, int value) {
        if (cap == 0) return;

        // Only update value
        if (valMap.find(key) != valMap.end()) {
            valMap[key] = value;
            touch(key);
            return;
        }
        // Need to evict a LFU key-value pair
        if (num >= cap) {
            // find the head of the bucket of minimum frequency
            auto head = buckets.end();
            head--;
            // find least visited key of this frequency and delete it
            int evict = head->back();
            head->pop_back();
            int freq = freqMap[evict];
            // if this bucket becomes empty, then delete it
            if (head->empty()) {
                headMap.erase(freq);
                buckets.pop_back();
            }
            // delete the evicted key
            valMap.erase(evict);
            freqMap.erase(evict);
            nodeMap.erase(evict);
            num--;
        }

        // find the head of frequency 1
        if (headMap.find(1) == headMap.end()) {
            buckets.push_back(list<int>());
            auto head = buckets.end(); head--;
            headMap[1] = head;
        }
        // insert new key at the front of list
        auto head = headMap[1];
        head->push_front(key);
        nodeMap[key] = head->begin();
        valMap[key] = value;
        freqMap[key] = 1;
        num++;
    }
};
```

### TreeSet + map O(log(n))

```cpp
class LFUCache {
private:
    struct Tuple {
        int key;
        int freq;
        int recent;

        Tuple() {}

        Tuple(int x, int y, int z): key(x), freq(y), recent(z) {}

        // For ordered set
        friend bool operator < (const Tuple& a, const Tuple& b) {
            if (a.freq != b.freq) return a.freq < b.freq;
            return a.recent < b.recent;
        }

        friend bool operator == (const Tuple& a, const Tuple& b) {
            return a.key==b.key && a.freq==b.freq && a.recent==b.recent;
        }
    };

    unordered_map<int, int> valMap;
    unordered_map<int, Tuple> keyMap;
    set<Tuple> treeSet;
    int num;
    int cap;
    int time;

public:
    LFUCache(int capacity) {
        num = 0;
        time = 0;
        cap = capacity;
    }

    int get(int key) {
        if (valMap.find(key) == valMap.end()) return -1;
        // Erase old Tuple and insert new one
        treeSet.erase(keyMap[key]);
        keyMap[key].freq++;
        keyMap[key].recent = time;
        treeSet.insert(keyMap[key]);
        time++;
        return valMap[key];
    }

    void put(int key, int value) {
        if (cap <= 0) return;

        if (valMap.find(key) != valMap.end()) {
            treeSet.erase(keyMap[key]);
            keyMap[key].freq++;
            keyMap[key].recent = time;
            valMap[key] = value;
            treeSet.insert(keyMap[key]);
        }
        else {
            if (num >= cap) {
                // Find LFU (ordered set head)
                auto iter = treeSet.begin();
                int evict = iter->key;
                treeSet.erase(iter);
                valMap.erase(evict);
                keyMap.erase(evict);
                num--;
            }
            keyMap[key] = Tuple(key, 1, time);
            treeSet.insert(keyMap[key]);
            valMap[key] = value;
            num++;
        }
        time++;
    }
};
```

### 简单版map + list O(1)

```cpp
class LFUCache {
private:
    unordered_map<int, int> valMap;
    unordered_map<int, int> freqMap;
    unordered_map<int, list<int>::iterator> iterMap;
    unordered_map<int, list<int>> bucketMap;
    int minFreq;
    int cap;
    int num;

    // visit @key
    void touch(int key) {
        int freq = freqMap[key];
        auto iter = iterMap[key];
        bucketMap[freq].erase(iter);  // erase key from old bucket
        if (freq == minFreq && bucketMap[freq].empty())  // update minFreq
            minFreq++;
        bucketMap[freq + 1].push_front(key);  // insert key to new bucket
        iterMap[key] = bucketMap[freq + 1].begin();
        freqMap[key]++;
    }

public:
    LFUCache(int capacity) {
        cap = capacity;
        num = 0;
        minFreq = 1;
    }

    int get(int key) {
        if (freqMap.find(key) == freqMap.end()) return -1;
        touch(key);
        return valMap[key];
    }

    void put(int key, int value) {
        if (cap == 0) return;

        if (freqMap.find(key) != freqMap.end()) {
            valMap[key] = value;
            touch(key);
            return;
        }
        if (num >= cap) {
            int evict = bucketMap[minFreq].back();  // find LFU
            // delete LFU
            bucketMap[minFreq].pop_back();
            valMap.erase(evict);
            freqMap.erase(evict);
            iterMap.erase(evict);
            num--;
        }
        // because the freq of new key is 1, minFreq will certainly become 1
        minFreq = 1;
        bucketMap[1].push_front(key);
        valMap[key] = value;
        freqMap[key] = 1;
        iterMap[key] = bucketMap[1].begin();
        num++;
    }
};
```
