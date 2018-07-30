---
title: Leetcode 21. Merge Two Sorted Lists（链表操作）
urlname: leetcode-21-merge-two-sorted-lists
toc: true
date: 2018-07-30 22:58:58
updated: 2018-07-30 22:58:58
tags: [Leetcode]
---

题目来源：[https://leetcode.com/problems/merge-two-sorted-lists/description/](https://leetcode.com/problems/merge-two-sorted-lists/description/)

标记难度：Easy

提交次数：1/1

代码效率：100.00%

## 题意

合并两个已排序的链表。

## 分析

注意边界情况。以及和链表搞来搞去实在并不能算是一件很有趣味的事情。

## 代码

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
        ListNode* head = new ListNode(-1);  // 新增的头结点，没有实际意义。当然也可以不加，写起来更麻烦一点。
        ListNode* p = head;
        // 其实我刚才写了半天才发现自己理解错了。我以为要求是把两个list交替拼接起来。
        // 但实际上是合并排序……
        while (l1 != NULL && l2 != NULL) {
            if (l1->val < l2->val) {
                p->next = l1;
                l1 = l1->next;
                p = p->next;
            }
            else {
                p->next = l2;
                l2 = l2->next;
                p = p->next;
            }
        }
        if (l1 != NULL)
            p->next = l1;
        else if (l2 != NULL)
            p->next = l2;

        p = head->next;
        delete head;
        return p;
    }
};
```
