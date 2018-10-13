---
title: Leetcode 919. Complete Binary Tree Inserter（堆）
urlname: leetcode-919-complete-binary-tree-inserter
toc: true
date: 2018-10-11 00:40:02
updated: 2018-10-14 01:33:00
tags: [Leetcode, Leetcode Contest, alg:Tree, alg:Heap]
---


题目来源：[https://leetcode.com/problems/complete-binary-tree-inserter/description/](https://leetcode.com/problems/complete-binary-tree-inserter/description/)

标记难度：Medium

提交次数：4/6

代码效率：

* vector：78.93%
* queue：48.93%

## 题意

给定一个完全二叉树，写一个向完全二叉树中插入元素的操作类。

## 分析

比赛的时候觉得这道题很水，随便写就好了。事实上确实差不多是这样的。

---

### vector

既然是完全二叉树的插入，那么可以利用数组保存堆的思想。如果根在1处，则`x`的左孩子在`2 * x`处，右孩子在`2 * x + 1`处；`x`的父亲在`floor(x / 2)`处。于是直接用一个vector做就可以了。预处理花费`O(N)`时间（因为需要把树中当前的结点都插入）；插入花费`O(1)`时间。听起来还可以。

### queue

这个方法来自[^solution]，和上一个解法差不多，都利用了完全二叉树只有最低一层可能不满，只有最低两层的结点可能只有`<2`个子结点的特性：

* 用一个队列维护当前还没有获得两个叶子结点的`TreeNode*`
* 插入新结点时，尝试插入到队头的结点的左侧或右侧（左侧优先）；然后再把新结点插入队尾
* 如果队头的结点已经有了两个子结点，则把它弹出

不过我看不出题解里使用deque的意义何在……

[^solution]: [Official Solution for Leetcode 919](https://leetcode.com/problems/complete-binary-tree-inserter/solution/)

### 直接计算

通过阅读12ms的示例代码，我发现了一种比较神奇的解法：不把整棵树缓存下来，直接在树上计算下一个结点的父节点位置。[^sample]

[^sample]: [Leetcode 919: sample 12 ms submission](https://leetcode.com/submissions/detail/181827960/)

这种做法的思路大致是这样的：

* 初始化：通过dfs计算结点数量`counter`
* 插入结点：
  * 根据`counter`计算当前树中层数`numRows`
  * 根据`counter`和`numRows`计算出当前结点的父结点是倒数第二层（或最后一层）的第几个结点
  * 通过二分查找找出这个结点并插入

具体内容实在没有时间去写了。

## 代码

### vector

其中初始化时用`vector`代替`queue`的trick来自[^lee]。

[^lee]: [lee215's Solution for Leetcode 919: O(1) Insert](https://leetcode.com/problems/complete-binary-tree-inserter/discuss/178424/C++JavaPython-O%281%29-Insert)

```cpp
class CBTInserter {
private:
    vector<TreeNode*> heap;

public:
    CBTInserter(TreeNode* root) {
        heap.push_back(nullptr);
        heap.push_back(root);

        for (int i = 1; i < heap.size(); i++) {
            TreeNode* x = heap[i];
            if (heap[i]->left != nullptr) heap.push_back(heap[i]->left);
            if (heap[i]->right != nullptr) heap.push_back(heap[i]->right);
        }
    }

    int insert(int v) {
        int n = heap.size();
        TreeNode* x = new TreeNode(v);
        heap.push_back(x);
        int m = n / 2;
        if (n % 2 == 0) heap[m]->left = x;
        else heap[m]->right = x;
        return heap[m]->val;
    }

    TreeNode* get_root() {
        return heap[1];
    }
};
```

### queue

```cpp
class CBTInserter {
private:
    queue<TreeNode*> q;
    TreeNode* root;

public:
    CBTInserter(TreeNode* root) {
        q.push(root);
        this->root = root;

        while (!q.empty()) {
            TreeNode* p = q.front();
            if (p->left != nullptr) {
                q.push(p->left);
                if (p->right != nullptr) {
                    q.push(p->right);
                    q.pop();
                }
                else break;
            }
            else break;
        }
    }

    int insert(int v) {
        TreeNode* p = q.front();
        TreeNode* n = new TreeNode(v);
        if (p->left == nullptr) {
            p->left = n;
            q.push(n);
            return p->val;
        }
        p->right = n;
        q.push(n);
        q.pop();
        return p->val;
    }

    TreeNode* get_root() {
        return root;
    }
};
```
