---
title: Codeforces 1037D. Valid BFS?（BFS）
urlname: codeforces-1037d-valid-bfs
toc: true
date: 2018-09-03 18:59:19
updated: 2018-09-03 21:17:00
tags: [Codeforces, Codeforces Contest, alg:Breadth-first Search]
---

题目来源：[https://codeforces.com/contest/1037/problem/D](https://codeforces.com/contest/1037/problem/D)

提交次数：2/3

## 题意

给定一棵结点编号从`1`到`n`的无根树，以及一个BFS序列，问该BFS序列是否为合法的树的BFS序列。保证BFS算法必从结点`1`开始，但BFS序列的第一个结点未必是`1`。

## 分析

比赛的时候我想了一种模拟的算法，经过一定的debug之后过了pretest，但后来在大数据量下超时了。

---

### 模拟算法

我的模拟算法很简单：

* 对每个结点，记录它是否已被访问（`visited[x]`）、已被访问的邻居个数（`count[x]`），以及总邻居个数（`degree[x]`）
* 用一个队列维护已被访问，但是邻居尚未都被访问的结点
* 验证BFS序列的第一个结点为`1`，`visited[1] = true`，并将`1`入队
* 顺序枚举BFS序列的其他结点`y`：
  * 验证`visited[y] == false`
  * 将队列中满足`count[x] == degree[x]`的结点全部弹出，直到得到第一个还有邻居未被访问的结点`x`
  * 验证`y`是`x`的邻居
  * `count[x]++`，`count[y] = 1`（因为`y`的邻居`x`已经被访问过了），并将`y`入队

问题是如何验证`y`是`x`的邻居。如果直接在邻接表中顺序查找，则在最坏情况下（`1`有`n-1`个孩子）上述算法的复杂度是`O(n^2)`，对于`n = 200000`的数据显然是过不了的。所以要把邻接表排序，然后用二分查找在里面寻找元素，这样复杂度在最坏情况下也降低到了`O(n * log(n))`。

### 构造算法

题解[^solution]中给出了另一种构造算法：

* 将每个结点的邻居按它们在BFS序列中出现的顺序排序
* 按上述排序之后的顺序进行一次BFS
* 比较两次BFS得到的结点序列是否相同

这种做法的正确性在于，事实上，决定BFS结果的是访问每个结点的子结点的顺序，而这一访问顺序和在BFS中出现的顺序显然是相同的。[^submission]

排序的时间复杂度最坏是`O(n * log(n))`，BFS的时间是`O(n)`，总时间复杂度和上一种差不多，但写出来的代码比上一种简洁很多。

[^solution]: [Manthan, Codefest'18, IIT (BHU) Editorial](https://codeforces.com/blog/entry/61606)

[^submission]: [官方代码](https://codeforces.com/contest/1037/submission/42406857)

## 代码

### 模拟算法

```cpp
#include <iostream>
#include <cstdio>
#include <queue>
#include <cstring>
#include <algorithm>
#include <vector>
using namespace std;
int main() {
    int n, x, y;
    cin >> n;
    vector<vector<int>> graph(n + 1, vector<int>());
    int bfs[n + 1];
    int marked[n + 1];
    bool visited[n + 1];
    queue<int> q;
    for (int i = 1; i < n; i++) {
        scanf("%d %d", &x, &y);
        graph[x].push_back(y);
        graph[y].push_back(x);
    }
    // 邻接表排序
    for (int i = 1; i <= n; i++)
        sort(graph[i].begin(), graph[i].end());

    for (int i = 0; i < n; i++)
        scanf("%d", &bfs[i]);
    if (bfs[0] != 1) {
        cout << "No" << endl;
        return 0;
    }
    memset(marked, 0, sizeof(marked));
    memset(visited, 0, sizeof(visited));
    visited[1] = true;
    q.push(1);
    bool ok = true;
    for (int i = 1; i < n; i++) {
        int x;
        if (q.empty()) {
            ok = false;
            break;
        }
        while (!q.empty()) {
            x = q.front();
            if (marked[x] == graph[x].size())
                q.pop();
            else
                break;
        }
        int y = bfs[i];
        if (visited[y]) {
            ok = false;
            break;
        }
        visited[y] = true;
        // 寻找孩子结点
        auto idx = lower_bound(graph[x].begin(), graph[x].end(), y);
        if (idx == graph[x].end() || *idx != y) {
            ok = false;
            break;
        }
        marked[x]++;
        marked[y] = 1;
        q.push(y);
    }

    if (!ok)
        cout << "No" << endl;
    else
        cout << "Yes" << endl;

    return 0;
}
```

### 构造算法

```cpp
// 参考：https://codeforces.com/contest/1037/submission/42406857
#include <iostream>
#include <cstdio>
#include <queue>
#include <cstring>
#include <algorithm>
#include <vector>

using namespace std;

vector<int> adj[200005];
int bfs[200005];
int ans[200005];
int pos[200005];
bool visited[200005];

int cmp(int x, int y) {
    return pos[x] < pos[y];
}

int main() {
    int n, x, y;
    ios_base::sync_with_stdio(false); cin.tie(nullptr); cout.tie(nullptr);
    cin >> n;
    for (int i = 1; i < n; i++) {
        cin >> x >> y;
        adj[x].push_back(y);
        adj[y].push_back(x);
    }
    for (int i = 0; i < n; i++) {
        cin >> bfs[i];
        pos[bfs[i]] = i;
    }
    // 按结点出现顺序为邻接表排序
    for (int i = 1; i <= n; i++)
        sort(adj[i].begin(), adj[i].end(), cmp);

    // 普通的BFS
    int m = 0;
    queue<int> q;
    visited[1] = true;
    q.push(1);
    ans[m++] = 1;
    while (!q.empty()) {
        int x = q.front();
        q.pop();
        for (int child: adj[x])
            if (!visited[child]) {
                visited[child] = true;
                ans[m++] = child;
                q.push(child);
            }
    }

    for (int i = 0; i < n; i++) {
        if (ans[i] != bfs[i]) {
            cout << "No" << endl;
            return 0;
        }
    }
    cout << "Yes" << endl;
    return 0;
}
```
