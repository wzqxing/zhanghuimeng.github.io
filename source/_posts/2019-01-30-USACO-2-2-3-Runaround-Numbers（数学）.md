---
title: 'USACO 2.2.3: Runaround Numbers（数学）'
urlname: usaco-2-2-3-runaround-numbers
toc: true
date: 2019-01-30 01:32:39
updated: 2019-01-30 01:32:39
tags: [USACO, alg:Math]
---

## 题意

见[洛谷 P1467 循环数 Runaround Numbers](https://www.luogu.org/problemnew/show/P1467)。

定义循环数为满足以下条件的整数：

* 各个数位都不重复
* 从最左边的数位开始进行如下操作：
  * 向右循环数该数位对应的数字个数
  * 然后从得到的数位开始继续数
  * 在各个数位都（不重不漏地）开始数了一遍之后，会回到最开始的数

给定正整数`M`，问大于`M`的下一个循环数是多少？

## 分析

按我惯常的习惯，肯定是先去[OEIS](https://oeis.org/)上查一查，不过结果是并没有。

做题的时候，我首先写了一个暴力判断某个数是否为循环数的代码。本来应该想想怎么优化（毕竟`M`据说最多9位，如果直接暴力挨个判断比`M`大的数会超时的吧），但是那天不知道哪根神经出了毛病，直接把暴力给交上去了。结果居然就过了，还挺快……？？

看了一眼测试数据，发现这范围跟说好的不一样啊，不是说`M`最多有9位吗，怎么这里最多只有7位？

```
------- test 1 [length 3 bytes] ----
99
------- test 2 [length 7 bytes] ----
111110
------- test 3 [length 7 bytes] ----
134259
------- test 4 [length 7 bytes] ----
348761
------- test 5 [length 8 bytes] ----
1000000
------- test 6 [length 8 bytes] ----
5000000
------- test 7 [length 8 bytes] ----
9000000
```

仔细一想，我意识到，9位的循环数是不存在的：假设这样的循环数是存在的，因为要求各个数位不同，那么必须有一个数位为9；但是这个数位数了9位之后只会回到自己，没办法构成循环。由此还可以得到一个推论，`n`位的循环数中必然不存在数位`n`。所以，暴力代码能过这件事看起来就正常一点了。

但是为什么测试数据只有7位呢？我尝试把所有长度<=9位的循环数都打印出来，结果得到的最大的数是9682415。所以，为什么8位的循环数也不存在呢？

我又仔细想了一下，发现8位的循环数确实不应该存在（而不是我的暴力写错了之类的）。由上面的推论可以得到，8位的循环数中没有8，因此只能是1-7和9这8个数字。假设有这样一个8位的循环数，我们从左侧的第`i`位出发，遍历了所有数位各一次后，当前的数位应该是`(i + 1 + 2 + .. + 7 + 9) % 8`，且这个数位应该还是第`i`位。问题是，`1 + 2 + .. + 7 + 9 = 37`，这样根本没办法回到原来的位的！所以8位的循环数必然是不存在的。由此还可以得出另一个推论，就是`n`位的循环数的数位之和必然模`n`余0。

---

当然，比较正确（而不依靠随便乱交……）的做法是，枚举出所有长度为`n`且各个数位不相等的数，然后再判断它们是否为循环数——因为9!=362880，所以不会超时。

题解里还有一种比较神奇的做法——从`M`直接生成下一个各个数位都不相等的数，然后再判断它们是否为循环数。不过他到底是怎么生成下一个数的我就没细看了……

## 分析

```cpp
/*
ID: zhanghu15
TASK: runround
LANG: C++14
*/

#include <iostream>
#include <fstream>
#include <sstream>
#include <vector>
#include <algorithm>
#include <cstring>
#include <cmath>

using namespace std;
typedef long long int LL;

bool test(int x) {
    int digits[10];
    int cnt[10];
    memset(cnt, 0, sizeof(cnt));
    int n = 0;
    while (x > 0) {
        digits[n++] = x % 10;
        if (cnt[x % 10]) return false;
        cnt[x % 10]++;
        x /= 10;
    }
    bool visited[10];
    memset(visited, 0, sizeof(visited));
    int cur = n - 1;
    for (int i = 0; i < n; i++) {
        visited[cur] = true;
        int next = ((cur - digits[cur]) % n + n) % n;
        if (visited[next] && i != n - 1) return false;
        cur = next;
    }
    return cur == n - 1;
}

int main() {
    ofstream fout("runround.out");
    ifstream fin("runround.in");
    int M;
    fin >> M;
    for (LL i = M + 1; i <= (LL) 1e10; i++)
        if (test(i)) {
            fout << i << endl;
            return 0;
        }
    return 0;
}
```