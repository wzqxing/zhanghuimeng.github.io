---
title: Leetcode 202. Happy Number（数列）
urlname: leetcode-202-happy-number
toc: true
mathjax: true
date: 2018-08-17 20:34:31
updated: 2018-08-17 22:28:31
tags: [Leetcode]
---

题目来源：[https://leetcode.com/problems/happy-number/description/](https://leetcode.com/problems/happy-number/description/)

标记难度：Easy

提交次数：3/3

代码效率：

* 普通方法：16.77%
* Floyd判圈算法：100.00%
* 42是万物的答案：16.77%

## 题意

判断一个正整数是否是[快乐数](https://zh.wikipedia.org/wiki/%E5%BF%AB%E6%A8%82%E6%95%B8)。

## 分析

这道题本身倒是很简单：直接迭代对数位平方求和，直到找到循环或者得到1即可。但是快乐数还有很多非常有趣的性质。除此之外，我们还可以应用[Floyd判圈算法](https://zh.wikipedia.org/wiki/Floyd%E5%88%A4%E5%9C%88%E7%AE%97%E6%B3%95)来解决找循环的问题。（[My solution in C( O(1) space and no magic math property involved )](https://leetcode.com/problems/happy-number/discuss/56917/My-solution-in-C%28-O%281%29-space-and-no-magic-math-property-involved-%29)）这样会稍微牺牲一点计算的常数，但会大大减少Hash所需的存储空间。我记得上个学期《现代密码学》讲碰撞攻击的时候也用到了相似的思路。

### 快乐数的性质

有两篇题解都讲到了快乐数的性质（[All you need to know about testing happy number!](https://leetcode.com/problems/happy-number/discuss/56918/All-you-need-to-know-about-testing-happy-number!)和[Explanation of why those posted algorithms are mathematically valid](https://leetcode.com/problems/happy-number/discuss/56919/Explanation-of-why-those-posted-algorithms-are-mathematically-valid)），不过并不是很全面。[这篇论文](http://math.bard.edu/student/pdfs/alison-mutter.pdf)中讲到了更多的内容。

下面令$f(n)$表示对n进行数位平方和运算得到的结果。

性质1：集合$H = {1}$是一个不动点。

证明：显然，$f(1) = 1^2 = 1$。

性质2：集合$S = {4, 16, 37, 58, 89, 145, 42, 20}$表示了一个长度为8的循环。对于任意$n \in S$，$f^m(n) \in S$。

证明：$f(4) = 4^2 = 16$，$f(16) = 1^2 + 6^2 = 37$，$f(37) = 3^2 + 7^2 = 58$，$f(58) = 5^2 + 8^2 = 89$，$f(89) = 8^2 + 9^2 = 145$，$f(145) = 1^2 + 4^2 + 5^2 = 42$，$f(42) = 4^2 + 2^2 = 20$，$f(20) = 2^2 + 0^2 = 4$。

定理：对于任意自然数$n$，总存在自然数$m$，使得$f^m(n) \in S$或$f^m(n) \in H$。这意味着一个自然数或者是快乐数，或者在经过足够多次的$f$运算后，总会落入$S$表示的循环中。

证明：对于任何自然数$n$，每一位数字的最大值都是9。当$n$为$k$位时，$f(n)$在$n$的每一位都为9时取到最大值$9^2 k$，因此$f(n) < 81k$。由于$k$位数的第一位必然不是0，因此$10^{k-1} \leq n < 10^k$。显然，$10^{k-1}$增长的速度比$81k$快得多，因此$k \geq 4$时，$81k \leq 10^{k-1}$。由$10^{k-1} \leq n$和$f(n) \leq 81k$可知，$f(n) \leq 81k \leq 10^{k-1} \leq n$，因此$k \geq 4$时$f(n) < n$，这意味着对于1000以上的数，它对应的计算结果必然比原来的数要小，函数是递减的。因此，只有$k < 4$时，$f(n)$可能会大于等于$n$。这种情况下$n$最大可能值为999，此时$f(n) = 243$。

因此我们得到了999这个界：仅当$n \leq 999$时，$f(n)$可能会大于等于$n$。但事实上我们可以把这个界缩得更小一些：显然，在$[1, 999]$的所有数中，$f(999)$是最大的，对于任意$n \in [1, 999]$，有$f(n) \leq f(999) = 243$。因此，当$243 < n \leq 999$时，$f(n) \leq 243 < n$。所以，仅当$n \leq 243$时，$f(n)$可能会大于等于$n$。

当$n > 243$时，函数是单调递减的，因此单由这样的$n$是不可能组成循环的；且必然存在自然数$m$，使得$1 \leq f^{(m)}(n) \leq 243$。下面，我们只需编程检查$[1, 243]$范围内的所有数。代码如下，运行之后检查结果，可以发现定理是正确的。

```cpp
#include <iostream>
using namespace std;
int calcHappy(int x) {
    int sum = 0;
    while (x > 0) {
        sum += (x % 10) * (x % 10);
        x /= 10;
    }
    return sum;
}
int main() {
    for (int i = 1; i <= 243; i++) {
        int x = i;
        cout << i << ", ";
        while (x != 1) {
            if (x == 4 || x == 16 || x == 37 || x == 58 || x == 89 || x == 145 || x == 42 || x == 20) {
                break;
            }
            x = calcHappy(x);
        }
        cout << x << endl;
    }
    return 0;
}
```

## 代码

### 普通方法

```cpp
class Solution {
private:
    int calcHappy(int x) {
        int sum = 0;
        while (x > 0) {
            sum += (x % 10) * (x % 10);
            x /= 10;
        }
        return sum;
    }

public:
    bool isHappy(int n) {
        if (n == 1)
            return true;
        // 对于一个int，10 * 9^2 = 810，所以这样计算肯定不会爆栈的
        bool visited[1000];
        memset(visited, 0, sizeof(visited));
        do {
            // cout << n << endl;
            n = calcHappy(n);
            if (visited[n] == true)
                return false;
            visited[n] = true;
        } while (n != 1);

        return true;
    }
};
```

### Floyd判圈算法

```cpp
class Solution {
private:
    int calcHappy(int x) {
        int sum = 0;
        while (x > 0) {
            sum += (x % 10) * (x % 10);
            x /= 10;
        }
        return sum;
    }

public:
    bool isHappy(int n) {
        if (n == 1)
            return true;
        int slow = n, fast = n;
        do {
            slow = calcHappy(slow);
            fast = calcHappy(fast);
            fast = calcHappy(fast);
        } while (slow != fast);

        return slow == 1;
    }
};
```

### 42是万物的答案

这一方法来自[StephanPochmann](https://leetcode.com/problems/happy-number/discuss/57028/42-is-the-answer)，看上去非常有趣。由于10进制快乐数一共只有一个非平凡的循环：`[4, 16, 37, 58, 89, 145, 42, 20]`，而[42](https://en.wikipedia.org/wiki/Phrases_from_The_Hitchhiker%27s_Guide_to_the_Galaxy#Answer_to_the_Ultimate_Question_of_Life.2C_the_Universe.2C_and_Everything_.2842.29)恰好在这个集合中，所以我们可以利用42做一点事情——比如在代码中只写出一个数字，42。

我尝试在C++中复制这一做法，但彻底失败了。可能是因为我对lambda表达式不够熟悉吧……

```cpp
class Solution {
private:
    int calcHappy(int x) {
        string s = to_string(x);
        vector<int> squares;
        for (int i = 0; i < s.length(); i++)
            squares.push_back(stoi(s.substr(i, 1)) * stoi(s.substr(i, 1)));
        return accumulate(squares.begin(), squares.end(), 0);
    }

public:
    bool isHappy(int n) {
        int prev;
        while (n != 42) {
            prev = n;
            n = calcHappy(n);
            if (prev == n)
                return true;
        }

        return false;
    }
};
```
