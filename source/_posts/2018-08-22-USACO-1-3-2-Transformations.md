---
title: 'USACO 1.3.2: Transformations'
urlname: usaco-1-3-2-transformations
toc: true
date: 2018-08-22 18:39:37
updated: 2018-08-22 19:02:00
tags: [USACO, alg:Math]
---

## 题意

见[洛谷 P1205](https://www.luogu.org/problemnew/show/P1205)。

## 分析

### 矩阵旋转

模拟矩阵变换，包括反射和旋转。听起来像是计算机图形学基础……如果想不清楚坐标到底怎么变换，手动模拟一下就很显然了。当然，事实上这一点是可以进行严格的数学推导的[^rotate]，但是里面用到了齐次坐标，我看不太懂。

[^rotate]: [How to rotate the positions of a matrix by 90 degrees](https://math.stackexchange.com/questions/1676441/how-to-rotate-the-positions-of-a-matrix-by-90-degrees)

另一个问题是，其实矩阵是可以做到原地旋转的，原理是每四个元素在旋转中组成一个圈，直接循环swap它们就可以了[^inplace]。当然反射更可以做到原地。

[^inplace]: [Inplace rotate square matrix by 90 degrees | Set 1](https://www.geeksforgeeks.org/inplace-rotate-square-matrix-by-90-degrees/)

### 运算符重载

这次标答的写法也用到了`struct`和运算符重载，这样写起来比较方便。不过我又一次（不知道第多少次了）忘掉了运算符重载的写法……

这次重载了`==`和`>>`，其中`>>`需要重载为友元函数，且第一个参数是`istream`。参见[^overload]。

[^overload]: [C++ 二元运算符重载](http://www.runoob.com/cplusplus/binary-operators-overloading.html)

## 代码

```cpp
/*
ID: zhanghu15
TASK: transform
LANG: C++14
*/

#include <iostream>
#include <fstream>
#include <cstring>
#include <algorithm>

using namespace std;

struct Board {
    int n;
    bool tile[11][11];
    bool original[11][11];
    bool tmp[11][11];

    Board(int n) {
        this->n = n;
    }

    friend istream &operator >> (istream& input, Board& b)
    {
        char ch;
        for (int i = 0; i < b.n; i++)
            for (int j = 0; j < b.n; j++) {
                input >> ch;
                if (ch == '@')
                    b.tile[i][j] = 1;
                else
                    b.tile[i][j] = 0;
            }
        memcpy(b.original, b.tile, sizeof(b.tile));
        return input;
    }

    bool operator == (Board& b) {
        if (n != b.n)
            return false;
        for (int i = 0; i < n; i++)
            for (int j = 0; j < n; j++)
                if (tile[i][j] != b.tile[i][j])
                    return false;
        return true;
    }

    void restore() {
        memcpy(tile, original, sizeof(original));
    }

    void turnClockwise90() {
        for (int i = 0; i < n; i++)
            for (int j = 0; j < n; j++)
                tmp[j][n - i - 1] = tile[i][j];
        memcpy(tile, tmp, sizeof(tmp));
    }

    void reflect() {
        for (int i = 0; i < n; i++)
            for (int j = 0; j < n; j++)
                tmp[i][n - j - 1] = tile[i][j];
        memcpy(tile, tmp, sizeof(tmp));
    }
};

int N;

int main() {
    ofstream fout("transform.out");
    ifstream fin("transform.in");

    fin >> N;

    Board from(N), to(N);
    fin >> from >> to;

    // #1
    from.turnClockwise90();
    if (from == to) {
        fout << 1 << endl;
        return 0;
    }

    // #2
    from.turnClockwise90();
    if (from == to) {
        fout << 2 << endl;
        return 0;
    }

    // #3
    from.turnClockwise90();
    if (from == to) {
        fout << 3 << endl;
        return 0;
    }

    // #4
    from.restore();
    from.reflect();
    if (from == to) {
        fout << 4 << endl;
        return 0;
    }

    // #5
    for (int i = 0; i < 3; i++) {
        from.turnClockwise90();
        if (from == to) {
            fout << 5 << endl;
            return 0;
        }
    }

    from.restore();
    if (from == to) {
        fout << 6 << endl;
        return 0;
    }

    fout << 7 << endl;
    return 0;
}
```
