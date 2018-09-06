---
title: 'Codeforces 1040A. Palindrome Dance（贪心），及比赛（Codeforces Round #507 Div. 2）总结'
urlname: codeforces-1040a-palindrome-dance-and-contest-codeforces-round-507
toc: true
date: 2018-09-06 21:33:19
updated: 2018-09-06 21:45:00
tags: [Codeforces, Codeforces Contest, alg:Greedy]
---

题目来源：[http://codeforces.com/contest/1040/problem/A](http://codeforces.com/contest/1040/problem/A)

提交次数：1/1

## 题意

给定`n`个排成一行的舞蹈者，他们会穿黑色和白色的舞蹈服，其中只有一些人已经买了舞蹈服，一些人还没有。白色舞蹈服价格为`a`，黑色舞蹈服价格为`b`。问是否可能给还没有买舞蹈服的人安排服装，使得这`n`个人舞蹈服装的颜色是左右对称的；如果可能的话，服装的最低总价格是多少。

## 分析

这次一共有5个题，我只做出来两个，第三题还交了一次错的，结果居然得了353 / 6242名，Rating增加了106，现在Div2这么水的吗？如果Rating不幸升高到了1700以上，我是不是只能去Div1了（然后一题飘过）？

---

显然是比较简单的一个回文数类型的题。对于`0 <= i <= n / 2`，考虑第`i`和第`n - i - 1`个人的舞蹈服的颜色：

* 首先注意`i == n - i - 1`的情况，如果颜色没有确定，则取价格较低的颜色
* 如果衣服颜色均已确定，则需要判断颜色是否相同，如果不同，输出`-1`
* 如果有一人确定，另一人不确定，则将不确定的人的颜色取为确定的人的颜色
* 如果两人衣服颜色都不确定，则将两人颜色都取为价格较低的颜色

没了。

## 代码

```cpp
#include <iostream>
#include <cstring>
using namespace std;
int color[25];
int main() {
    int n, pay[2], minn;
    cin >> n >> pay[0] >> pay[1];
    minn = min(pay[0], pay[1]);
    for (int i = 0; i < n; i++)
        cin >> color[i];
    int sum = 0;
    for (int i = 0; i <= n - i - 1; i++) {
        if (i == n - i - 1) {
            if (color[i] == 2)
                sum += minn;
            break;
        }
        if (color[i] != 2 && color[n - i - 1] == 2) sum += pay[color[i]];
        else if (color[i] == 2 && color[n - i - 1] != 2) sum += pay[color[n - i - 1]];
        else if (color[i] == 2 && color[n - i - 1] == 2) sum += 2 * minn;
        else if (color[i] != 2 && color[n - i - 1] != 2) {
            if (color[i] != color[n - i - 1]) {
                cout << -1 << endl;
                return 0;
            }
        }
    }
    cout << sum << endl;
    return 0;
}
```
