---
title: Codeforces 1107E. Vasya and Binary String（DP）
urlname: codeforces-1107e-vasya-and-binary-string
toc: true
date: 2019-01-31 17:31:57
updated: 2019-01-31 17:31:57
tags: [Codeforces, Codeforces Contest, alg:Dynamic Programming]
---

题目来源：[https://codeforces.com/contest/1107/problem/E](https://codeforces.com/contest/1107/problem/E)

提交次数：1/1

## 题意

给定一个长度为`n`的只包含0和1的字符串，将该串中连续`i`个相同字符消去将得到`a[i]`的收益，问消去整个字符串能得到的最大收益是多少？

## 分析

在不会做的三道题上鸽了好久，最后还是觉得DP最好写……

---

这个题解[soln1]特别简明扼要，大概抓住了问题的本质，不过我可能还没有完全理解它：

* 每一个状态都可以用`[start, end, prefix]`表示，其中`start`和`end`是字符串中的起始和结束index（不妨假设两端都是闭区间）（不是也行），`prefix`是字符串前面附加的和`s[start]`相等的字符的数量（包括`s[start]`）（不包括大概也行）
* 每一个状态都有两种递推方法：
  * 第一种是直接消去字符串前面附加的字符：`dp[start, end, prefix] = a[prefix] + dp[start + 1, end, 1]`
  * 第二种是在字符串其他位置找到一个和`s[start]`相同的字符，然后消去中间的字符，获得一个更大的前缀：`dp[start, end, prefix] = max(dp[start+1, i-1, 1] + dp[i, end, prefix+1]) (start < i <= end, s[i] == s[start])`

虽然这个做法看起来很有道理，但其实我会有几个疑问。比如说，一些状态看起来其实是等价的：对于字符串`"0001"`，`dp[0, 3, 1]`和`dp[2, 3, 3]`表示的都是整个字符串。显然我们在递推过程中更容易得到前一种（没有被简化的）状态。那这两种状态有什么关系呢？是否需要手动简化？事实上，并不需要手动简化，`dp[0, 3, 1]`在递推中会自动得到`dp[1, 3, 2]`这个状态，并且进一步得到`dp[2, 3, 3]`。所以，与其说后一种状态是“简化之后的”，不如说是另一种对状态的看法，而且是一种一般化的表示。

至于递推的顺序，大概是得`end - start`从小到大，且`prefix`从小到大。这听起来有点麻烦（而且可能会增加复杂度），不如就写成递归形式算了。

[soln1]: [Codeforces Blog - Quick unofficial editorial for Educational Round 59 (Div. 2)](https://codeforces.com/blog/entry/64833)

[soln2]: [Codeforces Blog - Educational Codeforces Round 59 Editorial](https://codeforces.com/blog/entry/64847)

## 代码

```cpp
#include <iostream>
#include <cstring>
using namespace std;
typedef long long int LL;
int n;
bool s[105];
LL a[105];
LL f[105][105][105];

LL calc(int start, int end, int p) {
    if (f[start][end][p] != -1) return f[start][end][p];
    if (start > end) return 0;
    if (start == end) {
        f[start][end][p] = a[p];
        return f[start][end][p];
    }
    f[start][end][p] = a[p] + calc(start + 1, end, 1);
    for (int i = start + 1; i <= end; i++) {
        if (s[i] != s[start]) continue;
        f[start][end][p] = max(f[start][end][p], calc(start + 1, i - 1, 1) + calc(i, end, p + 1));
    }
    return f[start][end][p];
}

int main() {
    cin >> n;
    for (int i = 0; i < n; i++) {
        char ch;
        cin >> ch;
        s[i] = ch - '0';
    }
    for (int i = 1; i <= n; i++)
        cin >> a[i];
    
    memset(f, -1, sizeof(f));
    cout << calc(0, n-1, 1) << endl;
}
```