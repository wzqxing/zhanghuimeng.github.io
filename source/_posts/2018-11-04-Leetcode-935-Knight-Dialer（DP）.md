---
title: Leetcode 935. Knight Dialer（DP）
urlname: leetcode-935-knight-dialer
toc: true
date: 2018-11-04 14:39:53
updated: 2018-11-04 15:44:53
tags: [Leetcode, Leetcode Contest, alg:Dynamic Programming, alg:Math]
---

题目来源：[https://leetcode.com/problems/knight-dialer/description/](https://leetcode.com/problems/knight-dialer/description/)

标记难度：Medium

提交次数：2/2

代码效率：

* 递推：16ms
* 矩阵乘法：8ms

## 题意

在如下的电话键盘的数字键上按照“马走日”的方式走N-1步，问最多能生成多少个不同的长度为N的数字。

![键盘](https://assets.leetcode.com/uploads/2018/10/30/keypad.png)

## 分析

这道题还是很简单的，就直接递推就可以了。令`f[i][j]`表示以数字`i`结尾的走了`j`步的数的总数（也就是长度为`j+1`的数）；然后从图中可以得出从哪些数字可以走到`i`，于是就可以递推了。

### 矩阵乘法

一个事实是，其实可以把每一步的推导过程写成矩阵乘法。由于

```cpp
f[0][i] = (f[4][i-1] + f[6][i-1]) % MOD;
f[1][i] = (f[6][i-1] + f[8][i-1]) % MOD;
f[2][i] = (f[7][i-1] + f[9][i-1]) % MOD;
f[3][i] = (f[4][i-1] + f[8][i-1]) % MOD;
f[4][i] = (f[0][i-1] + f[3][i-1] + f[9][i-1]) % MOD;
f[5][i] = 0;
f[6][i] = (f[0][i-1] + f[1][i-1] + f[7][i-1]) % MOD;
f[7][i] = (f[2][i-1] + f[6][i-1]) % MOD;
f[8][i] = (f[1][i-1] + f[3][i-1]) % MOD;
f[9][i] = (f[2][i-1] + f[4][i-1]) % MOD;
```

因此

```
f[0][i] = [0 0 0 0 1 0 1 0 0 0] * f[0][i-1]
f[1][i] = [0 0 0 0 0 0 1 0 1 0] * f[1][i-1]
f[2][i] = [0 0 0 0 0 0 0 1 0 1] * f[2][i-1]
f[3][i] = [0 0 0 0 1 0 0 0 1 0] * f[3][i-1]
f[4][i] = [1 0 0 1 0 0 0 0 0 1] * f[4][i-1]
f[5][i] = [0 0 0 0 0 0 0 0 0 0] * f[5][i-1]
f[6][i] = [1 1 0 0 0 0 0 1 0 0] * f[6][i-1]
f[7][i] = [0 0 1 0 0 0 1 0 0 0] * f[7][i-1]
f[8][i] = [0 1 0 1 0 0 0 0 0 0] * f[8][i-1]
f[9][i] = [0 0 1 0 1 0 0 0 0 0] * f[9][i-1]
```

然后就可以利用矩阵快速幂的方法优化成`O(log(N))`了。真是非常优秀的想法……[^lee215]

[^lee215]: [lee215's Solution for Leetcode 915 - Knight Dialer](https://leetcode.com/problems/knight-dialer/discuss/189252/O%28logN%29)

---

以及为什么Leetcode里python可以用numpy啊，这不科学，我希望C++增加Eigen库……

## 代码

### 递推

```cpp
class Solution {
private:
    long long int f[10][5005];

public:
    int knightDialer(int N) {
        memset(f, -1, sizeof(f));
        const long long int MOD = 1000000007;
        long long int ans = 0;
        for (int i = 0; i < 10; i++) f[i][0] = 1;
        for (int i = 1; i < N; i++) {
            f[0][i] = (f[4][i-1] + f[6][i-1]) % MOD;
            f[1][i] = (f[6][i-1] + f[8][i-1]) % MOD;
            f[2][i] = (f[7][i-1] + f[9][i-1]) % MOD;
            f[3][i] = (f[4][i-1] + f[8][i-1]) % MOD;
            f[4][i] = (f[0][i-1] + f[3][i-1] + f[9][i-1]) % MOD;
            f[5][i] = 0;
            f[6][i] = (f[0][i-1] + f[1][i-1] + f[7][i-1]) % MOD;
            f[7][i] = (f[2][i-1] + f[6][i-1]) % MOD;
            f[8][i] = (f[1][i-1] + f[3][i-1]) % MOD;
            f[9][i] = (f[2][i-1] + f[4][i-1]) % MOD;
        }
        for (int i = 0; i < 10; i++)
            ans = (ans + f[i][N-1]) % MOD;
        return ans;
    }
};
```

### 矩阵乘法

```cpp
class Solution {
private:
    struct Matrix {
        int n, m;
        long long int mat[20][20];

        Matrix(int _n, int _m) {
            n = _n;
            m = _m;
            memset(mat, 0, sizeof(mat));
        }

        friend Matrix operator * (const Matrix& a, const Matrix& b) {
            int n = a.n, m = b.m;
            Matrix c(n, m);
            for (int i = 0; i < n; i++)
                for (int j = 0; j < a.m; j++)
                    for (int k = 0; k < m; k++) {
                        c.mat[i][k] = (c.mat[i][k] + a.mat[i][j] * b.mat[j][k]) % MOD;
                    }
            return c;
        }

        long long int sum() {
            long long int sum = 0;
            for (int i = 0; i < n; i++)
                for (int j = 0; j < m; j++)
                    sum = (sum + mat[i][j]) % MOD;
            return sum;
        }
    };

public:
    static const long long MOD = 1000000007;

    int knightDialer(int N) {
        Matrix f(10, 1);
        for (int i = 0; i < 10; i++)
            f.mat[i][0] = 1;
        Matrix A(10, 10);
        A.mat[0][4] = A.mat[0][6] = 1;
        A.mat[1][6] = A.mat[1][8] = 1;
        A.mat[2][7] = A.mat[2][9] = 1;
        A.mat[3][4] = A.mat[3][8] = 1;
        A.mat[4][0] = A.mat[4][3] = A.mat[4][9] = 1;
        A.mat[6][0] = A.mat[6][1] = A.mat[6][7] = 1;
        A.mat[7][2] = A.mat[7][6] = 1;
        A.mat[8][1] = A.mat[8][3] = 1;
        A.mat[9][2] = A.mat[9][4] = 1;

        Matrix I(10, 10);
        for (int i = 0; i < 10; i++)
            I.mat[i][i] = 1;
        Matrix pow2 = A, pow = I;
        N--;
        while (N > 0) {
            if (N % 2 != 0)
                pow = pow * pow2;
            pow2 = pow2 * pow2;
            N >>= 1;
        }

        f = pow * f;
        return f.sum();
    }
};
```
