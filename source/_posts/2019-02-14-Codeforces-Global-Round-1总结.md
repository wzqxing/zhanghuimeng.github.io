---
title: Codeforces Global Round 1总结
urlname: codeforces-global-round-1
toc: true
date: 2019-02-14 14:38:41
updated: 2019-02-14 14:38:41
tags: [Codeforces]
categories: Codeforces
---

因为写总结和做题实在太艰难了，我决定以后CF每场比赛只写一篇总结……

[^sln]: [The Editorial of the First Codeforces Global Round](https://codeforces.com/blog/entry/65079)

## 1110A

题目来源：[https://codeforces.com/contest/1110/problem/A](https://codeforces.com/contest/1110/problem/A)

提交次数：1/1

### 题意

### 分析

### 代码

## 1110D

题目来源：[https://codeforces.com/contest/1110/problem/D](https://codeforces.com/contest/1110/problem/D)

提交次数：1/1

### 题意

给定若干个1和m之间的整数，定义triplet为形如`(x, x, x)`或`(x, x+1, x+2)`的元组，问这些整数最多一共能组成多少个triplet？

### 分析

这道题有一个非常必要的观察：如果有大于等于三个的形如`[x, x+1, x+2]`的triplet，那么完全可以把其中的三个替换成`[x, x, x]`，`[x+1, x+1, x+1]`和`[x+2, x+2, x+2]`。这也就意味着对于每一个`x`，形如`[x-2, x-1, x]`的triplet最多只有2个。[^sln]

然后我就想了一种错漏百出的方法，调了一个下午，终于调出来了。

令`f[i][x][y]`表示考虑前`i-1`个数组成的triplet，`a[i-1]`还剩`x`个，`a[i-2]`还剩`y`个时的triplet总数。我心想：既然连续的triplet最多只有2个，那么`x`和`y`的上限就都是2。（后来事实证明这很离谱）于是列出向后的转移方程：

```cpp
// 不组成形如(i-2, i-1, i)的triplet
if (a[i] >= 0)
    f[i+1][(a[i] - 0) % 3][x - 0] =
        max(f[i+1][(a[i]- 0) % 3][x - 0], f[i][x][y] + 0 + (a[i] - 0) / 3);
// 组成一个形如(i-2, i-1, i)的triplet
if (a[i] >= 1 && x >= 1 && y >= 1)
    f[i+1][(a[i] - 1) % 3][x - 1] =
        max(f[i+1][(a[i] - 1) % 3][x - 1], f[i][x][y] + 1 + (a[i] - 1) / 3);
// 组成两个形如(i-2, i-1, i)的triplet
if (a[i] >= 2 && x >= 2 && y >= 2)
    f[i+1][(a[i] - 2) % 3][x - 2] =
        max(f[i+1][(a[i] - 2) % 3][x - 2], f[i][x][y] + 2 + (a[i] - 2) / 3);
```

跑一下看看，发现错得离谱。思考了一段时间之后，发觉这个转移方程“太准确了”。比如说，`a[4]=2`时，`f[4][2][2]`可以转移到`f[5][1][1]`，但也应该可以转移到`f[5][0][1]`和`f[5][1][0]`（丢掉一些4和3也是可以的）。于是把转移方程修改如下，加入了后面两维的更多可能性：

```cpp
if (a[i] >= 0) {
    for (int k = 0; k <= min(a[i], 2); k++) {
        for (int j = 0; j <= x - 0; j++) {
            f[i+1][k][j] = 
                max(f[i+1][k][j], f[i][x][y] + 0 + (a[i] - k) / 3);
        }
    }
}
if (a[i] >= 1 && x >= 1 && y >= 1) {
    for (int k = 0; k <= min(a[i] - 1, 2); k++) {
        for (int j = 0; j <= x - 1; j++) {
            f[i+1][k][j] = 
                max(f[i+1][k][j], f[i][x][y] + 1 + (a[i] - k - 1) / 3);
        }
    }
}
if (a[i] >= 2 && x >= 2 && y >= 2) {
    for (int k = 0; k <= min(a[i] - 2, 2); k++) {
        f[i+1][k][x - 2] = 
            max(f[i+1][k][x - 2], f[i][x][y] + 2 + (a[i] - k - 2) / 3);
    }
}
```

结果还是不对。经过更加漫长的debug，我意识到这种做法里的上限不能是2，因为它表示的是整体剩下的上限，而不是一共有多少个以它结尾的triplet。于是我直接把上限改成了6（既然有三种triplet的可能性），然后就过了（虽然耗时非常长）。

### 代码

```cpp
#include <iostream>
#include <cstring>
using namespace std;
int n, m;
int cnt[1000005];
int f[1000005][7][7];
int main() {
    cin >> n >> m;
    int *a = cnt + 1;  // 处理i-2的边缘情况
    for (int i = 0; i < n; i++) {
        int x;
        cin >> x;
        a[x]++;
    }
    int ans = 0;
    for (int i = 1; i <= m + 1; i++) {
        for (int x = 0; x <= min(6, a[i-1]); x++)
            for (int y = 0; y <= min(6, a[i-2]); y++) {
                if (a[i] >= 0) {
                    for (int k = 0; k <= min(a[i], 6); k++) {
                        for (int j = 0; j <= x - 0; j++) {
                            f[i+1][k][j] = 
                                max(f[i+1][k][j], f[i][x][y] + 0 + (a[i] - k) / 3);
                        }
                    }
                }
                if (a[i] >= 1 && x >= 1 && y >= 1) {
                    for (int k = 0; k <= min(a[i] - 1, 6); k++) {
                        for (int j = 0; j <= x - 1; j++) {
                            f[i+1][k][j] = 
                                max(f[i+1][k][j], f[i][x][y] + 1 + (a[i] - k - 1) / 3);
                        }
                    }
                }
                if (a[i] >= 2 && x >= 2 && y >= 2) {
                    for (int k = 0; k <= min(a[i] - 2, 6); k++) {
                        f[i+1][k][x - 2] = 
                            max(f[i+1][k][x - 2], f[i][x][y] + 2 + (a[i] - k - 2) / 3);
                    }
                }
            }
    }
    for (int x = 0; x <= 6; x++)
        for (int y = 0; y <= 6; y++)
            ans = max(ans, f[m + 2][x][y]);
    cout << ans << endl;
    return 0;
}
```

## 1110F

题目来源：[https://codeforces.com/contest/1110/problem/F](https://codeforces.com/contest/1110/problem/F)

提交次数：1/1

### 题意

给定一棵带权树，所有结点按DFS遍历顺序从1到n编号，回答q次询问：给定整数v、l和r，找到从结点v到编号在l和r之间的叶结点之间的最短距离。

### 分析

题解里给出了一种很类似于可持久化线段树的离线方法：对根结点记录它到每个叶子的距离，然后从根结点走到需要查询的结点v，同时根据边权更新它到叶子的距离。不过并不是很详细。[^sln]

另一种方法是使用重心剖分（centroid decomposition，我就简称CD了）。首先对树进行重心剖分。对于每个叶结点，将它存储在它在重心剖分树的每个祖先结点中，也就是对重心剖分树中的每个结点，维护它子树中的叶结点的一个list，包括编号和（到该结点的）距离。需要回答询问时，对于结点v在重心剖分树中的每个祖先结点，查询该结点对应的叶结点中，编号在l和r之间的叶结点中的最短距离。这一步可以用线段树。[^cd]

不妨举个例子。这是CF上的第五个测试数据：

```
10 10
1 12
2 89
3 20
3 37
3 15
2 43
7 8
8 31
1 52
8 8 9
1 1 8
7 7 10
4 3 4
9 3 8
7 7 9
6 7 10
2 3 7
6 8 10
7 1 4
```

![左侧是原树，右侧是重心剖分树，以及子树中叶结点到当前结点的距离](tree.jpg)

[^cd]: [Codeforces - comment of gaurav172](https://codeforces.com/blog/entry/65059?#comment-490727)

真的去写的时候，照例遇到了一万个问题，不过幸好这次有一份写得相当不错的代码[^tfg-code]可以参照，省了很多时间。重心剖分的模板每次都背错这种愚蠢的事情就不说了，不过仍然会遇到如何组织树的这个问题。之前已经说过了，其实用修正过写法的`set`和`vector`加上删除标志都可以接受；但这次需要存边权，换成`map`听起来有点不太像一棵树。（倒不如说是我觉得这样遍历太麻烦了）所以换成了`vector<pair<int, long long>>`。

除了重心剖分以外，为了计算距离，当然LCA也是需要的。这次我虽然基本没有背错LCA模板，但我忘记这是棵带权树了。当然改起来很容易，把深度改成到根结点的距离就行。

初始化的部分倒是挺好写的——如果某个结点是叶子，那么就把它加到它在CD树的所有祖先（包括自己）的叶结点list中，同时计算距离。在这一步（或者不如说是下一步）中，我没有意识到这个list可能是空的，因为CD树的叶结点不一定是原树中的叶结点，所以仍然花了一些debug的时间。

所以下面得给每个结点都建一棵线段树。还采用自顶向下的那种方法就实在太冗长了，于是我直接抄了[^tfg-code]中的[zkw_cf线段树](https://codeforces.com/blog/entry/18051)，这个东西是对zkw线段树的改进，不仅是自底向上的，而且只需要使用`2*n`的空间，不需要和2的幂对齐了。不过我还没太搞明白这是怎么做到的，就直接抄了。。。

下面这个问题花了我很久去debug，听起来十分愚蠢，但事实就是这样的……CD树上每个叶结点的线段树都是做了离散化的，只包含有的叶结点的编号。所以需要查的时候，显然需要找到编号的index。所以下面的问题是：对于排好序的`pair<int, LL> a[n]`和`l <= r`，如何找到最左边的满足`a[i].first >= l`的`i`和最右边的满足`a[i].first <= r`的`i`？答案当然是二分查找，但是过程相当tricky……总之我最后也是又抄代码了。

最后一个问题很傻逼。我又忘记在CD树中从下向上查时，两个结点的距离不能用它们到中间结点的距离的和来推导了，这只能重新直接在原树中算……

[^tfg-code]: [tfg's solution for 1110F](https://codeforces.com/contest/1110/submission/49592451)

总的来说这算法比较慢。我现在懒得去分析复杂度了……

### 代码

```cpp
#include <iostream>
#include <algorithm>
#include <cstdio>
#include <set>
#include <vector>
#include <map>
using namespace std;
typedef long long LL;
int q, n;
vector<pair<int, LL>> tree[500005];
bool deleted[500005];
int cd_father[500005];
int father[500005][30];
LL dist[500005];
int level[500005];
int subSize[500005];

// 抄来的zkw_cf线段树
struct TreeNode {
    vector<pair<int, LL>> leaves;
    vector<LL> tree;
    int n;

    void init() {
        n = leaves.size();
        sort(leaves.begin(), leaves.end());
        tree.resize(2 * n);
        for (int i = 0; i < n; i++)
            tree[n + i] = leaves[i].second;
        for (int i = n - 1; i > 0; i--)
            tree[i] = min(tree[2 * i], tree[2 * i + 1]);
    }

    LL query(int l, int r, bool convert = false) {

        if (leaves.size() == 0 || r < leaves.front().first || leaves.back().first < l) return 1e16;
        if (convert) {
            // 抄的代码（注意r是开区间，这是这种线段树写法的要求）
            l = lower_bound(leaves.begin(), leaves.end(), make_pair(l, (LL) -1)) - leaves.begin();
            r = lower_bound(leaves.begin(), leaves.end(), make_pair(r + 1, (LL) -1)) - leaves.begin();
        }
        LL ans = 1e16;
        for (l += n, r += n; l < r; l /= 2, r /= 2) {
            if (l & 1) ans = min(ans, tree[l++]);
            if (r & 1) ans = min(ans, tree[--r]);
        }
        return ans;
    }

} segTree[500005];

void dfs(int cur, int parent, LL d, int l) {
    dist[cur] = d;
    level[cur] = l;
    father[cur][0] = parent == -1 ? cur : parent;
    for (int i = 0; i < tree[cur].size(); i++)
        if (tree[cur][i].first != parent)
            dfs(tree[cur][i].first, cur, d + tree[cur][i].second, l + 1);
}

void lca_init() {
    for (int dep = 1; dep < 30; dep++)
        for (int i = 1; i <= n; i++)
            father[i][dep] = father[father[i][dep-1]][dep-1];
}

int get_lca(int x, int y) {
    if (level[x] < level[y]) swap(x, y);
    int d = level[x] - level[y];
    for (int i = 0; i < 30; i++)
        if (d & (1 << i))
            x = father[x][i];
    if (x == y) return x;
    for (int i = 29; i >= 0; i--)
        if (father[x][i] != father[y][i])
            x = father[x][i], y = father[y][i];
    return father[x][0];
}

LL get_dist(int x, int y) {
    return dist[x] + dist[y] - 2 * dist[get_lca(x, y)];
}

int dfs_sub_size(int cur, int parent) {
    subSize[cur] = 1;
    for (int i = 0; i < tree[cur].size(); i++) {
        int u = tree[cur][i].first;
        if (u != parent && !deleted[u])
            subSize[cur] += dfs_sub_size(u, cur);
    }
    return subSize[cur];
}

int get_centroid(int cur, int parent, int n) {
    for (int i = 0; i < tree[cur].size(); i++)
        if (tree[cur][i].first != parent && !deleted[tree[cur][i].first] 
            && subSize[tree[cur][i].first] > n / 2)
            return get_centroid(tree[cur][i].first, cur, n);
    return cur;
}

void centroid_decomposition(int cur, int parent) {
    int n = dfs_sub_size(cur, parent);
    int centroid = get_centroid(cur, parent, n);
    cd_father[centroid] = parent;
    deleted[centroid] = true;
    for (int i = 0; i < tree[centroid].size(); i++) {
        int u = tree[centroid][i].first;
        if (u != parent && !deleted[u])
            centroid_decomposition(u, centroid);
    }
}

void init() {
    dfs(1, -1, 0, 0);
    lca_init();
    centroid_decomposition(1, -1);

    for (int i = 2; i <= n; i++) {
        if (tree[i].size() != 1) continue;
        // 是叶子
        int cur = i;
        while (cur != -1) {
            segTree[cur].leaves.emplace_back(i, get_dist(i, cur));
            cur = cd_father[cur];
        }
    }
    for (int i = 1; i <= n; i++) {
        segTree[i].init();
    }
}

LL query(int v, int l, int r) {
    LL ans = 1e18;
    int p = v;
    while (p != -1) {
        // upDist并不是累加的！（某个bug曾经出现的位置）
        ans = min(ans, segTree[p].query(l, r, true) + get_dist(v, p));
        p = cd_father[p];
    }
    return ans;
}

int main() {
    scanf("%d %d", &n, &q);
    for (int i = 2; i <= n; i++) {
        int p, w;
        scanf("%d %d", &p, &w);
        tree[i].emplace_back(p, w);
        tree[p].emplace_back(i, w);
    }

    init();

    int v, l, r;
    for (int i = 0; i < q; i++) {
        scanf("%d %d %d", &v, &l, &r);
        printf("%lld\n", query(v, l, r));
    }
    return 0;
}

```