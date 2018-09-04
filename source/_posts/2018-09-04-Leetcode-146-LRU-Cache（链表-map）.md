---
title: Leetcode 146. LRU Cache（链表+map）
urlname: leetcode-146-lru-cache
toc: true
date: 2018-09-04 15:14:43
updated: 2018-09-04 16:00:00
tags: [Leetcode, alg:Linked List, alg:Hash Table]
---

题目来源：[https://leetcode.com/problems/merge-two-sorted-lists/description/](https://leetcode.com/problems/merge-two-sorted-lists/description/)

标记难度：Hard

提交次数：1/1

代码效率：

* 手写链表+map：92.93%
* list+map：33.54%

## 题意

实现一个LRU缓存的功能。

## 分析

这个问题的标准答案显然是：

* 用一个`map<int, int>`维护`(key, value)`对
* 用一个双向链表维护访问顺序
* 用一个`map<int, Node*>`查询`key`在链表中对应的结点

然后具体操作是：

* 查询：在`map<int, int>`中查询有没有这个`key`，如果找到了，则在`map<int, Node*>`中找到它对应的`Node*`，在链表中把它的位置移到链表最前面
* 插入：
  * 如果不需要删除旧结点，则新建一个`Node*`，把它插入到链表最前面；维护`map<int, int>`和`map<int, Node*>`
  * 如果需要删除旧结点，则把链表最末端的`Node*`取出来并删掉，得到对应的`key`，在两个`map`中删除对应元素；然后再插入新结点

手写一个双向链表并不难，不过更好的方法是利用`std::list`，我之前从未用过。`list`满足链表的一般性质，也就是迭代器只会因为删除而失效，不会因为其他原因失效。[^list]

[^list]: [C++11 code 74ms - Hash table + List](https://leetcode.com/problems/lru-cache/discuss/45976/C++11-code-74ms-Hash-table-+-List)

## 代码

### 手写list

一个事实是，在使用了哨兵结点`head`和`tail`之后，在`deleteNode`、`insertBefore`和`insertAfter`函数中，其实都不需要检查空结点了。[^null]

[^null]: [Hashtable + Double linked list (with a touch of pseudo nodes)](https://leetcode.com/problems/lru-cache/discuss/45911/Java-Hashtable-+-Double-linked-list-%28with-a-touch-of-pseudo-nodes%29)

```cpp
class LRUCache {
private:
    struct Node {
        int key;
        Node* prev;
        Node* next;

        Node(int k) {
            key = k;
            prev = next = nullptr;
        }
    };

    void deleteNode(Node* node) {
        Node* prev = node->prev;
        Node* next = node->next;
        if (prev != nullptr)
            prev->next = next;
        if (next != nullptr)
            next->prev = prev;
    }

    void insertBefore(Node* node, Node* i) {
        Node* prev = node->prev;
        if (prev != nullptr) prev->next = i;
        i->prev = prev;
        i->next = node;
        node->prev = i;
    }

    void insertAfter(Node* node, Node* i) {
        Node* next = node->next;
        if (next != nullptr) next->prev = i;
        i->prev = node;
        i->next = next;
        node->next = i;
    }

    void moveToFront(Node* node) {
        deleteNode(node);
        insertAfter(head, node);
    }

    int capacity;
    int num;
    unordered_map<int, int> valMap;
    unordered_map<int, Node*> nodeMap;
    Node* head;
    Node* tail;

public:
    LRUCache(int capacity) {
        this->capacity = capacity;
        num = 0;
        head = new Node(-1);
        tail = new Node(-1);
        head->next = tail;
        tail->prev = head;
    }

    int get(int key) {
        auto i = valMap.find(key);
        if (i == valMap.end())
            return -1;
        moveToFront(nodeMap[key]);
        return i->second;
    }

    void put(int key, int value) {
        if (valMap.find(key) != valMap.end()) {
            valMap[key] = value;
            moveToFront(nodeMap[key]);
        }
        else {
            if (num < capacity) {
                Node* node = new Node(key);
                nodeMap[key] = node;
                valMap[key] = value;
                insertAfter(head, node);
                num++;
            }
            else {
                // evict out a least used one
                Node* node = tail->prev;
                deleteNode(node);
                nodeMap.erase(node->key);
                valMap.erase(node->key);

                node->key = key;
                nodeMap[key] = node;
                valMap[key] = value;
                insertAfter(head, node);
            }
        }
    }
};

/**
 * Your LRUCache object will be instantiated and called as such:
 * LRUCache obj = new LRUCache(capacity);
 * int param_1 = obj.get(key);
 * obj.put(key,value);
 */
```

### STL list

```cpp
class LRUCache {
private:
    int capacity;
    int num;
    unordered_map<int, int> valMap;
    unordered_map<int, list<int>::iterator> posMap;
    list<int> orderList;

    void moveToFront(int key) {
        orderList.erase(posMap[key]);
        orderList.push_front(key);
        posMap[key] = orderList.begin();
    }

public:
    LRUCache(int capacity) {
        this->capacity = capacity;
        num = 0;
    }

    int get(int key) {
        auto i = valMap.find(key);
        if (i == valMap.end())
            return -1;
        moveToFront(key);
        return i->second;
    }

    void put(int key, int value) {
        if (valMap.find(key) != valMap.end()) {
            valMap[key] = value;
            moveToFront(key);
        }
        else {
            if (num < capacity) {
                valMap[key] = value;
                orderList.push_front(key);
                posMap[key] = orderList.begin();
                num++;
            }
            else {
                // evict out a least used one
                int evict = orderList.back();
                orderList.pop_back();
                valMap.erase(evict);
                posMap.erase(evict);

                valMap[key] = value;
                orderList.push_front(key);
                posMap[key] = orderList.begin();
            }
        }
    }
};

/**
 * Your LRUCache object will be instantiated and called as such:
 * LRUCache obj = new LRUCache(capacity);
 * int param_1 = obj.get(key);
 * obj.put(key,value);
 */
```
