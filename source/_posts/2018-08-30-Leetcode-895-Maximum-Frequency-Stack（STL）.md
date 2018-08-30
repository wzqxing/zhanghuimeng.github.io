---
title: Leetcode 895. Maximum Frequency Stack（STL）
urlname: leetcode-895-maximum-frequency-stack
toc: true
date: 2018-08-30 15:48:06
updated: 2018-08-30 19:29:06
tags: [Leetcode, LeetcodeContest]
---

题目来源：[https://leetcode.com/problems/maximum-frequency-stack/description/](https://leetcode.com/problems/maximum-frequency-stack/description/)

标记难度：Hard

提交次数：2/3

代码效率：

* map+heap：41.40%
* vector+stack+map：66.98%

## 题意

实现一种名为`freqStack`的数据结构，提供以下两种功能：

* `push(int x)`：将整数`x`入栈
* `pop()`：删除并返回栈中出现次数最多的元素；如果有平局，则删除最靠近栈顶的元素。

单个测试样例中每种操作的数量均不会超过`10000`。

## 分析

这是周赛（99）的第四题。比赛的时候我一直在死磕第三题，所以没有时间做这道题，但是后来我发现这道题还挺简单的，花了不到半个小时就做出来了。

---

### map+heap

我感觉题意中有一个地方说得不够清楚，不过在之后的例子中解决了这种模糊性：对于出现次数最多的那些元素，我们删除的是最靠近栈顶的一个。

我的做法是这样的：

* 使用一个`map<int, int>`记录每种元素的个数（`cntMap`）
* 使用一个`map<int, vector<int>>`记录每种元素的位置（`posMap`）
* 使用一个`priority_queue<(cnt, lastPos, x)>`记录和寻找栈中出现次数最多的元素（`heap`）

对于`push(x)`操作，`cntMap[x]++`，并为`x`保存当前栈顶位置（不妨假定我们用一个数组来维护栈，且删除操作不回收空间），作为`x`的出现位置之一：`posMap[x].push_back(curPos)`。最后将元组`(cnt, lastPos, x)`存入`heap`中。（我知道C++现在还没有tuple，但是实际上就是这么个东西。）复杂度为`O(log(N))`（其中`N`表示总操作次数）。

对于`pop()`操作，从`heap`中取出堆顶元素，检查其内容是否合法（`cnt == cntMap[x] && lastPos == posMap[x].back()`），直到找到一组合法的`(cnt, lastPos, x)`。然后删除`x`最靠近栈顶的出现位置，`cntMap[x]--`，`posMap[x].pop_back()`。复杂度还是`O(log(N))`。

C++中没有为`priority_queue`提供方便的删除函数，所以上述代码中每插入一个元素，都不计代价地向`priority_queue`中插入一个新的元组，而在`pop()`操作中需要检查元组的合法性。因为操作数量只有`10000`，所以这种操作在此题中还不会导致内存爆炸。

这种奇怪的操作手法是我从Dijstra最短路算法的C++ STL实现里学到的[^dijstra]。但是，如果真的会内存爆炸的话，考虑用便于删除的`set`代替`priority_queue`比较好[^set]。

[^dijstra]: [Dijkstra’s Shortest Path Algorithm using priority_queue of STL](https://www.geeksforgeeks.org/dijkstras-shortest-path-algorithm-using-priority_queue-stl/)

[^set]: [Dijkstra’s shortest path algorithm using set in STL](https://www.geeksforgeeks.org/dijkstras-shortest-path-algorithm-using-set-in-stl/)

### vector+stack+map

我觉得这是一种很非常有趣的做法。据说采用的是桶排序的思想。[^bucket]

[^bucket]: [JAVA O(1) solution easy understand using bucket sort](https://leetcode.com/problems/maximum-frequency-stack/discuss/163453/JAVA-O%281%29-solution-easy-understand-using-bucket-sort)

需要的数据结构包括：

* 一个`vector<stack<int>>`：
  * `vector`是元素出现频率的“桶”
  * `stack<int>`存储的是该频率下出现的所有元素，通过`stack`维护了元素之间的顺序性
* 一个`map<int, int>`：存储某一元素在栈中的当前频率

对于`push(x)`操作，首先在`map`中更新该元素的频率，然后在更新后的频率对应的桶中插入该元素，并且更新`maxFreq`。

对于`pop()`操作，从`maxFreq`对应的桶中弹出栈顶元素`x`，并在`map`中更新该元素的频率；如果该桶变空，则删除该桶，并更新`maxFreq`。

虽然我觉得这种算法是正确的，但我很难直观地想象这种方法为何正确。显然，所有桶中的元素加起来构成了这个栈中的所有元素，但它们到底是以何种形式分布在不同桶里的呢？大概是“插入该元素时该元素的出现频率”，或者说“第几个这种元素”。在需要删除时，删除的是最多的元素中最靠近栈顶的一个，或者说是其中最后一个插入的。

## 代码

### map+heap

```cpp
class FreqStack {
private:
    struct Num {
        int cnt;
        int lastTime;
        int number;

        friend bool operator < (const Num& a, const Num& b) {
            if (a.cnt != b.cnt)
                return a.cnt < b.cnt;
            return a.lastTime < b.lastTime;
        }

        Num (const int& c, const int& l, const int& n) {
            cnt = c;
            lastTime = l;
            number = n;
        }
    };

    map<int, vector<int>> posMap;
    map<int, int> cntMap;
    priority_queue<Num> heap;
    int pos;

public:
    FreqStack() {
        pos = 0;
    }

    void push(int x) {
        posMap[x].push_back(pos);
        cntMap[x]++;
        heap.push(Num(cntMap[x], pos, x));
        pos++;
    }

    int pop() {
        while (!heap.empty()) {
            Num num = heap.top();
            heap.pop();

            // cout << heap.size() << endl;
            // cout << num.cnt << ' ' << num.lastTime << ' ' << num.number << endl;

            if (num.cnt != cntMap[num.number] || num.lastTime != posMap[num.number].back())
                continue;

            cntMap[num.number]--;
            posMap[num.number].pop_back();
            if (cntMap[num.number] > 0) {
                num.lastTime = posMap[num.number].back();
                num.cnt--;
                heap.push(num);
            }

            return num.number;
        }
    }
};

/**
 * Your FreqStack object will be instantiated and called as such:
 * FreqStack obj = new FreqStack();
 * obj.push(x);
 * int param_2 = obj.pop();
 */

// 用一个优先队列存储pair<times, lastTime, number>
// 再用一个map存储每个number出现的位置，以及具体次数
```

### vector+stack+map

```cpp
class FreqStack {
private:
    vector<stack<int>> bucket;
    unordered_map<int, int> freqMap;
    int maxFreq;

public:
    FreqStack() {
        maxFreq = 0;
    }

    void push(int x) {
        freqMap[x]++;
        int freq = freqMap[x];
        if (freq > maxFreq) {
            maxFreq = freq;
            bucket.push_back(stack<int>());
        }
        bucket[freq-1].push(x);
    }

    int pop() {
        int x = bucket[maxFreq - 1].top();
        bucket[maxFreq - 1].pop();
        if (bucket[maxFreq - 1].empty()) {
            maxFreq--;
            bucket.pop_back();
        }
        freqMap[x]--;
        return x;
    }
};

/**
 * Your FreqStack object will be instantiated and called as such:
 * FreqStack obj = new FreqStack();
 * obj.push(x);
 * int param_2 = obj.pop();
 */
```
