---
title: Leetcode 906. Super Palindromes（数学）
urlname: leetcode-906-super-palindromes
toc: true
date: 2018-09-16 21:42:38
updated: 2018-09-19 18:11:00
tags: [Leetcode, Leetcode Contest, alg:Math]
---

题目来源：[https://leetcode.com/problems/super-palindromes/description/](https://leetcode.com/problems/super-palindromes/description/)

标记难度：Hard

提交次数：1/1

代码效率：

* 穷举法：552ms
* 构造法：4ms（82.91%）

## 题意

统计`[L, R]`区间内“超级回文数”的个数。“超级回文数”的定义是，它是一个回文数，且也是一个回文数的平方。保证`L <= R <= 10^18`。

## 分析

### 穷举法

显然，一个事实是，我们可以通过穷举`[1, 10^9]`内的回文数并判断其平方是否为回文数，来穷举`[1, 10^18]`内的超级回文数。而且，通过回文数的特性，我们可以通过穷举`[1, 10^4.5]`内的数并把它反过来拼成一个回文数（当然有两种拼法）来穷举`[1, 10^9]`内的回文数。所以我们得到了一个`O(U^0.25) (U = upper limit)`的算法。[^solution]

[^solution]: [Leetcode 905 Solution](https://leetcode.com/problems/super-palindromes/solution/)

### 构造法

很快就可以发现，超级回文数是相当稀疏的，在`[1, 10^18]`的范围内，总共只有70个。（[OEIS A002779](https://oeis.org/A002779)中收录了平方回文数的数列，但似乎还并没有“超级回文数”。）所以这就导向了一种采用构造法的可能性，可以大大降低时间复杂度。

```
1, 4, 9, 121, 484, 10201, 12321, 14641, 40804, 44944, 1002001, 1234321, 4008004,
100020001, 102030201, 104060401, 121242121, 123454321, 125686521, 400080004,
404090404, 10000200001, 10221412201, 12102420121, 12345654321, 40000800004,
1000002000001, 1002003002001, 1004006004001, 1020304030201, 1022325232201,
1024348434201, 1210024200121, 1212225222121, 1214428244121, 1232346432321,
1234567654321, 4000008000004, 4004009004004, 100000020000001, 100220141022001,
102012040210201, 102234363432201, 121000242000121, 121242363242121, 123212464212321,
123456787654321, 400000080000004, 10000000200000001, 10002000300020001,
10004000600040001, 10020210401202001, 10022212521222001, 10024214841242001,
10201020402010201, 10203040504030201, 10205060806050201, 10221432623412201,
10223454745432201, 12100002420000121, 12102202520220121, 12104402820440121,
12122232623222121, 12124434743442121, 12321024642012321, 12323244744232321,
12343456865434321, 12345678987654321, 40000000800000004, 40004000900040004
```

从上面的列表中可以发现，所有超级回文数的长度都是奇数。这并不是巧合。证明如下：假设`A = a^2`，且`A`和`a`都是回文数。显然，只有在`a * a`时首位发生进位的情况下，`A`的长度才有可能变成偶数。为了使首位发生进位，需要`a`的首位`>= 3`。

假定`a = a[n-1] a[n-2] a[n-3] ... a[0]`，则`a * a`的竖式可以表示为：

```
                                                  a[n-1]   a[n-2]   a[n-3] ...
                                                  a[n-1]   a[n-2]   a[n-3] ...
 ×
--------------------------------------------------
                  a[n-1]*a[n-2]   a[n-2]*a[n-3] ...
  a[n-1]*a[n-1]   a[n-1]*a[n-2] ...
```

当`a[n-1] = 2`时，即使`[n-2] = 9`，`2*a[n-1]*a[n-2]`加上进位最大只可能为45，计算出的第一位最大为8，不会发生进位。所以要求`a[n-1] >= 3`。

下面可以根据`a[n-1]`分类讨论：

* 当`a[n-1] = 3`时，`A`的前两位必在10和15之间；但由于`a`是回文数，因此`a`的末位也为3，此时`A`的末位必为9，所以`A`不是回文数
* 当`a[n-1] = 4`时，`A`的前两位必在16和24之间；但由于`a`是回文数，因此`a`的末位也为4，此时`A`的末位必为6，所以`A`不是回文数
* 当`a[n-1] = 5`时，`A`的前两位必在25和35之间；但由于`a`是回文数，因此`a`的末位也为5，此时`A`的末位必为5，所以`A`不是回文数
* 当`a[n-1] = 6`时，`A`的前两位必在36和48之间；但由于`a`是回文数，因此`a`的末位也为6，此时`A`的末位必为6，所以`A`不是回文数
* 当`a[n-1] = 7`时，`A`的前两位必在49和63之间；但由于`a`是回文数，因此`a`的末位也为7，此时`A`的末位必为9，所以`A`不是回文数
* 当`a[n-1] = 8`时，`A`的前两位必在64和80之间；但由于`a`是回文数，因此`a`的末位也为8，此时`A`的末位必为4，所以`A`不是回文数
* 当`a[n-1] = 9`时，`A`的前两位必在81和99之间；但由于`a`是回文数，因此`a`的末位也为9，此时`A`的末位必为1，所以`A`不是回文数

假定`a`的长度为`n`，则`A`的长度必为`2*n - 1`。此时，由于`a * a`的首位（即使加上进位也）不会进位，说明`a * a`的末位也必然不会进位（因为`a`是回文数）；所以`A`的倒数第二位没有被进位；又由于`A`是回文数，所以`A`的正数第二位也没有被进位。以此类推，可以用数学归纳法证明，回文数`a * a`能产生超级回文数，当且仅当`a * a`的过程中没有发生过任何进位。[^explanation]

[^explanation]: [Explanation For the Math behind Generating All Super Palindromes](https://leetcode.com/problems/super-palindromes/discuss/170792/Python-O%2845%29-solution.-No-cheating.-Generate-all-super-palindromes-directly./176657)

此时，我们可以开始尝试枚举`<= 10^9`且不会发生进位的所有回文数。由于要求每一位都不能进位，显然回文数的数字只能包括`{0, 1, 2, 3}`。所以我们仍然可以采取类似于之前的枚举的方法：枚举回文数的一半（最大5位），然后扩展出回文数的下一半。而且，在这一情况下，我们可以尝试直接推导乘法应有的计算结果。

对于扩展出奇数长度的回文数的情形：

```
n = 9,
a = a[8] a[7] a[6] a[5] a[4] a[3] a[2] a[1] a[0]
  = a[0] a[1] a[2] a[3] a[4] a[3] a[2] a[1] a[0]

A[0] = a[0]^2
A[1] = 2*a[0]*a[1]
A[2] = 2*a[0]*a[2] + a[1]^2
A[3] = 2*a[0]*a[3] + 2*a[1]*a[2]
A[4] = 2*a[0]*a[4] + 2*a[1]*a[3] + a[2]^2
A[5] = 2*a[0]*a[3] + 2*a[1]*a[4] + 2*a[2]*a[3]
A[6] = 2*a[0]*a[2] + 2*a[1]*a[4] + 2*a[2]*a[4] + a[3]^2
A[7] = 2*a[0]*a[1] + 2*a[1]*a[3] + 2*a[2]*a[3] + 2*a[3]*a[4]
A[8] = 2*a[0]^2 + 2*a[1]^2 + 2*a[2]^2 + 2*a[3]^2 + a[4]^2
...
```

显然`A[8]`是其中最大的数位。

对于扩展出偶数长度的回文数的情形：

```
n = 10,
a = a[9] a[8] a[7] a[6] a[5] a[4] a[3] a[2] a[1] a[0]
  = a[0] a[1] a[2] a[3] a[4] a[4] a[3] a[2] a[1] a[0]

A[0] = a[0]^2
A[1] = 2*a[0]*a[1]
A[2] = 2*a[0]*a[2] + a[1]^2
A[3] = 2*a[0]*a[3] + 2*a[1]*a[2]
A[4] = 2*a[0]*a[4] + 2*a[1]*a[3] + a[2]^2
A[5] = 2*a[0]*a[5] + 2*a[1]*a[4] + 2*a[2]*a[3]
A[6] = 2*a[0]*a[3] + 2*a[1]*a[4] + 2*a[2]*a[4] + a[3]^2
A[7] = 2*a[0]*a[2] + 2*a[1]*a[3] + 2*a[2]*a[4] + 2*a[3]*a[4]
A[8] = 2*a[0]*a[1] + 2*a[1]*a[2] + 2*a[2]*a[3] + 2*a[3]*a[4] + a[4]^2
A[9] = 2*a[0]*a[0] + 2*a[1]*a[1] + 2*a[2]*a[2] + 2*a[3]*a[3] + 2*a[4]*a[4]
...
```

显然`A[9]`是其中最大的数位。

这样我们就可以直接判断这个最大的数位是否会发生进位了。

然后复杂度分析我实在没看懂……[^generation]

[^generation]: [Python O(4^5) solution. No cheating. Generate all super-palindromes directly. ](https://leetcode.com/problems/super-palindromes/discuss/170792/Python-O%2845%29-solution.-No-cheating.-Generate-all-super-palindromes-directly.)

## 代码

### 穷举法

```cpp
class Solution {
private:
    bool isPalindrome(long long x) {
        string str = to_string(x);
        int n = str.length();
        for (int i = 0; i < n - i - 1; i++)
            if (str[i] != str[n - i - 1])
                return false;
        return true;
    }

    long long stol(string str) {
        long long x = 0;
        for (char ch: str)
            x = x * 10 + ch - '0';
        return x;
    }

public:
    int superpalindromesInRange(string L, string R) {
        int ans = 0;
        long long l = stol(L), r = stol(R);
        // 31623约为10^4.5
        for (long long i = 1; i <= 31623; i++) {
            // 按拼成的回文数的长度的奇偶性分出的两种算法
            for (int j = 0; j <= 1; j++) {
                string str = to_string(i);
                int n = str.length();
                for (int i = n - j - 1; i >= 0; i--)
                    str += str[i];
                long long p1 = stol(str);
                long long p2 = p1 * p1;

                if (l <= p2 && p2 <= r && isPalindrome(p2)) {
                    ans++;
                    // cout << p1 << ' ' << p2 << endl;
                }
            }
        }
        return ans;
    }
};
```

### 构造法

```cpp
class Solution {
    bool isPalindrome(long long x) {
        string str = to_string(x);
        int n = str.length();
        for (int i = 0; i < n - i - 1; i++)
            if (str[i] != str[n - i - 1])
                return false;
        return true;
    }

    long long stol(string str) {
        long long x = 0;
        for (char ch: str)
            x = x * 10 + ch - '0';
        return x;
    }

public:
    int superpalindromesInRange(string L, string R) {
        int sum = 0;
        long long l = stol(L), r = stol(R);
        for (int a0 = 0; a0 <= 3; a0++) {
            for (int a1 = 0; a1 <= 3; a1++) {
                for (int a2 = 0; a2 <= 3; a2++) {
                    for (int a3 = 0; a3 <= 3; a3++) {
                        for (int a4 = 0; a4 <= 3; a4++) {
                            // 回文数a长度为奇数
                            if (2*a0*a0 + 2*a1*a1 + 2*a2*a2 + 2*a3*a3 + a4*a4 < 10) {
                                string str = to_string(a0) + to_string(a1) + to_string(a2) + to_string(a3) + to_string(a4);
                                int left = stoi(str);
                                if (left != 0) {
                                    // 这一转换是为了消除前导0
                                    string numStr = str = to_string(left);
                                    reverse(numStr.begin(), numStr.end());
                                    numStr = str.substr(0, str.length() - 1) + numStr;
                                    long long num = stol(numStr);
                                    num = num * num;
                                    if (l <= num && num <= r && isPalindrome(num))
                                        sum++;
                                }
                            }
                            // 回文数a长度为偶数
                            if (2*a0*a0 + 2*a1*a1 + 2*a2*a2 + 2*a3*a3 + 2*a4*a4 < 10) {
                                string str = to_string(a0) + to_string(a1) + to_string(a2) + to_string(a3) + to_string(a4);
                                int left = stoi(str);
                                if (left != 0) {
                                    string numStr = str = to_string(left);
                                    reverse(numStr.begin(), numStr.end());
                                    numStr = str + numStr;
                                    long long num = stol(numStr);
                                    num = num * num;
                                    if (l <= num && num <= r && isPalindrome(num))
                                        sum++;
                                }
                            }
                        }
                    }
                }
            }
        }
        return sum;
    }
};
```
