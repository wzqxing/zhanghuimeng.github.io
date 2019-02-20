---
title: 重心剖分（Centroid Decomposition）简介及例题
urlname: centroid-decomposition-summaary-and-example
toc: true
date: 2019-02-16 01:07:47
updated: 2019-02-20 18:43:00
tags: [alg:Centroid Decomposition]
categories: Codeforces
---

这篇文章主要参考了[Centroid Decomposition](https://codeforces.com/blog/entry/52492)和[An illustrated introduction to centroid decomposition](https://medium.com/carpanese/an-illustrated-introduction-to-centroid-decomposition-8c1989d53308)两篇文章。

---

## 问题的描述

首先给出一道例题：[Codeforces 342E. Xenia and Tree](http://codeforces.com/problemset/problem/342/E)。这道题的题意是这样的：有一棵结点数量为`n`的树，结点标号为1到`n`。起始时，结点1是红色的，其他结点都是蓝色的。编程处理以下两种查询：

* `update(a)`：将`a`更新为红色
* `query(a)`：查询`a`离最近的红色点的距离

![两种颜色的结点](xenia.png)

可以立刻想到两种解法：

1. 查询时做BFS/DFS（`O(N)`），更新时直接更新（`O(1)`）
2. 维护每个结点到最近的红色点的距离，查询时直接查询（`O(1)`），更新时做BFS/DFS（`O(N)`）

显然这两种解法都不够好。重心剖分（Centroid Decomposition，以下简写为CD）则可以在这两种方法之间取得平衡，使得查询和更新的代价都变成`O(log(N))`。

### 重心的定义

记树的总结点数为`n`，定义树的重心为，移除后使得留下的所有连通分量（树）的大小均不超过`n/2`的结点。

![一个重心的例子](centroid-def.gif)

请注意：重心不是中心。

### 如何找到树的重心？

下面给出一种计算树的重心的算法。首先任取结点`a`，以`a`为树的根结点，计算它的所有子树的大小。如果这些子树的大小均不超过`n/2`，则`a`就是重心。否则，必然存在一棵（且只能有一棵——这一点是平凡的）子树，大小超过`n/2`。记`b`为该子树的根结点，对`b`重复上述算法。

![上述算法的执行过程](centroid-alg.gif)

对`b`来说，以`a`为根的子树的大小必然不超过`n/2`，因此算法不会重复访问已经访问过的结点，因此算法是正确的，它的复杂度是`O(n)`。

![为什么算法是正确的](centroid-alg-proof.png)

在具体实现中，以`a`为根结点，首先用DFS求出每棵子树的大小；然后用DFS寻找重心。因为算法不需要重复访问已经访问过的结点，因此对于每个结点，考虑它的子结点对应的子树大小即可。

```cpp
int subSize[N];
set<int> tree[N];

// 计算子树大小
int dfs(int u, int p) {
    subSize[u] = 1;
    for (int v: tree[u])
        if (v != p)
            subSize[u] += dfs(v, u);
    return subSize[u];
}

// 计算重心
int get_centroid(int u, int p, int n) {
    for (int v: tree[u])
        if (v != p && subSize[v] > n/2)
            return get_centroid(v, u, n);
    return u;
}
```

### 练习

练习1：请证明每棵树最多只有一个重心，或者给出反例。

反例：如下图。

![蓝色和红色结点各表示一个重心](one-centroid.png)

练习2：在什么样的树中，中心和重心是相同的？

我感觉，计算出重心之后，以重心为根的树中，如果深度最大的两棵子树的深度相等或只相差1，那么中心等于重心。

## 什么是重心剖分？

树的重心剖分是另一棵树，它递归定义为：

* 树根是原树的重心
* 树根的子结点是原树中移除重心后留下的子树的重心

![一棵树的重心剖分](cd.png)

### 实现

直接按照定义实现这棵树即可。

```cpp
int subSize[N];
set<int> tree[N];  // 为了方便删除……
int cd_father[N];

// 计算子树大小
int dfs(int u, int p) {
    subSize[u] = 1;
    for (int v: tree[u])
        if (v != p)
            subSize[u] += dfs(v, u);
    return subSize[u];
}

// 计算重心
int get_centroid(int u, int p, int n) {
    for (int v: tree[u])
        if (v != p && subSize[v] > n/2)
            return get_centroid(v, u, n);
    return u;
}

// 重心分解
void centroid_decomposition(int u, int p) {
    int n = dfs(u, p);
    int centroid = get_centroid(u, p, n);
    cd_father[centroid] = p;
    for (int v: tree[centroid])
        if (v != p) {
            tree[v].erase(centroid);
            centroid_decomposition(v, centroid);
        }
    tree[centroid].clear();
}
```

### 时间复杂度

建树的时间复杂度是多少？首先可以给出一个时间复杂度的上界：因为需要对每个结点都执行一次`centroid_decomposition`，且`dfs`的代价最多为`O(n)`，因此时间复杂度最多为`O(n^2)`。

不过事实上并没有那么多。对每个结点执行`centroid_decomposition`时，对应的连通分量大小已经大大减小了，所以`dfs`的代价也降低了——这是因为根据重心的性质，每个连通分量的大小最多为`n/2`。事实上这就类似于归并排序的分析：

![每一层的代价之和都是O(n)](complexity.png)

因为树的高度是`O(log(n))`，因此总时间复杂度为`O(n*log(n))`。

而且实现中移除边的过程不会影响总时间复杂度，因为最多有`O(n)`条边需要移除，而移除每条边的代价最多是`O(log(n))`。（当然，你也可以不这么实现，省一点常数。）

### 重心剖分的性质

1. 在CD树中，结点属于它的所有祖先对应的连通分量。

![结点14属于14、15、11和3对应的连通分量](centroid-property.png)

证明：在CD树中，结点`a`是`b`的子结点，仅当`a`属于移除`b`后产生的连通分量。显然这件事的前提是，`a`属于`b`对应的连通分量。（这听起来是平凡的。）

2. 原树中结点`a`到结点`b`的最短路可以分解成结点`a`到`lca(a, b)`和`lca(a, b)`到`b`的两条路径，其中`lca(a, b)`是CD树中`a`和`b`的LCA。

![原树中从9到10的最短路可以分解成从9到3的路径和从3到10的路径。](cd-property-2.png)

证明：由性质1，`a`和`b`都属于`lca(a, b)`对应的连通分量。假定`lca(a, b)`并不在从`a`到`b`的最短路上，则在原树中移除`lca(a, b)`后，`a`和`b`仍然在同一个连通分量中，这意味着该连通分量的重心是`a`和`b`的比`lca(a, b)`更低的共同祖先，这显然是荒谬的。

3. 原树中的`n^2`条路径（此处把退化的路径也算进去了）均可分解成两条路径，这两条路径都属于在CD树中每个结点到它的所有祖先结点的共`O(n*log(n))`条路径的集合。（听起来真是晦涩……）

这个性质比较难，但非常重要。以下图为例：

![结点14属于14、15、11和3对应的连通分量](centroid-property.png)

共有`n`条从结点14开始的路径。这些路径可以分成以下几类：

1. `a in {14}`：从14到14，再从14到`a`
2. `a in {15}`：从14到15，再从15到`a`
3. `a in {6, 9, 13}`：从14到11，再从11到`a`
4. `a in {1, 2, 4, 5, 7, 8, 10, 12}`：从14到3，再从3到`a`

显然14、15、11和3都是14在CD树中的祖先。这种分类方法的思路是这样的：不是选择路径的两个端点，而是选择两个结点在CD树中的LCA。

证明1：性质2说明，原树中的每条路径都可以分解成两条路径（`a`到`lca(a, b)`，以及`lca(a, b)`到`b`）。下面证明从每个结点到它的CD树中祖先结点的路径总数是`O(n*log(n))`。显然CD树的高度是`O(log(n))`，共有`n`个结点，所以祖先总数是`O(n*log(n))`。

证明2：这次考虑CD树中每个结点的后代数量。显然根结点的后代总数是`n-1`，而且每一层的结点的后代总数都是`O(n)`，因此总路径数为`O(n*log(n))`。

### 练习

练习3：给定下图中的CD树，求原树。是否有多个可能的答案？

![CD树](ex3-tree.png)

这棵树看起来好像很有问题，居然有重复结点，算了不管它了。。。不过显然CD树和原树不是一一对应的，举个最简单的例子，如果连通分量只剩下两个结点，那么这两个结点哪一个做重心都可以。

练习4：证明每棵CD树都是自己的CD树，或者举出反例。

证明：由CD树的构造过程可知，对于CD树的每棵子树，记其大小为`n`，去掉根结点后剩下的的每棵子树的大小均不超过`n/2`。这是因为每棵子树都是和一个联通分量对应的。因此，对CD树做重心剖分时，只需取每层的根结点为重心即可。

练习5：考虑以下陈述：“对于任意有根树，从`a`到`b`的路径都可以分解成从`a`到`lca(a, b)`的路径和从`lca(a, b)`到`b`的路径，这样我们就可以应用性质3中的方法进行处理。”如果这是真的，我们为什么需要重心剖分？

答：这确实是真的，但对于高度没有限制的树，这么做没有意义。考虑退化成一条链的树，根结点的后代数量是`n-1`，深度为1的结点的后代数量是`n-2`，以此类推，得到的分解路径总数是`n(n-1)/2`，和树中所有路径总数的数量级相同，没法起到简化表示的作用。

## 例题

### [Codeforces 342E. Xenia and Tree](https://codeforces.com/contest/342/problem/E)

#### 题意

略

#### 分析

这道题就是上面讲解时用到的例题，应该很好理解。将树进行重心剖分之后，每两个结点之间的距离都可以分解成它们在原树中到重心剖分树中的LCA的距离。这句话听起来太绕了，不如说，对于每两个结点，它们在重心剖分树中的LCA必然会出现在它们在原树中的最短路径上。从这就可以直接推导出，原树中的每条路径都能以两个端点在重心剖分树中的LCA为终点分解成两条路径。

所以我们可以考虑用`ans`来维护重心剖分树中每个结点到它的子树中最近的红色结点的距离。初始时，`ans[a] = inf`（之后才将第一个结点涂成红色）。

![左侧是染色的原树，右侧是重心剖分和连通分量。](xenia-cd.png)

对于每个`update(a)`操作，因为`a`出现在它的祖先结点对应的分量中，所以只需对它的每个祖先结点`b`，更新`ans[b] = min(ans[b], dist(a, b))`。由于树的高度为`O(log(n))`，计算`dist(a, b)`的复杂度是`O(log(n))`，因此更新操作的复杂度是`O(log^2(n))`。

对于每个`query(a)`操作，只需对它的所有祖先结点`b`，取`dist(a, b) + ans[b]`的最小值。如果记`ans[b]`对应的结点为`c`，则我们实际上是把从`a`到`c`的路径分解成了从`a`到`b`的路径（`dist(a, b)`）和从`b`到`c`的路径（`ans[b]`）。这意味着`dist(a, b) + ans[b]`是从`a`到`b`对应的连通分量中离`b`最近的红色结点的距离。查询操作的时间复杂度也是`O(log^2(n))`。

---

我花了特别久的时间debug。模板背错之后出现的那些问题就不说了——一背错就很可能会死循环。首先，照原文中那种删除边的写法是行不通的：

```cpp
void build(int u, int p) {
    int n = dfs(u, p); // find the size of each subtree
    int centroid = dfs(u, p, n); // find the centroid
    if (p == -1) p = centroid; // dad of root is the root itself
    dad[centroid] = p;

    // for each tree resulting from the removal of the centroid
    for (auto v : tree[centroid])
        // v被删除后，指针就失效了，肯定会挂
        tree[centroid].erase(v), // remove the edge to disconnect
        tree[v].erase(centroid), // the component from the tree
        build(v, centroid);
}
```

改成不会产生指针失效的版本也不行，会超时。我目前看到了两种比较好的解决方案：

* 仍然用`set`，但是在`for`循环中不从`centroid`对应的`set`删除，在`for`循环结束后再统一清空
* 改用`vector`，单独记录删除标记

然后我想了想，觉得自己的LCA写的恐怕大有问题（因为太久没写了），于是就找了个地方，抄了一下LCA的主要计算过程。这之后我觉得没有什么问题了，但是交上去却持续WA。我感到很困惑，查了又查，却找不到什么错误。最后我找到了一份相当不错的结构和我类似的[参考代码](https://codeforces.com/contest/342/submission/23224047)，抱着“模块化debug”的心情把我的代码中的LCA整个换成了这份代码里的LCA——

结果竟然就过了！！！

原来我太久没写LCA，把初始化时求`2^k`级祖先的内外循环给搞反了。推导`father[i][j]`时可能需要的`father[?][j-1]`的第一维是不确定的，因此应该把`j`放在外层循环。

```cpp
void lca_init() {
    for (int j = 1; j < 21; j++) {
        for (int i = 1; i <= n; i++)
            father[i][j] = father[father[i][j-1]][j-1];
    }
}
```

#### 代码

```cpp
#include <iostream>
#include <cstdio>
#include <vector>
#include <set>
using namespace std;
int n, m;
typedef long long LL;

vector<int> G[100002];
bool deleted[100002];
int subTreeSize[100002];
int cd_father[100002];
int father[100002][21];
LL dist[100002];
LL ans[100002];

// 计算父结点和结点深度（用于LCA）
void dfs(int cur, int parent, int depth) {
    father[cur][0] = parent == -1 ? cur : parent;
    dist[cur] = depth;
    for (int u: G[cur])
        if (u != parent) {
            dfs(u, cur, depth + 1);
        }
}

// LCA初始化
void lca_init() {
    for (int j = 1; j < 21; j++) {
        for (int i = 1; i <= n; i++)
            father[i][j] = father[father[i][j-1]][j-1];
    }
}

// 计算x和y的LCA
int get_lca(int x, int y) {
    if (dist[x] < dist[y]) swap(x, y);
    int d = dist[x] - dist[y];
    for (int i = 20; i >= 0; i--) {
        if (d & (1 << i))
            x = father[x][i];
    }
    if (x == y) return x;
    for (int i = 20; i >= 0; i--) {
        if (father[x][i] != father[y][i]) {
            x = father[x][i];
            y = father[y][i];
        }
    }
    return father[x][0];
}

// 根据LCA和深度计算x和y在树中的距离
LL get_dist(int x, int y) {
    int fa = get_lca(x, y);
    return dist[x] + dist[y] - 2 * dist[fa];
}

// 计算子树大小（每次重心剖分的子树都需要）
void get_size(int cur, int parent) {
    subTreeSize[cur] = 1;
    for (int u: G[cur]) {
        if (!deleted[u] && u != parent) {
            get_size(u, cur);
            subTreeSize[cur] += subTreeSize[u];
        }
    }
}

// 计算重心
int get_centroid(int cur, int parent, int n) {
    for (int u: G[cur]) {
        if (!deleted[u] && u != parent && subTreeSize[u] > n / 2)
            return get_centroid(u, cur, n);
    }
    return cur;
}

// 递归进行重心剖分
void centroid_decomposition(int cur, int parent) {
    get_size(cur, parent);
    int centroid = get_centroid(cur, parent, subTreeSize[cur]);
    cd_father[centroid] = parent;
    // 这里采取的则是一种比较愚蠢的策略，单独为结点记录了删除标记……
    deleted[centroid] = true;
    for (int u: G[centroid])
        if (!deleted[u] && u != parent)
            centroid_decomposition(u, centroid);
}

// 将a更新为红色结点
void update(int a) {
    int b = a;
    // 对于a在CD树中的每个祖先（包括a），更新a到它的距离
    // 注意不是a在CD树中到它的距离！！！
    while (b != -1) {
        ans[b] = min(ans[b], get_dist(a, b));
        b = cd_father[b];
    }
}

// 查询a和最近的红色结点之间的距离
int query(int a) {
    int b = a;
    LL x = 1e9;
    // 对于a在CD树中的每个祖先（包括a），取答案为（a到该祖先的距离+该祖先到最近红色结点距离）的最小值
    // 注意距离不是CD树中的距离！！！
    while (b != -1) {
        x = min(x, ans[b] + get_dist(a, b));
        b = cd_father[b];
    }
    return x;
}

int main() {
    scanf("%d %d", &n, &m);
    int a, b;
    int t, v;
    for (int i = 0; i < n - 1; i++) {
        scanf("%d %d", &a, &b);
        G[a].push_back(b);
        G[b].push_back(a);
    }
    // 一些必要的初始化
    dfs(1, -1, 0);
    lca_init();
    centroid_decomposition(1, -1);

    for (int i = 1; i <= n; i++)
        ans[i] = 1e9;
    update(1);
    for (int i = 0; i < m; i++) {
        scanf("%d %d", &t, &v);
        if (t == 1) update(v);
        else printf("%d\n", query(v));
    }
    return 0;
}
```

### [Codeforces 321C. Ciel the Commander](https://codeforces.com/contest/321/problem/C)

#### 题意

给定一棵树，要求把上面的所有结点用`A`到`Z`标记，使得对于任意两个标记相同的结点，它们之间的最短路上至少有一个标记（字典序）更小的结点。

#### 分析

只要会重心剖分，想到这道题要用到重心剖分并不难（……这好像是句废话，考虑到这道题是作为例题出现的……），所以难点在于如何证明重心剖分得到的是最优解。（对于实现而言，这并不是个难点）

[题解](https://codeforces.com/blog/entry/8192)里给出了两种证明思路。第一种是自顶向下构造。显然，rank为A的结点只能有一个。选定一个结点为rank A之后，树会被分成几个连通分量，这些连通分量之间不会有非法路径（因为必须要通过这个结点），所以可以单独考虑这些连通分量，于是我们得到了一个递归解法。问题是应该怎么选择A。单从连通分量的大小来考虑，我们希望这些连通分量的大小尽量小，所以不妨取rank A结点为重心。然后判断CD树的高度是否大于26就行。

不过，问题是我觉得不能单从连通分量的大小来考虑。但到底怎么考虑比较好，这个我也不知道……

#### 代码

```cpp
#include <iostream>
#include <set>
using namespace std;
set<int> g[100005];
int cd_father[100005];
int subSize[100005];
int cd_depth[100005];
int n;
int maxDepth;

int get_size(int cur, int p) {
    subSize[cur] = 1;
    for (int u: g[cur])
        if (u != p)
            subSize[cur] += get_size(u, cur);
    return subSize[cur];
}

int get_centroid(int cur, int p, int n) {
    for (int u: g[cur])
        if (u != p && subSize[u] > n / 2)
            return get_centroid(u, cur, n);
    return cur;
}

void centroid_decomposition(int cur, int p, int depth) {
    int n = get_size(cur, p);
    int centroid = get_centroid(cur, p, n);
    cd_father[centroid] = p;
    cd_depth[centroid] = depth;
    maxDepth = max(depth, maxDepth);
    for (int u: g[centroid])
        if (u != p) {
            g[u].erase(centroid);
            centroid_decomposition(u, centroid, depth + 1);
        }
    g[centroid].clear();
}

int main() {
    cin >> n;
    int u, v;
    for (int i = 1; i < n; i++) {
        cin >> u >> v;
        g[u].insert(v);
        g[v].insert(u);
    }
    centroid_decomposition(1, -1, 0);
    if (maxDepth >= 26) {
        cout << "Impossible!" << endl;
        return 0;
    }
    for (int i = 1; i <= n; i++)
        cout << (char) (cd_depth[i] + 'A') << ' ';
    cout << endl;
    return 0;
}
```
