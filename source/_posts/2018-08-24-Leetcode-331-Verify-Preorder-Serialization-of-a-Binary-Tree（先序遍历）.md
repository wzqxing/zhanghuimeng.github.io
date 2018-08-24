---
title: Leetcode 331. Verify Preorder Serialization of a Binary Tree（先序遍历）
urlname: leetcode-331-verify-preorder-serialization-of-a-binary-tree
toc: true
date: 2018-08-24 15:30:05
updated: 2018-08-24 17:32:05
tags: [Leetcode]
---

题目来源：[https://leetcode.com/problems/verify-preorder-serialization-of-a-binary-tree/description/](https://leetcode.com/problems/verify-preorder-serialization-of-a-binary-tree/description/)

标记难度：Medium

提交次数：1/2

代码效率：

* 冗长的做法：73.99%
* 考虑有向图的度：73.64%
* 递归删除子树：73.64%

## 题意

给定一个二叉树的先序遍历序列（但是其中列出了空结点），要求判断这个序列是否是合法的。

## 分析

我看到题目中说“不要把树重建出来”之后，随便脑补了一个算法：实际上是类似于重建的做法，用栈取代了递归建树的过程，没有明确记录每个结点的左孩子和右孩子都是什么，只记录了是否存在。中间WA了一次是因为系统要求把一个空结点也判定为合法的先序遍历序列。

做完之后我觉得这是一道非常有趣的题。我们都知道只用先序遍历是无法重建一棵树的，因为几乎完全无法确定树的形状；但把中间遍历过的空结点记录下来之后（这个说法不是很准确；我觉得准确的说法是，对于每个结点都无条件地访问它的左孩子和右孩子，如果访问到的结点为空，仍然按顺序记录到遍历序列中，但不会再去访问它的孩子，显然也没有），我们就可以把它重建出来了。我认为这些多出来的信息（空结点的位置）能够帮助我们确定树的形状，所以空结点实际上提供的信息量是很大的，这一点很有趣。

或者也可以采取另一种看法：在这里空结点就是叶结点，相当于在树上新加了一层叶子；我们仍然做的是一次普通的先序遍历，但是在其中标记了哪些是叶结点。依靠这些叶结点作为定位信息，我们可以把树重建出来。

不过，如果把整棵树都真的重建出来，感觉像是过度利用已有的信息了，如果只需要验证这个序列的合法性，评论区给出了很多很妙的做法。

### 考虑有向图的度

[7 lines Easy Java Solution](https://leetcode.com/problems/verify-preorder-serialization-of-a-binary-tree/discuss/78551/7-lines-Easy-Java-Solution)中提供了一种非常有趣的做法：把二叉树看成一个有向图（当然，树本来就是有向图，但我平时很少这么看……），然后计算图中的总入度和总出度是否相等，且要求计算过程中总出度始终>=总入度；如果满足要求，则原序列为合法的先序遍历序列。

计算总入度和总出度的方法是比较简单的：

* 根结点提供两个出度
* 每个除根结点以外的非空结点提供两个出度，一个入度
* 每个空结点提供一个入度

上述做法的必要性是显然的，对于一棵合法的树，它的总出度和总入度必然是相等的。至于充分性，我觉得可以考虑不合法的几种情况。一个先序遍历序列不合法，只可能是因为：

* 有的结点缺少左孩子或右孩子：出度+1
* 有的结点缺少父节点（除根结点外）：入度+1

但是，单纯这样考虑就会遇到问题：如果恰好有一个结点缺少一个孩子，另一个结点缺少一个父亲，那出度和入度就正好“中和”了。所以我们还需要“总出度始终>=总入度”这个条件。我们在按顺序检查遍历序列的每个结点时，由先序遍历的性质，必然是先检查到父结点，再检查到子节点，所以每检查一个非根节点时，必须有足够的出度，才能保证它有父结点，否则树仍然是不合法的。

所以事实上这个条件可以缩得更紧：在处理完最后一个结点之前，要求总出度始终>总入度。

[The simplest python solution with explanation (no stack, no recursion)](https://leetcode.com/problems/verify-preorder-serialization-of-a-binary-tree/discuss/78564/The-simplest-python-solution-with-explanation-%28no-stack-no-recursion%29)中的做法本质上是相同的，但作者解释得更好。

### 递归删除子树

[Java intuitive 22ms solution with stack](https://leetcode.com/problems/verify-preorder-serialization-of-a-binary-tree/discuss/78566/Java-intuitive-22ms-solution-with-stack)中提供了一种看起来很简单的方法：

* 遇到非空结点则入栈
* 遇到空结点时，检查栈顶是否为空结点
  * 如果栈为空，返回错误
  * 如果栈顶为非空结点，则将空结点入栈
  * 如果栈顶为空结点，则将空结点弹出，并将栈顶（此时必为非空结点）也弹出，然后连续处理，直到栈顶不是空结点为止
* 验证最后栈中只剩一个空结点

这种方法本质上就是不断把子树替换成空结点。

我用一种比较笨的方法实现了替换，但实际上连续弹出结点的时候必然是先弹出一个空结点，再弹出一个非空结点，再弹出一个空结点……如此交替，且最后栈不能为空。

## 代码

### 冗长的做法

```cpp
class Solution {
private:
    // https://stackoverflow.com/questions/236129/the-most-elegant-way-to-iterate-the-words-of-a-string
    std::vector<std::string> split(const std::string &text, char sep) {
        std::vector<std::string> tokens;
        std::size_t start = 0, end = 0;
        while ((end = text.find(sep, start)) != std::string::npos) {
            tokens.push_back(text.substr(start, end - start));
            start = end + 1;
        }
        tokens.push_back(text.substr(start));
        return tokens;
    }

    struct Node {
        bool left;
        bool right;

        Node() {
            left = right = false;
        }
    };

public:
    bool isValidSerialization(string preorder) {
        // 可以按照堆排序的思路，顺序计算每一个结点是否是valid的
        // 不对，这个是前序遍历，不是层次遍历……
        // 又遇到这个split的问题了==

        // 这是什么鬼特判
        if (preorder == "#")
            return true;

        vector<string> tokens(split(preorder, ','));

        stack<Node> tree;
        if (tokens.size() < 1 || tokens[0] == "#")
            return false;
        tree.push(Node());
        for (int i = 1; i < tokens.size(); i++) {
            if (tree.empty())
                return false;
            if (tree.top().left && tree.top().right)
                return false;
            if (!tree.top().left)
                tree.top().left = true;
            else
                tree.top().right = true;
            if (tokens[i] == "#") {
                while (!tree.empty() && tree.top().left && tree.top().right)
                    tree.pop();
            }
            else {
                tree.push(Node());
            }
        }

        if (!tree.empty())
            return false;

        return true;
    }
};
```

### 考虑有向图的度

```cpp
class Solution {
private:
    // https://stackoverflow.com/questions/236129/the-most-elegant-way-to-iterate-the-words-of-a-string
    std::vector<std::string> split(const std::string &text, char sep) {
        std::vector<std::string> tokens;
        std::size_t start = 0, end = 0;
        while ((end = text.find(sep, start)) != std::string::npos) {
            tokens.push_back(text.substr(start, end - start));
            start = end + 1;
        }
        tokens.push_back(text.substr(start));
        return tokens;
    }

public:
    bool isValidSerialization(string preorder) {
        if (preorder == "#")
            return true;

        vector<string> tokens(split(preorder, ','));

        int inDegree = 0, outDegree = 0;
        for (int i = 0; i < tokens.size(); i++) {
            if (i == 0) {
                if (tokens[i] == "#")
                    return false;
                outDegree += 2;
            }
            else if (tokens[i] == "#") {
                inDegree++;
            }
            else {
                inDegree++;
                outDegree += 2;
            }
            if (outDegree < inDegree || (outDegree == inDegree && i != tokens.size() - 1))
                return false;
        }

        return outDegree == inDegree;
    }
};
```

### 递归删除子树

```cpp
class Solution {
private:
    // https://stackoverflow.com/questions/236129/the-most-elegant-way-to-iterate-the-words-of-a-string
    std::vector<std::string> split(const std::string &text, char sep) {
        std::vector<std::string> tokens;
        std::size_t start = 0, end = 0;
        while ((end = text.find(sep, start)) != std::string::npos) {
            tokens.push_back(text.substr(start, end - start));
            start = end + 1;
        }
        tokens.push_back(text.substr(start));
        return tokens;
    }

public:
    bool isValidSerialization(string preorder) {
        vector<string> tokens(split(preorder, ','));

        stack<string> tree;
        for (int i = 0; i < tokens.size(); i++) {
            string node = tokens[i];
            if (i == 0) {
                tree.push(node);
                continue;
            }
            tree.push(node);
            while (tree.size() >= 2) {
                string n1 = tree.top();
                tree.pop();
                string n2 = tree.top();
                tree.pop();
                if (n1 == "#" && n2 == "#") {
                    if (tree.empty())
                        return false;
                    tree.pop();
                    tree.push("#");
                }
                else {
                    tree.push(n2);
                    tree.push(n1);
                    break;
                }
            }
        }

        return !tree.empty() && tree.size() == 1 && tree.top() == "#";
    }
};
```
