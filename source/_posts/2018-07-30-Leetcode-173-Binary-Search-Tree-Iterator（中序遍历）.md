---
title: Leetcode 173. Binary Search Tree Iterator（中序遍历）
urlname: leetcode-173-binary-search-tree-iterator
toc: true
date: 2018-07-31 00:19:34
updated: 2018-07-31 00:53:00
tags: [Leetcode, alg:Tree, alg:Binary Search Tree, alg:In-Order Traversal]
---

题目来源：[https://leetcode.com/problems/binary-search-tree-iterator/description/](https://leetcode.com/problems/binary-search-tree-iterator/description/)

标记难度：Medium

提交次数：1/1

代码效率：98.53%

## 题意

写一个二叉搜索树的迭代器，包括初始化、`next()`和`hasNext()`操作，要求平摊时间复杂度为O(1)，空间复杂度为O(h)。

## 分析

因为空间复杂度是O(h)，所以肯定不能把整个树直接存成一个数组然后直接查找，而是要利用树的性质。好吧，其实《数据结构》里在迭代版中序遍历中讲了这个东西。

![中序遍历过程](in-order-traverse.png)

>与所有遍历一样，中序遍历的实质功能也可理解为，为所有节点赋予一个次序，从而将半线性的二叉树转化为线性结构。于是一旦指定了遍历策略，即可与向量和列表一样，在二叉树的节点之间定义前驱与后继关系。其中没有前驱（后继）的节点称作首（末）节点。
对于后面将要介绍的二叉搜索树，中序遍历的作用至关重要。相关算法必需的一项基本操作，就是定位任一节点在中序遍历序列中的直接后继。为此，可实现succ()接口如代码5.16所示。

```
// 代码5.16 二叉树节点直接后继的定位
template <typename T> BinNodePosi(T) BinNode<T>::succ() { //定位节点v的直接后继
    BinNodePosi(T) s = this; //记录后继的临时变量
    if (rChild) { //若有右孩子，则直接后继必在右子树中，具体地就是
        s = rChild; //右子树中
        while (HasLChild(*s)) s = s->lChild; //最靠左（最小）的节点
    } else { //否则，直接后继应是“将当前节点包含于其左子树中的最低祖先”，具体地就是
        while (IsRChild(*s)) s = s->parent; //逆向地沿右向分支，不断朝左上方移动
        s = s->parent; //最后再朝右上方移动一步，即抵达直接后继（如果存在）
    }
    return s;
}
```

>这里，共分两大类情况。若当前节点有右孩子，则其直接后继必然存在，且属于其右子树。此时只需转入右子树，再沿该子树的最左侧通路朝左下方深入，直到抵达子树中最靠左（最小）的节点。
反之，若当前节点没有右子树，则若其直接后继存在， 必为该节点的某一祖先，且是将当前节点纳入其左子树的最低祖先。于是首先沿右侧通路朝左上方上升，当不能继续前进时，再朝右上方移动一步即可。
作为后一情况的特例，出口时s可能为NULL。这意味着此前沿着右侧通路向上的回溯，抵达了树根。也就是说，当前节点是全树右侧通路的终点——它也是中序遍历的终点，没有后继。
（摘自《数据结构(C++语言版)》（第三版），清华大学出版社，2013.9）

我的代码实现得不怎么漂亮，不过里面被注释掉的关键的一句代码`path.push(_cur);`变相实现了“右侧通路朝左上方上升，当不能继续前进时，再朝右上方移动一步”这个操作：栈中实际上存储的就是经过这个操作之后能够得到的结点，因此右转时的结点是不能入栈的。

## 代码

### 我的代码
```
/**
 * Definition for binary tree
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class BSTIterator {
private:
    TreeNode* _root;
    TreeNode* _cur;
    stack<TreeNode*> path;

public:
    BSTIterator(TreeNode *root) {
        _root = root;
        _cur = root;
        // 找到树中最小的结点
        while (_cur != NULL && _cur->left != NULL) {
            path.push(_cur);
            _cur = _cur->left;
        }

        // cout << _cur->val << endl;
    }

    /** @return whether we have a next smallest number */
    bool hasNext() {
        // 和next()的判断逻辑相同：
        // 当前结点有右子树，或者仍然有向上回溯的空间
        return _cur != NULL;
    }

    /** @return the next smallest number */
    int next() {
        int smallest = _cur->val;
        // cout << smallest << ' ' << path.size() << endl;
        // 寻找_cur的后继
        // _cur的右子树不为空，此时后继必为_cur的右子树中最靠左的结点
        if (_cur->right != NULL) {
            // 必须注释掉这句话，因为path（这个名字起得不好）指的并不是从root到当前结点的路径上的所有结点
            // 而是“右子树仍未被访问到的结点”
            // path.push(_cur);
            _cur = _cur->right;
            while (_cur->left != NULL) {
                path.push(_cur);
                _cur = _cur->left;
            }
        }
        // _cur的右子树为空，此时需要向上回溯
        // 因为TreeNode的定义中没有提供父结点指针，所以只好用栈来记录了
        // 当然，按照邓公的意见，这样可以省空间，应该是最好的……
        else {
            if (path.size() > 0) {
                _cur = path.top();
                path.pop();
            }
            else
                _cur = NULL;
        }

        return smallest;
    }
};

/**
 * Your BSTIterator will be called like this:
 * BSTIterator i = BSTIterator(root);
 * while (i.hasNext()) cout << i.next();
 */
```

### Sample 20ms Submission

我刚发现在提交后的结果页面点时间会得到相应的标程，惊了，Leetcode真是越来越强了。

```
/**
 * Definition for binary tree
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class BSTIterator {
public:
    stack<TreeNode *> s;
    void addToStack(TreeNode * root){
        while(root){
            s.push(root);
            root=root->left;
        }
    }

    BSTIterator(TreeNode *root) {
        addToStack(root);
    }

    /** @return whether we have a next smallest number */
    bool hasNext() {
        return s.size()>0;
    }

    /** @return the next smallest number */
    int next() {
        int res = s.top()->val;
        TreeNode * top = s.top();
        s.pop();
        if(top-> right){
            addToStack(top->right);
        }

        return res;
    }
};

/**
 * Your BSTIterator will be called like this:
 * BSTIterator i = BSTIterator(root);
 * while (i.hasNext()) cout << i.next();
 */
```
