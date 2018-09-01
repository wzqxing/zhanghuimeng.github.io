---
title: Leetcode 225. Implement Stack using Queues（栈和队列）
urlname: leetcode-225-implement-stack-using-queues
toc: true
date: 2018-09-01 11:22:24
updated: 2018-09-01 15:31:00
tags: [Leetcode, Stack, Queue]
---

题目来源：[https://leetcode.com/problems/binary-search-tree-iterator/description/](https://leetcode.com/problems/binary-search-tree-iterator/description/)

标记难度：Easy

提交次数：1/1

代码效率：

* 2个队列（1）：100.00%
* 2个队列（2）：100.00%
* 1个队列：100.00%
* 用队列模拟链表：100.00%

## 题意

只通过队列实现栈。

## 分析

### 使用两个队列

我的第一反应就是弄两个队列，`push(x)`的时候， 把`x`放到第一个队列的队尾；`pop()`的时候，把第一个队列里的元素依次弹出再插入到第二个队列里，直到把最末一个元素弹出来，然后交换这两个队列。在这种实现下，`push(x)`的复杂度是`O(1)`，`pop()`的复杂度是`O(n)`。

另一种类似的思路也是使用两个队列，`push(x)`的时候，把`x`放入第二个队列的队尾（此时第二个队列为空），然后把第一个队列里的元素依次弹出，插入到第二个队列中，最后交换两个队列；`pop()`的时候直接弹出第一个队列的队头。在这种实现下，`push(x)`的复杂度是`O(n)`，`pop()`的复杂度是`O(1)`，元素在队列中的顺序正好和上一种做法相反。

另一个问题是，还有`top()`和`pop()`两种操作需要实现，第一种做法的`top()`操作也是`O(n)`的（因为栈顶元素在队尾，所以不得不全都重新移动一遍），所以可能第二种做法比较好。

### 使用一个队列

一种更简单的方法是只用一个队列，`push(x)`的时候，先把`x`放到队尾，然后把`x`前面的元素全都弹出，依次插入到`x`后面；`pop()`的时候，直接弹出队头。这种方法的复杂度和两个队列里的第二种是相同的。[^solution]

[^solution]: [Leetcode 255 Solution](https://leetcode.com/problems/implement-stack-using-queues/solution/)

### 用队列模拟链表

这是[StephanPochmann](https://leetcode.com/stefanpochmann/)提出的一种很有创意的方法。简单来说，就是用一个队列表示链表中的一项，队列中一共两个元素，第一个元素是数值，第二个元素是链表指针。然后就可以用链表来表示栈了。在这种做法中，插入和删除都是`O(1)`的。[^linkedlist]

[^linkedlist]: [O(1) purely with queues](https://leetcode.com/problems/implement-stack-using-queues/discuss/62522/O%281%29-purely-with-queues)

## 代码

### 两个队列（1）

```cpp
class MyStack {
private:
    queue<int> q[2];
    int cur;

public:
    /** Initialize your data structure here. */
    MyStack() {
        cur = 0;
    }

    /** Push element x onto stack. */
    void push(int x) {
        q[cur].push(x);
    }

    /** Removes the element on top of the stack and returns that element. */
    int pop() {
        int x;
        while (!q[cur].empty()) {
            x = q[cur].front();
            q[cur].pop();
            if (q[cur].empty())
                break;
            q[(cur + 1) % 2].push(x);
        }
        cur = (cur + 1) % 2;
        return x;
    }

    /** Get the top element. */
    int top() {
        int x = pop();
        push(x);
        return x;
    }

    /** Returns whether the stack is empty. */
    bool empty() {
        return q[cur].empty();
    }
};

/**
 * Your MyStack object will be instantiated and called as such:
 * MyStack obj = new MyStack();
 * obj.push(x);
 * int param_2 = obj.pop();
 * int param_3 = obj.top();
 * bool param_4 = obj.empty();
 */
// 使用一种O(n)的愚蠢的方法，把元素在两个队列之间移动……
```

### 两个队列（2）

```cpp
class MyStack {
private:
    queue<int> q[2];
    int cur;

public:
    /** Initialize your data structure here. */
    MyStack() {
        cur = 0;
    }

    /** Push element x onto stack. */
    void push(int x) {
        q[(cur + 1) % 2].push(x);
        while (!q[cur].empty()) {
            q[(cur + 1) % 2].push(q[cur].front());
            q[cur].pop();
        }
        cur = (cur + 1) % 2;
    }

    /** Removes the element on top of the stack and returns that element. */
    int pop() {
        int x = q[cur].front();
        q[cur].pop();
        return x;
    }

    /** Get the top element. */
    int top() {
        return q[cur].front();
    }

    /** Returns whether the stack is empty. */
    bool empty() {
        return q[cur].empty();
    }
};

/**
 * Your MyStack object will be instantiated and called as such:
 * MyStack obj = new MyStack();
 * obj.push(x);
 * int param_2 = obj.pop();
 * int param_3 = obj.top();
 * bool param_4 = obj.empty();
 */
```

### 一个队列

```cpp
class MyStack {
private:
    queue<int> q;

public:
    /** Initialize your data structure here. */
    MyStack() {

    }

    /** Push element x onto stack. */
    void push(int x) {
        q.push(x);
        for (int i = 0; i < q.size() - 1; i++) {
            q.push(q.front());
            q.pop();
        }
    }

    /** Removes the element on top of the stack and returns that element. */
    int pop() {
        int x = q.front();
        q.pop();
        return x;
    }

    /** Get the top element. */
    int top() {
        return q.front();
    }

    /** Returns whether the stack is empty. */
    bool empty() {
        return q.empty();
    }
};

/**
 * Your MyStack object will be instantiated and called as such:
 * MyStack obj = new MyStack();
 * obj.push(x);
 * int param_2 = obj.pop();
 * int param_3 = obj.top();
 * bool param_4 = obj.empty();
 */
```

### 用链表模拟队列

```cpp
class MyStack {
private:
    queue<void*>* head;

public:
    /** Initialize your data structure here. */
    MyStack() {
        head = NULL;
    }

    /** Push element x onto stack. */
    void push(int x) {
        queue<void*>* newNode = new queue<void*>();
        int* val = new int(x);
        newNode->push((void*) val);
        newNode->push((void*) head);
        head = newNode;
    }

    /** Removes the element on top of the stack and returns that element. */
    int pop() {
        int* val = (int*) head->front();
        head->pop();
        queue<void*>* newHead = (queue<void*>*) head->front();
        head->pop();

        int x = *val;
        delete val;
        delete head;
        head = newHead;
        return x;
    }

    /** Get the top element. */
    int top() {
        int* val = (int*) head->front();
        int x = *val;
        return x;
    }

    /** Returns whether the stack is empty. */
    bool empty() {
        return head == NULL;
    }
};

/**
 * Your MyStack object will be instantiated and called as such:
 * MyStack obj = new MyStack();
 * obj.push(x);
 * int param_2 = obj.pop();
 * int param_3 = obj.top();
 * bool param_4 = obj.empty();
 */
```
