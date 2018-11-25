---
title: Leetcode 887. Super Egg Drop（动态规划）
urlname: leetcode-887-super-egg-drop
toc: true
date: 2018-10-07 10:50:56
updated: 2018-11-25 18:59:56
tags: [Leetcode, Leetcode Contest, alg:Dynamic Programming, alg:Math]
---

题目来源：[https://leetcode.com/problems/super-egg-drop/description/](https://leetcode.com/problems/super-egg-drop/description/)

标记难度：Hard

提交次数：1/3

代码效率：

* 动态规划（plain）：TLE
* 动态规划（二分优化）：14.92%

## 题意

你有`K`个鸡蛋，它们从高度`>F`的地方被丢下来时会破碎，但从高度`<=F`的地方被丢下来时不会破碎。但是你不知道`F`的值。现有一栋高楼，高度`N`满足`0 <= F <= N`，请在这栋楼上通过丢鸡蛋的方式尝试确定`F`的值。问：对于任意`F`，至少需要丢多少次鸡蛋才能确定`F`的值？

---

题意还是挺绕的。丢鸡蛋这一点我当时就想了很久也没明白……比如说，你只有一个鸡蛋：假如直接把它从高度为`N`的地方丢下来，鸡蛋有可能碎了，也可能没有碎。如果它没有碎，则可以确定`F = N`；否则只能知道`F < N`，而且鸡蛋已经碎了，没有更多鸡蛋可以扔了，所以无法确定`F`。因此更合理的做法是，先把鸡蛋从`1`处扔下来，假如碎了则`F = 0`，结束；否则可知`F > 0`，再把鸡蛋从`2`处扔下来，假如碎了则`F = 1`，否则可知`F > 1`……以此类推。所以`K = 1`时需要的丢鸡蛋次数为`N - 1`。

因为只有一个鸡蛋，所以不能冒太大风险；假如有两个鸡蛋，就可以先把其中一个从`N / 2`处丢下来。假如鸡蛋碎了，则可以知道`F < N / 2`；否则`F >= N / 2`……以此类推。

## 分析

比赛的时候我好像尝试这样推导了一下：以两个鸡蛋的情况为例，如果进行二分式的扔法，就会出现这样的情形：

* 从`N/2`处丢下鸡蛋，鸡蛋碎了，于是之后只能从下往上逐层丢鸡蛋，需要约`N/2`次操作
* 从`N/2`处丢下鸡蛋，鸡蛋没碎，于是可以把这个鸡蛋再次从`3/4*N`层丢下去
  * 如果鸡蛋碎了，则之后需要从`N/2+1`到`3/4*N-1`逐层丢鸡蛋，需要约`N/4`次操作
  * 如果鸡蛋没碎，则可以把这个鸡蛋再次从`7/8*N`层丢下去
  * ……

可以看出，鸡蛋碎了的情况需要的迭代次数比较多，所以不应该二分，而应该偏分，鸡蛋可能碎掉的情况分配的层数少一点……比如三分？

想到这里比赛就结束了……

---

### 动态规划

事实上正解之一是动态规划。之前分析的时候并没有意识到这一点：丢下一个鸡蛋，它碎了/没碎之后，事实上当前的状态仍然可以用鸡蛋数目和总层数（如果鸡蛋碎了则层数上限减小，否则层数下限增大，但和从0开始的层数的情况仍然相同）来概括。于是我们就得到了一个子状态。

令`f[n][k]`表示用`k`个鸡蛋在`n`层中确定`F`至少需要的操作次数，则`f[n][k] = 1 + min(max(f[i][k-1], f[n-i][k]))`，且`f[n][1] = n`，`f[0][k] = 0`。这种做法的复杂度为`O(N^2 * K)`。[^Greatjian]

[^Greatjian]: [Greatjian's solution for Leetcode 887 - Python DP from kn^2 to knlogn to kn](https://leetcode.com/problems/super-egg-drop/discuss/159079/Python-DP-from-kn2-to-knlogn-to-kn)

然后就TLE了。（也对，N的范围是10000）

对于一个确定的`k`，显然`f[n][k]`随着`n`的增大而增大。（如果这从式子并不显然的话……那么显然如果鸡蛋不变，层数越多，所需的操作数就越多）。所以`f[i][k-1]`是随着`i`的增大而增大的，`f[n-i][k]`是随着`i`的增大而减小的。因此这两者的最大值是随着`i`先减小再增大的。所以可以通过二分查找找到这个点。这种做法的复杂度是`O(N\*K\*log(N))（注意边界条件）[^solution]

[^solution]: [Leetcode's Official Solution for Leetcode 887](https://leetcode.com/problems/super-egg-drop/solution/)

还有一种做法可以优化到`O(KN)`，但我不想管它了，毕竟这道题拖了有一个多月了，太累了……

## 代码

### 普通的动态规划

```cpp
class Solution {
public:
    int superEggDrop(int K, int N) {
        int f[N + 1][K + 1];
        for (int i = 0; i <= K; i++) {
            f[0][i] = 0;  // zero floor, no need to check
            f[1][i] = 1;  // one floor, check once
        }
        for (int i = 0; i <= N; i++) {
            f[i][1] = i;  // one egg checks all floors
            f[i][0] = 0;  // no egg to check
        }

        for (int i = 2; i <= N; i++) {
            for (int j = 2; j <= K; j++) {
                f[i][j] = INT_MAX;
                for (int k = 1; k < i; k++) {
                    f[i][j] = min(f[i][j], max(f[i - k][j], f[k - 1][j - 1]) + 1);
                }
            }
        }
        return f[N][K];
    }
};
```

### 二分优化的动态规划

```cpp
class Solution {
public:
    int superEggDrop(int K, int N) {
        int f[N + 1][K + 1];
        for (int i = 0; i <= K; i++) {
            f[0][i] = 0;  // zero floor, no need to check
            f[1][i] = 1;  // one floor, check once
        }
        for (int i = 0; i <= N; i++) {
            f[i][1] = i;  // one egg checks all floors
            f[i][0] = 0;  // no egg to check
        }
        
        for (int i = 2; i <= N; i++) {
            for (int j = 2; j <= K; j++) {
                f[i][j] = INT_MAX;
                
                // 注意边界条件
                int left = 1, right = i;
                while (left + 1 < right) {
                    int mid = (left + right) / 2;
                    if (f[i - mid][j] < f[mid - 1][j - 1])
                        right = mid;
                    else if (f[i - mid][j] > f[mid - 1][j - 1])
                        left = mid;
                    else
                        left = right = mid;
                }
                f[i][j] = min(max(f[right - 1][j - 1], f[i - right][j]), max(f[left - 1][j - 1], f[i - left][j])) + 1;
            }
        }
        return f[N][K];
    }
};
```
