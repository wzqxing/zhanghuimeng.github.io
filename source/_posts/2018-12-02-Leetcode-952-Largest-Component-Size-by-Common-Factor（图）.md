---
title: Leetcode 952. Largest Component Size by Common Factor（图）
urlname: leetcode-952-largest-component-size-by-common-factor
toc: true
date: 2018-12-02 15:00:31
updated: 2018-12-02 16:37:00
tags: [Leetcode, Leetcode Contest, alg:Graph, alg:Union-find Forest, alg:Math]
---

题目来源：[https://leetcode.com/problems/largest-component-size-by-common-factor/description/](https://leetcode.com/problems/largest-component-size-by-common-factor/description/)

标记难度：Hard

提交次数：7/8

代码效率：

* 埃式筛法，map存储分解结果，普通并查集：1324ms
* 埃式筛法，map存储分解结果，size优化并查集：1452, 1848ms
* 欧拉筛法，map存储分解结果，普通并查集：TLE, 1868ms
* 埃式筛法（减少素数），map存储分解结果，普通并查集：144ms
* 素数打表，map存储分解结果，普通并查集：132ms
* 素数打表，手写链表，普通并查集：132ms

## 题意

给定一些结点，每个结点上有一个值，令两个结点有边相连当且仅当上面的值有大于1的公因子。问图中最大的连通分量的大小。

结点数量在`[1, 20000]`范围内，值的大小在`[1, 100000]`范围内。

## 分析

这道题有一个显然的做法：

* 首先打一个质数表。
  * 我之前认为需要打`[1, 100000]`范围内的质数（事实证明，一共有九千多个），但事实上不需要，只要打`[1, sqrt(100000)]`范围内的素数就可以了。在做质因数分解的时候，如果用上述范围内的质数没能约到1，则剩下的数必然是个大素数，不需要打表打到这个范围。
  * 打表可以用埃式筛法或者欧拉筛法（我之前在[某次模拟赛](/post/thuss-2016-postgraduate-entrance-exam-simulation-b-prime/)中做过类似的题）。
* 然后对每个数值作质因数分解，对每个质数开一个链表（或者类似的结构），如果一个质数是某个数的因数，就把这个数（的index）放到链表中。
* 然后对每条链表中的值在并查集中作合并操作。
* 最后找出并查集中最大的集合。

然后可以进行一些优化：

* 换成欧拉筛法（结果耗时反而变多了）
* 对并查集进行size优化（结果耗时反而变多了）
* 只打`sqrt(100000)`范围内的质数表（耗时骤降，变成约10%）
* 手工打质数表（耗时稍微减少）
* map换成手写链表（耗时居然没变）

想到算法的复杂度实际上是`O(NP)`，少遍历质数表好像的确能降低复杂度……

以及我在评论区里看到一个直接在做欧拉筛的时候进行合并的方法[^groawr]，非常简洁（但好像没有我快？看来打整张质数表还是太耗费时间了）。

[^groawr]: [groawr's Solution for Leetcode 952 - C++ Sieve of Erastosthenes](https://leetcode.com/problems/largest-component-size-by-common-factor/discuss/200613/C++-Sieve-of-Erastosthenes)

## 代码

这次提交了很多版本的代码，这里放两个最快的好了。

### map版本

132ms。

```cpp
class Solution {
private:
    // 并查集
    int _fa[20005];
    int _size[20005];
    int n;
    
    void init() {
        for (int i = 0; i < n; i++) {
            _fa[i] = i;
            _size[i] = 1;
        }
    }
    
    int fa(int x) {
        if (_fa[x] == x) return x;
        _size[_fa[x]] += _size[x];
        _size[x] = 0;
        return _fa[x] = fa(_fa[x]);
    }
    
    int merge(int x, int y) {
        x = fa(x);
        y = fa(y);
        if (_size[x] < _size[y]) swap(x, y);
        _fa[y] = x;
        return fa(y);
    }
    
    int size(int x) {
        x = fa(x);
        return _size[x];
    }
    
public:
    int largestComponentSize(vector<int>& A) {
        int primes[] = {2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53, 59, 61, 67, 71, 73, 79, 
                  83, 89, 97, 101, 103, 107, 109, 113, 127, 131, 137, 139, 149, 151, 157, 163, 167, 
                  173, 179, 181, 191, 193, 197, 199, 211, 223, 227, 229, 233, 239, 241, 251, 257, 263, 
                  269, 271, 277, 281, 283, 293, 307, 311, 313, 317};
        int m = sizeof(primes) / sizeof(int);
        // 质因数分解，插入对应的vector
        map<int, vector<int>> primeMap;
        n = A.size();
        for (int i = 0; i < n; i++) {
            int x = A[i];
            int j = 0;
            while (j < m) {
                if (x == 1) break;
                while (j < m && x % primes[j] != 0) j++;
                if (j >= m) break;
                while (x % primes[j] == 0) x /= primes[j];
                primeMap[primes[j]].push_back(i);
                j++;
            }
            if (x != 1) primeMap[x].push_back(i);
        }
        // 合并每个质因数对应的所有数
        init();
        for (auto const& p: primeMap) {
            if (p.second.size() > 1) {
                int fa0 = fa(p.second[0]);
                for (int i = 1; i < p.second.size(); i++) {
                    fa0 = merge(fa0, p.second[i]);
                }
            }
        }
        // 找到最大的集合，输出结果
        int maxn = -1;
        for (int i = 0; i < n; i++)
            maxn = max(maxn, size(i));
        return maxn;
    }
};
```

### 链表版本

也是132ms。

```cpp
class Solution {
private:
    // 并查集
    int _fa[20005];
    int _size[20005];
    int n;
    
    void init() {
        for (int i = 0; i < n; i++) {
            _fa[i] = i;
            _size[i] = 1;
        }
    }
    
    int fa(int x) {
        if (_fa[x] == x) return x;
        _size[_fa[x]] += _size[x];
        _size[x] = 0;
        return _fa[x] = fa(_fa[x]);
    }
    
    int merge(int x, int y) {
        x = fa(x);
        y = fa(y);
        if (_size[x] < _size[y]) swap(x, y);
        _fa[y] = x;
        return fa(y);
    }
    
    int size(int x) {
        x = fa(x);
        return _size[x];
    }
    
    // 链表
    struct Node {
        int val;
        Node* next;
        
        Node(int x) {
            val = x;
            next = NULL;
        }
    };
    
    Node* heads[100000];
    inline void insert(int prime, int i) {
        Node* node = new Node(i);
        node->next = heads[prime];
        heads[prime] = node;
    }
    
public:
    int largestComponentSize(vector<int>& A) {
        int primes[] = {2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53, 59, 61, 67, 71, 73, 79, 
                  83, 89, 97, 101, 103, 107, 109, 113, 127, 131, 137, 139, 149, 151, 157, 163, 167, 
                  173, 179, 181, 191, 193, 197, 199, 211, 223, 227, 229, 233, 239, 241, 251, 257, 263, 
                  269, 271, 277, 281, 283, 293, 307, 311, 313, 317};
        int m = sizeof(primes) / sizeof(int);
        // 质因数分解，插入对应链表
        n = A.size();
        memset(heads, 0, sizeof(heads));
        for (int i = 0; i < n; i++) {
            int x = A[i];
            int j = 0;
            while (j < m) {
                if (x == 1) break;
                while (j < m && x % primes[j] != 0) j++;
                if (j >= m) break;
                while (x % primes[j] == 0) x /= primes[j];
                insert(primes[j], i);
                j++;
            }
            if (x != 1) insert(x, i);
        }
        // 合并每个质因数对应的所有数
        init();
        for (int i = 2; i < 100000; i++) {
            if (heads[i] != NULL) {
                int fa0 = fa(heads[i]->val);
                Node* p = heads[i]->next;
                for (p != NULL; p != NULL; p = p->next) {
                    fa0 = merge(fa0, p->val);
                }
            }
        }
        // 找到最大的集合，输出结果
        int maxn = -1;
        for (int i = 0; i < n; i++)
            maxn = max(maxn, size(i));
        return maxn;
    }
};
```