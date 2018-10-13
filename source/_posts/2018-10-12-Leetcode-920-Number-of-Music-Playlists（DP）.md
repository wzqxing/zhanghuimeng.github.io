---
title: Leetcode 920. Number of Music Playlists（DP）
urlname: leetcode-920-number-of-music-playlists
toc: true
date: 2018-10-12 11:02:39
updated: 2018-10-14 02:59:00
tags: [Leetcode, Leetcode Contest, Alg:Dynamic Programming]
---

题目来源：[https://leetcode.com/problems/number-of-music-playlists/description/](https://leetcode.com/problems/number-of-music-playlists/description/)

标记难度：Hard

提交次数：2/2

代码效率：

* DP：75.46%
* DP (Optimized)：75.46%

## 题意

有`N`首不同的歌，共需要播放`L`次，要求：

* 每首歌至少被播放一次
* 每首歌被播放的间隔至少是`K`

求共有多少种播放方式。数据保证`0 <= K < N <= L <= 100`。

## 分析

比赛的时候我大胆地推测这道题是DP。然后就没了，还是不会写……

---

### 动态规划

如果用DP来做，第一个问题是状态方程应该包含哪些变量。不妨先把状态方程设置成`dp[n][l][k]`。然后显然需要思考从哪些状态进行递推这个问题。我觉得之前我总会知道递归的状态的，但现在这好像是一个值得思考的问题。

* `dp[n-1][l][k]`：比现在少一首歌，但播放列表长度仍然是`l`。如何把多出来的歌塞进播放列表中是一个问题。
* `dp[n][l-1][k]`：和现在歌的数量相同，但播放列表长度是`l-1`。任何在`[l-1-k, l-1]`位置没有被播放过的歌都可以被再播一遍；而且这一区间内的歌必然是没有重复的。这大概是此题的一个重要性质：合法的播放列表中，任何长度为`k`的区间内的歌都是互不重复的。很好。所以答案`+= dp[n][l-1][k] * (n - k)`。
* `dp[n][l][k-1]`：因为破坏了之前的性质，所以感到很难用这个方法来递推。
* `dp[n-1][l-1][k]`：比现在少一首歌，播放列表长度也少一首。<del>把新歌插入到播放列表的任何位置都可以（不会缩小任何两首相同的歌之间的间隔），因此答案`+= dp[n-1][l-1][k] * (l + 1)`。</del>只需把新歌插入到播放列表的最后位置。这首歌可以是任意一首，因此答案`+= dp[n-1][l-1][k] * n`。[^lee215]

[^lee215]: [lee215's DP Solution for Leetcode 920](https://leetcode.com/problems/number-of-music-playlists/discuss/178415/C++JavaPython-DP-Solution)

结论是，我们需要考虑的子状态只包括`dp[n][l-1][k]`和`dp[n-1][l-1][k]`。由于题目中有`N <= L`的要求，这一设置是合理的。同时，由于发现根本没有用到`k`，不妨把`k`去掉，得到`dp[n][l-1]`和`dp[n-1][l-1]`。

以及上面被划掉的部分说明我没有理解清楚此处**DP**到底意味着什么。DP的状态本身表示什么是很容易说明的：`dp[n][l]`表示长度为`l`且有`n`首不同的歌的播放列表的总数；而`dp[n][l] += dp[n-1][l-1][k] * n`表示的是，从当前的`n`首歌里任选一首删除，用`n-1`首歌组成长度为`l-1`的播放列表；最后再把选出来的这首歌放在播放列表最后。之所以不是把这首歌插入到播放列表的各个位置，是因为那样就会得到与其他歌重复的状态，因此需要加以限制。

### 数学+DP

这部分是题解提供的。说实话，我并没有太看懂题解……所以就不写了……[^solution]

[^solution]: [Leetcode 920 Official Solution](https://leetcode.com/articles/number-of-music-playlists/)

## 代码

### DP

```cpp
class Solution {
public:
    int numMusicPlaylists(int N, int L, int K) {
        long long int f[N + 1][L + 1];
        memset(f, 0, sizeof(f));
        f[1][1] = 1;
        for (int i = 1; i <= N; i++)
            for (int j = 1; j <= L; j++) {
                f[i][j] += f[i][j-1] * max(i - K, 0) + f[i-1][j-1] * i;
                f[i][j] %= 1000000007;
            }
        return f[N][L];
    }
};
```

### DP (Optimized)

显然上述代码还有很多可以优化的地方，包括但不限于[^lee215]：

* 显然只有`j >= i`时`f[i][j]`才有意义。
* 当`i == K + 1`时，意味着我们只能先播放`K + 1`首歌曲，然后按相同顺序继续播放同样的歌曲，因此总的可能性数量为`i!`。
* 当`i == j`时，歌曲数量和播放列表长度相同，不需要考虑`K`的问题，因此总的可能性数量也为`i!`。

可以根据上述内容对DP代码进行优化。（虽然结果好像也没快多少。）

```cpp
class Solution {
private:
    inline long long factorial(int x) {
        long long int sum = 1;
        while (x) sum = (sum * x--) % 1000000007;
        return sum;
    }

public:
    int numMusicPlaylists(int N, int L, int K) {
        long long int f[N + 1][L + 1];
        for (int i = K + 1; i <= N; i++)
            for (int j = i; j <= L; j++) {
                if (i == K + 1 || i == j)
                    f[i][j] = factorial(i);
                else
                    f[i][j] = (f[i][j-1] * (i - K) + f[i-1][j-1] * i) % 1000000007;
            }
        return f[N][L];
    }
};
```
