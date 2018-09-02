---
title: Leetcode 168. Excel Sheet Column Title（进制转换）
urlname: leetcode-168-excel-sheet-column-title
toc: true
date: 2018-09-01 10:30:59
updated: 2018-09-01 10:40:00
tags: [Leetcode, alg:Math]
---

题目来源：[https://leetcode.com/problems/excel-sheet-column-title/description/](https://leetcode.com/problems/excel-sheet-column-title/description/)

标记难度：Easy

提交次数：1/1

代码效率：100.00%

## 题意

把数字转换成Excel的列标题（形如A，AB，AZ……）。

## 分析

这道题刚好和[Leetcode 171](/post/leetcode-171-excel-sheet-column-number)是相反的。就像之前分析的那样，这并不是一个严格的进制转换问题，所以也不能直接按照普通进制转换的方法去做。

于是我就把刚才的递推式拿过来取了个逆：

```
number = number*26 + ch-'A' + 1
=>
ch = (number - 1) % 26 + 'A'
```

另一种方法[^solution]是进行观察：

```
A   1     AA    26+ 1     BA  2×26+ 1     ...     ZA  26×26+ 1     AAA  1×26²+1×26+ 1
B   2     AB    26+ 2     BB  2×26+ 2     ...     ZB  26×26+ 2     AAB  1×26²+1×26+ 2
.   .     ..    .....     ..  .......     ...     ..  ........     ...  .............
.   .     ..    .....     ..  .......     ...     ..  ........     ...  .............
.   .     ..    .....     ..  .......     ...     ..  ........     ...  .............
Z  26     AZ    26+26     BZ  2×26+26     ...     ZZ  26×26+26     AAZ  1×26²+1×26+26
```

通过观察可以发现，由于数字代表的值的范围是`1-26`，所以直接模26会导致Z消失；因此可以先`-1`再模26，然后就可以把得到的`0-25`的数字映射到`A-Z`的范围内了。

[^solution]: [Python solution with explanation](https://leetcode.com/problems/excel-sheet-column-title/discuss/51404/Python-solution-with-explanation)

## 代码

```cpp
class Solution {
public:
    string convertToTitle(int n) {
        if (n <= 0)
            return "";
        string ans;
        while (n > 0) {
            n--;
            ans = (char) (n % 26 + 'A') + ans;
            n /= 26;
        }
        return ans;
    }
};
```
