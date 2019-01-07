---
title: 'USACO 1.6.2: Prime Palindromes'
urlname: usaco-1-6-2-prime-palindromes
toc: true
date: 2019-01-06 20:26:00
updated: 2019-01-07 21:10:00
tags: [USACO, alg:Math]
---

## 题意

见[P1217 \[USACO1.5\]回文质数 Prime Palindromes](https://www.luogu.org/problemnew/show/P1217)。

求`[a, b]`（`5 <= a < b <= 100,000,000`）中所有的质回文数。

## 分析

上次从[Leetcode 906](/post/leetcode-906-super-palindromes)中学到的技巧是，在需要找出具有某种性质的回文数时，先生成回文数，再判断性质。这是因为生成回文数的方法实际上是很简单的，随便拿一个数，把它反过来再接到后面就行。当然，需要注意生成奇数和偶数长度两种回文数的方法。

还有就是，需要判断`10^8`范围内的数是否为质数时，只需要打`10^4`范围内的表，这是因为，`10^8`范围内的合数必然有`10^4`范围内的因子。

怎么确定上下界是一个问题。我的方法是分别计算出`a`和`b`的长度，然后只生成在相应长度范围的回文数；生成之后再判断是否在范围内。

---

题解里更详细地叙述了从小到大生成回文数的过程：

* 生成长度为1的回文数：用`1..9`进行翻转，重复中间字符，得到`1..9`
* 生成长度为2的回文数：用`1..9`进行翻转，不重复中间字符，得到`11..99`
* 生成长度为3的回文数：用`10..99`进行翻转，重复中间字符，得到`101...999`
* 生成长度为4的回文数：用`10..99`进行翻转，不重复中间字符，得到`1001...9999`

由于回文数一共也只有10000个左右，直接全生成出来再判断是否在`[a, b]`范围内也可以。

另一个观察是，任何长度为偶数的回文数都是11的倍数，所以可以不管除了11之外的所有长度为偶数的回文数。[^11]

[^11]: [stackexchange - Proving a palindromic integer with an even number of digits is divisible by 11](https://math.stackexchange.com/a/985360)

---

质回文数也是一个被正式定义了的序列。[^oeis]

[^oeis]: [OEIS - A002385 Palindromic primes: prime numbers whose decimal expansion is a palindrome.](https://oeis.org/A002385)

## 代码

```cpp
/*
ID: zhanghu15
TASK: pprime
LANG: C++14
*/

#include <iostream>
#include <fstream>
#include <vector>
#include <algorithm>
#include <cstring>
#include <cmath>

using namespace std;
typedef long long int LL;

int prime[10000];
int n;
bool isPrime[10000];
void init() {
    memset(isPrime, 1, sizeof(isPrime));
    isPrime[1] = false;
    for (int i = 2; i < 10000; i++) {
        if (isPrime[i]) {
            prime[n++] = i;
            for (int j = i * i; j < 10000; j += i)
                isPrime[j] = false;
        }
    }
}

bool checkIsPrime(LL x) {
    if (x < 10000) return isPrime[x];
    for (int i = 0; i < n; i++) {
        if (x % prime[i] == 0) return false;
    }
    return true;
}

LL getPalindrome(LL x, int isOdd) {
    LL l = 0, d[10], y = x;
    while (y > 0) {
        d[l++] = y % 10;
        y /= 10;
    }
    for (int i = isOdd; i < l; i++)
        x = x*10 + d[i];
    return x;
}

int getLen(LL x) {
    int l = 0;
    while (x > 0) {
        l++;
        x /= 10;
    }
    return l;
}

int pow10[10] = {1, 10, (int) 1e2, (int) 1e3, (int) 1e4, (int) 1e5, (int) 1e6, 
                 (int) 1e7, (int) 1e8, (int) 1e9};

int main() {
    ofstream fout("pprime.out");
    ifstream fin("pprime.in");
    int a, b;
    fin >> a >> b;

    // 算出[1, 10000]范围内所有的质数
    init();

    // 枚举奇数和偶数长度的回文数（注意long long int）
    int len1 = getLen(a) / 2, len2 = ceil(getLen(b) / 2.0);
    for (int l = len1; l <= len2; l++) {
        for (LL i = pow10[l]; i < pow10[l+1]; i++) {
            LL x = getPalindrome(i, 1);
            if (a <= x && x <= b && checkIsPrime(x))
                fout << x << endl;
        }
        for (LL i = pow10[l]; i < pow10[l+1]; i++) {
            LL x = getPalindrome(i, 0);
            if (a <= x && x <= b && checkIsPrime(x))
                fout << x << endl;
        }
    }
    return 0;
}
```