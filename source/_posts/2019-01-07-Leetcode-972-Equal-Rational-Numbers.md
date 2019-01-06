---
title: Leetcode 972. Equal Rational Numbers
urlname: leetcode-972-equal-rational-numbers
toc: true
date: 2019-01-07 01:14:10
updated: 2019-01-07 02:00:00
tags: [Leetcode, Leetcode Contest, alg:Math]
---

题目来源：[https://leetcode.com/problems/equal-rational-numbers/description/](https://leetcode.com/problems/equal-rational-numbers/description/)

标记难度：Hard

提交次数：2/4

代码效率：

* 转成分数：4ms
* 精度hack：4ms

## 题意

给定两个普通小数或循环小数的字符表示，问这两个小数是否相等。

## 分析

比赛的时候我没做出来，准确的说法是没做完（因为期间有其他的事情）。我的思路是这样的：

记小数的三个部分分别为`A`，`B`，`C`：则这个小数实际上就是`A.BCCCCC...`。记这个小数为`x`，在它两侧分别乘上`10^B.length`和`10^(B.length + C.length)`，则会得到

```
             10^B.length x =  AB.CCCC...
10^(B.length + C.length) x = ABC.CCCC...
```

下式减上式，得到

```
10^B.length (10^C.length - 1) x = ABC - AB
```

即

```
x = (ABC - AB) / (10^B.length (10^C.length - 1))
```

分别求出分子分母后化简，即可判断两个分数是否相同。

---

当然我知道，看到这道题的第一个思路一般是直接判断这两个数转成double之后是否相等。这样做显然可能有精度问题，不过因为题目要求中整数部分和非重复部分长度都很短，所以把重复部分多重复几次，还是有可能过的。[^lee215]

[^lee215]: [lee215's solution for Leetcode 972 - \[C++/Python\] Easy Cheat](https://leetcode.com/problems/equal-rational-numbers/discuss/214203/C++Python-Easy-Cheat)

## 代码

### 分数转换

```cpp
class Solution {
    typedef long long int LL;
    
private:
    LL pow10(int i) {
        LL x = 1;
        while (i--) x *= 10;
        return x;
    }
    
    LL gcd(LL x, LL y) {
        if (y == 0) return x;
        return gcd(y, x % y);
    }
    
    pair<LL, LL> toFrac(LL A, LL lb, LL B, LL lc, LL C) {
        if (lb == 0 && lc == 0) return {A, 1};
        if (lc == 0) return {A * pow10(lb) + B, pow10(lb)};
        
        LL up, down;  // 分子和分母（我懒得记它们的英文了，就分数线上下好了）
        LL ABC = (A * pow10(lb) + B) * pow10(lc) + C;
        LL AB = A * pow10(lb) + B;
        up = ABC - AB;
        down = pow10(lb) * (pow10(lc) - 1);

        LL g = gcd(up, down);
        up /= g;
        down /= g;
        return {up, down};
    }
    
    // 将小数表示分成A（整数）、B（不重复部分小数）、C（重复部分小数）三个部分
    // 虽然也许用stod和substr之类的函数会更快……
    void parse(string S, LL& A, LL& lb, LL& B, LL& lc, LL& C) {
        int flag = 0;
        A = lb = B = lc = C = 0;
        for (char ch: S) {
            if (flag == 0) {
                if ('0' <= ch && ch <= '9') A = A * 10 + ch - '0';
                else if (ch == '.') {
                    flag = 1;
                    lb = B = 0;
                }
            }
            if (flag == 1) {
                if ('0' <= ch && ch <= '9') {
                    B = B * 10 + ch - '0';
                    lb++;
                }
                else if (ch == '('){
                    flag = 2;
                    lc = C = 0;
                }
            }
            if (flag == 2) {
                if ('0' <= ch && ch <= '9') {
                    C = C * 10 + ch - '0';
                    lc++;
                }
            }
        }
    }
    
public:
    bool isRationalEqual(string S, string T) {
        LL A, lb, B, lc, C;
        parse(S, A, lb, B, lc, C);
        pair<LL, LL> p1 = toFrac(A, lb, B, lc, C);
        parse(T, A, lb, B, lc, C);
        pair<LL, LL> p2 = toFrac(A, lb, B, lc, C);
        return p1 == p2;
    }
};
```

### 精度hack

```cpp
class Solution {
private:
    double convert(string S) {
        int i = S.find('(');
        if (i == -1) return stod(S);
        string start = S.substr(0, i);
        string rep = S.substr(i + 1, S.length() - i - 2);
        for (int j = 0; j < 20; j++)
            start += rep;
        return stod(start);
    }
    
public:
    bool isRationalEqual(string S, string T) {
        return convert(S) == convert(T);
    }
};
```