---
title: Leetcode 232. Implement Queue using Stacks（栈和队列）
urlname: leetcode-232-implement-queue-using-stacks
toc: true
date: 2018-09-05 15:30:06
updated: 2018-09-05 16:54:00
tags: [Leetcode, alg:Stack, alg:Queue]
---

题目来源：[https://leetcode.com/problems/implement-queue-using-stacks/description/](https://leetcode.com/problems/implement-queue-using-stacks/description/)

标记难度：Easy

提交次数：3/3

代码效率：

* 我的做法（两个栈）：100.00%
* 稍微好一点的双栈版本：100.00%
* 平摊O(1)的双栈版本：100.00%

## 题意

用栈实现队列。

## 分析

这道题和[Leetcode 225](/post/leetcode-225-implement-stack-using-queues)正好对应。不过，栈和队列相比，只有一个出口，所以解法也少一种。

### 法1：我的做法

我的做法特别简单直接明了，用一个栈（`s1`）模拟队列（栈顶表示队尾，栈底表示队头），另一个栈（`s2`）作为临时存储空间。

* `push(x)`：直接把`x`放入`s1`栈顶，时间复杂度为`O(1)`。
* `pop()`：把`s1`中的元素逐个弹出并插入到`s2`中，直到得到位于`s1`栈底的队头元素；然后把`s2`中的元素再逐个弹出，放回`s1`中，时间复杂度是`O(n)`（因为这里的数据结构是栈，所以临时存储会改变元素的顺序，按照这种思路，只能把元素再重新放回去了）。
* `peek()`：和`pop()`的过程基本相同，但不删除队头元素，时间复杂度为`O(n)`。
* `empty()`：返回`s1.empty()`，时间复杂度为`O(1)`。

这种做法虽然不算很错，但显然不是一种很聪明的办法：因为栈只有一端是开口的，我还把队头放在不开的那一头，所以`pop()`和`peek()`都要花费`O(n)`的时间，相当离谱。

### 法2：较少的时间复杂度

这种做法的思路和上一种做法很相似，也是用一个栈（`s1`）模拟队列，另一个栈（`s2`）作为临时存储空间；主要的区别是，此时用栈顶表示队头，栈底表示队尾。[^solution]

* `push(x)`：把`s1`中的元素逐个弹出并插入到`s2`中，把`x`插入`s1`中，最后再把`s2`中的元素再逐个弹出，放回`s1`中。时间复杂度是`O(n)`。

![push操作](232_queue_using_stacksBPush.png)

* `pop()`：直接从`s1`栈顶弹出元素，时间复杂度是`O(1)`。

![pop操作](232_queue_using_stacksBPop.png)

* `peek()`：返回`s1`栈顶元素，时间复杂度是`O(1)`。
* `empty()`：返回`s1.empty()`，时间复杂度是`O(1)`。

[^solution]: [232. Implement Queue using Stacks](https://leetcode.com/articles/implement-queue-using-stacks/)（本文中的图片都来自这里）

### 法3：平摊O(1)复杂度

这种做法大约是法1的改进版。可以发现，在法1里，进行`pop()`的过程中，元素在`s2`中排成了正确的顺序，队头在栈顶。那我们不妨就把这些元素留在`s1`里面。并且用`front`变量维护`s1`栈顶元素。

* `push(x)`：直接把`x`放入`s1`栈顶，维护`front`，时间复杂度是`O(1)`。

![push操作](232_queue_using_stacksAPush.png)

* `pop()`：如果`s2`为空，则把`s1`中的元素逐个弹出并插入到`s2`中；从`s2`中弹出栈顶元素。显然，单次时间复杂度最坏可能是`O(n)`，但并不是每次都需要花这么多时间；事实上，每个元素只会进入`s1`一次，然后再进入`s2`一次，所以整体的时间复杂度只有`O(1)`；平摊到每次操作上，就是`O(1)`。

![pop操作](232_queue_using_stacksAPop.png)

* `peek()`：如果`s2`不为空，则返回`s2`栈顶元素；否则返回`front`。时间复杂度为`O(1)`。
* `empty()`：返回`s1.empty() && s2.empty()`。时间复杂度是`O(1)`。

以及，我感觉用栈模拟链表再模拟队列的做法应该也是可行的，不过既然以及有了平摊`O(1)`的方法，似乎没有这个必要了。

## 代码

### 法1

```cpp
class MyQueue {
private:
    stack<int> s1, s2;

public:
    MyQueue() {

    }

    void push(int x) {
        s1.push(x);
    }

    int pop() {
        while (!s1.empty()) {
            s2.push(s1.top());
            s1.pop();
        }
        int x = s2.top();
        s2.pop();
        while (!s2.empty()) {
            s1.push(s2.top());
            s2.pop();
        }
        return x;
    }

    int peek() {
        while (!s1.empty()) {
            s2.push(s1.top());
            s1.pop();
        }
        int x = s2.top();
        while (!s2.empty()) {
            s1.push(s2.top());
            s2.pop();
        }
        return x;
    }

    bool empty() {
        return s1.empty();
    }
};
```

### 法2

```cpp
class MyQueue {
private:
    stack<int> s1, s2;

public:
    MyQueue() {
    }

    void push(int x) {
        while (!s1.empty()) {
            s2.push(s1.top());
            s1.pop();
        }
        s1.push(x);
        while (!s2.empty()) {
            s1.push(s2.top());
            s2.pop();
        }
    }

    int pop() {
        int x = s1.top();
        s1.pop();
        return x;
    }

    int peek() {
        return s1.top();
    }

    bool empty() {
        return s1.empty();
    }
};
```

### 法3

```cpp
class MyQueue {
private:
    stack<int> s1, s2;

public:
    MyQueue() {
    }

    void push(int x) {
        s1.push(x);
    }

    int pop() {
        if (s2.empty()) {
            while (!s1.empty()) {
                s2.push(s1.top());
                s1.pop();
            }
        }
        int x = s2.top();
        s2.pop();
        return x;
    }

    // 我觉得不维护front变量，而把peek写成和pop类似的形式并不会改变整体的平摊时间复杂度
    // 但是可能会增加peek的访问时间
    int peek() {
        if (s2.empty()) {
            while (!s1.empty()) {
                s2.push(s1.top());
                s1.pop();
            }
        }
        return s2.top();
    }

    bool empty() {
        return s1.empty() && s2.empty();
    }
};
```
