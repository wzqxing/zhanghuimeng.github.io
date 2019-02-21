---
title: Codeforces Educational Codeforces Round 60 总结
urlname: codeforces-educational-codeforces-round-60-summary
toc: true
date: 2019-02-20 18:50:42
updated: 2019-02-21 19:32:00
tags: [Codeforces, Codeforces Contest, alg:Binary Search, alg:Matrix, alg:Math]
categories: Codeforces
---

这次比赛做得乱七八糟，事实上一共只做出来两道题，结果rating还涨了一点点，看来确实有点难。。

[^sln1]: [Announcement - Educational Codeforces Round 60 \[Rated for Div. 2\]](https://codeforces.com/blog/entry/65308)

[^sln2]: [Educational Codeforces Round 60 Editorial](https://codeforces.com/blog/entry/65365)

## 1117A

题目来源：[https://codeforces.com/contest/1117/problem/A](https://codeforces.com/contest/1117/problem/A)

提交次数：1/1

### 题意

给定一个数组，问数组中满足平均数最大的前提下长度最长的子数列。

### 分析

就是找到最大值然后计算最大值最多连续出现了多少次而已。水题。

### 代码

```cpp
#include <iostream>
using namespace std;
int n;
int a[100005];
int main() {
    int maxn = -1;
    cin >> n;
    for (int i = 0; i < n; i++) {
        cin >> a[i];
        maxn = max(maxn, a[i]);
    }
    int ans = 0, l = 1;
    for (int i = 1; i <= n; i++) {
        if (i == n || a[i] != a[i - 1]) {
            if (a[i - 1] == maxn) ans = max(l, ans);
            l = 1;
        }
        else l++;
    }
    cout << ans << endl;
    return 0;
}
```

## 1117B

题目来源：[https://codeforces.com/contest/1117/problem/B](https://codeforces.com/contest/1117/problem/B)

提交次数：1/1

### 题意

给定`n`个emote，每个emote可以将对手的快乐值增加`a[i]`，你可以使用一些emote共`m`次，相同的emote最多连续使用`k`次，问最多能增加多少快乐值？

### 分析

找出值最大和值次大的emote，连续用最大的emote`k`次，用一次次大的emote，再接着用最大的emote，以此类推，直到一共用了`m`次为止。水题。

### 代码

```cpp
#include <iostream>
#include <algorithm>
using namespace std;
typedef long long LL;
LL n, m, k;
LL a[200005];
int main() {
    cin >> n >> m >> k;
    for (int i = 0; i < n; i++)
        cin >> a[i];
    sort(a, a + n);
    LL cnt = m / (k + 1);
    LL ans = cnt * (a[n-1] * k + a[n-2]) + (m - (k + 1) * cnt) * a[n-1];
    cout << ans << endl;
    return 0;
}
```

## 1117C

题目来源：[https://codeforces.com/contest/1117/problem/C](https://codeforces.com/contest/1117/problem/C)

提交次数：1/4

### 题意

你在`(x1, y1)`处有一艘船，每天可以向上或下、左、右移动1格。每天的移动效果会和天气效果叠加，天气效果会使得船根据风向往上或下、左、右移动一格。给定`m`天的天气，之后的天气是循环的，问最少需要几天，船才能到达`(x2, y2)`处？

### 分析

比赛的时候我意识到走完一个天气循环之后，能到的位置是一个菱形了（或者说是一个尖朝上的正方形）；但很可惜我没有意识到这件事的本质，还打算拿计算几何来做（？？）。显然，这件事的本质是这样的：把天气推船走的路和船自己走的路分开，可以得出一个结论：天气在一个循环内推船走的路是固定的，不妨记为`(dx, dy)`；船自己可以在这个点周围走出一个菱形，或者说是所有满足`|x-dx|+|y-dy|<=n`的点。所以对天数二分查找就好了。当然，还是需要重视一下二分查找的前提：只要`x`天能到，那么`y>=x`天也能到。（大不了不动）[^sln2]

除此之外就是注意上界。显然如果能到达`(x2, y2)`，每个天气循环至少要向这个方向前进1格（曼哈顿距离），所以上界是`(|x2-x1| + |y2-y1|)*n`。如果走了这么多还没到，说明到不了了。

### 代码

因为`long long int`的使用错了好多次……

```cpp
#include <iostream>
#include <algorithm>
using namespace std;
typedef long long LL;
LL x1, y1, x2, y2;
int n;
char s[100005];
pair<LL, LL> pos[100005];
LL fx, fy;
// U, R, D, L
int mx[4] = {0, 1, 0, -1};
int my[4] = {1, 0, -1, 0};

bool isOk(LL days) {
    LL sx = (days / n) * fx, sy = (days / n) * fy;
    sx += pos[days % n].first, sy += pos[days % n].second;
    return abs(x2 - sx) + abs(y2 - sy) <= days;
}

int main() {
    cin >> x1 >> y1 >> x2 >> y2;
    x2 -= x1;
    y2 -= y1;
    cin >> n;
    cin >> s;
    for (int i = 1; i <= n; i++) {
        int d;
        if (s[i-1] == 'U') d = 0;
        else if (s[i-1] == 'R') d = 1;
        else if (s[i-1] == 'D') d = 2;
        else if (s[i-1] == 'L') d = 3;
        pos[i].first = pos[i-1].first + mx[d];
        pos[i].second = pos[i-1].second + my[d];
    }
    fx = pos[n].first, fy = pos[n].second;

    LL l = 0, r = (abs(x2) + abs(y2)) * n;
    while (l < r) {
        LL m = (l + r) / 2;
        if (isOk(m)) r = m;
        else l = m + 1;
    }
    if (!isOk(l)) cout << -1 << endl;
    else cout << l << endl;
    return 0;
}
```

## 1117D

题目来源：[https://codeforces.com/contest/1117/problem/D](https://codeforces.com/contest/1117/problem/D)

提交次数：1/1

### 题意

一颗魔法宝石可以分裂成`M`颗普通宝石，问有多少种选择魔法宝石的方法，使得分裂后总的宝石数量是`N`？不同的魔法宝石数量和不同index的分裂被认为是不同的方法。`N<=10^18`，`M<=100`。

### 分析

比赛的时候我努力推了一个公式出来：

$$\sum_{i=0}^{\lfloor N/M \rfloor} \binom{N-(M-1)i}{i}$$

但是肯定不能这么算，因为`N`太大了。我心想：用动态规划估计也不行。

事实上得用到动态规划递推的思路，看到`N`和`M`的大小，我早该想到是矩阵快速幂才对。

令`f[n]`表示`N=n`时的方法数量。显然有两种方法进行递推：一种是加上一块不分裂的宝石（`f[n-1]`）；另一种是加上一块分裂的宝石（`f[n-M]`）。（虽然第二维看起来没有什么意义……）考虑到递推的本质，不需要乘新加的宝石的位置，因此：

```
f[n] = f[n-1] + f[n-M]
```

现在就可以造一个矩阵乘法迭代公式了：

```
| f[n-M]  f[n-M+1]  ...  f[n-1] | * | 0 0 0 ... 0 1 | = | f[n-M+1] |
                                    | 1 0 0 ... 0 0 |   | f[n-M+2] |
                                    | 0 1 0 ... 0 0 |   |  ...     |
                                    | 0 0 1 ... 0 0 |   | f[n-1]   |
                                    | 0 0 0 ... 1 1 |   | f[n]     |
```

然后矩阵快速幂即可。

太久没写矩阵快速幂，手都生了……

### 代码

```cpp
#include <iostream>
#include <cstring>
using namespace std;
typedef long long LL;
LL N;
int M;

const LL P = 1000000007;

struct Matrix {
    int n, m;
    LL a[105][105];

    Matrix(int _n, int _m) {
        memset(a, 0, sizeof(a));
        n = _n;
        m = _m;
    }

    friend Matrix operator * (const Matrix& m1, const Matrix& m2) {
        Matrix m3(m1.n, m2.m);
        for (int i = 1; i <= m1.n; i++)
            for (int j = 1; j <= m1.m; j++)
                for (int k = 1; k <= m2.m; k++)
                    m3.a[i][k] = (m3.a[i][k] + m1.a[i][j] * m2.a[j][k]) % P;
        return m3;
    }

    void print() {
        cout << n << ' ' << m << endl;
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= m; j++)
                cout << a[i][j] << ' ';
            cout << endl;
        }
    }
};

Matrix getIdentity(int n) {
    Matrix mat(n, n);
    for (int i = 1; i <= n; i++)
        mat.a[i][i] = 1;
    return mat;
}

int main() {
    cin >> N >> M;
    if (N < M) {
        cout << 1 << endl;
        return 0;
    }
    Matrix v(1, M);
    for (int i = 1; i < M; i++)
        v.a[1][i] = 1;
    v.a[1][M] = 2;
    
    Matrix x(M, M);
    for (int i = 2; i <= M; i++)
        x.a[i][i - 1] = 1;
    x.a[1][M] = x.a[M][M] = 1;

    LL delta = N - M;
    Matrix pow = x;
    Matrix ans = getIdentity(M);
    while (delta > 0) {
        if (delta & 1) ans = ans * pow;
        delta >>= 1;
        pow = pow * pow;
    }
    v = v * ans;
    cout << v.a[1][M] << endl;
    return 0;
}
```

## 1117E

题目来源：[https://codeforces.com/contest/1117/problem/E](https://codeforces.com/contest/1117/problem/E)

提交次数：1/1

### 题意

这是一道交互题。给定一个字符串，已知对它进行了`<=n`次swap操作；你可以拿出一个新的和它长度一样的字符串，然后得到对这个字符串进行相同的swap之后的结果。你最多可以提交三次字符串。问原来的字符串是多少？

### 分析

比赛的时候，我看了看这道题……觉得很有趣，但应该想半天也做不出来吧。（我从来没在比赛里做对过交互题）这道题的一种解法是这样的：既然每次提交的字符串的swap操作是一样的，不妨把提交的三次字符串看成是一个字符串，它的每个位置是由三个字符组成的一个“超字符”。这样我们就可以认为每个位置的“超字符”是两两不等的了！因为`26^3 = 17576 >= 1e4`，这是可行的。

然后就可以立即得到每个输入位置到输出位置的映射了，倒过来对原来的字符串重新做一遍就行了。

评论区里还出现了用中国剩余定理的做法，我没细看。[^saeed_odak]

[^saeed_odak]: [saeed_odak's comment for 1117E](https://codeforces.com/blog/entry/65365?#comment-493763)

### 代码

```cpp
#include <iostream>
#include <cstring>
using namespace std;

char s[10005];
char q[3][10005];
char a[3][10005];
int mapping[10005];
int n;

int main() {
    cin >> s;
    n = strlen(s);
    int m = 0;
    for (int i = 0; i < 26; i++) {
        for (int j = 0; j < 26; j++) {
            for (int k = 0; k < 26; k++) {
                q[0][m] = i + 'a';
                q[1][m] = j + 'a';
                q[2][m] = k + 'a';
                m++;
                if (m >= n) break;
            }
            if (m >= n) break;
        }
        if (m >= n) break;
    }
    for (int i = 0; i < 3; i++) {
        cout << "? " << q[i] << endl;
        cin >> a[i];
    }
    for (int i = 0; i < n; i++) {
        int idx = (a[0][i] - 'a') * 26 * 26 + (a[1][i] - 'a') * 26 + (a[2][i] - 'a');
        mapping[idx] = i;
    }
    cout << "! ";
    for (int i = 0; i < n; i++)
        cout << s[mapping[i]];
    cout << endl;
    return 0;
}
```