---
title: Leetcode 401. Binary Watch（枚举）
urlname: leetcode-401-binary-watch
toc: true
date: 2018-08-25 09:05:03
updated: 2018-08-25 10:05:03
tags: [Leetcode]
---

题目来源：[https://leetcode.com/problems/binary-watch/description/](https://leetcode.com/problems/binary-watch/description/)

标记难度：Easy

提交次数：1/1

代码效率：100%

## 题意

有一个二进制LED表，用6个LED灯（`1, 2, 4, 8, 16, 32`）表示分钟（0-59），4个LED灯（`1, 2, 4, 8`）表示小时（0-11）。给定共有多少个LED灯是亮的，求所有可能表示的时间。

## 分析

显然我们可以用搜索来解决这个问题：枚举小时表示中有多少个LED灯是亮的，然后找出所有可能的组合，最后拼在一起。[3ms Java Solution Using Backtracking and Idea of "Permutation and Combination"](https://leetcode.com/problems/binary-watch/discuss/88456/3ms-Java-Solution-Using-Backtracking-and-Idea-of-%22Permutation-and-Combination%22)使用的就是这一解法，非常直接。

但是由于时间表示本身的性质（就像在[Leetcode 539](/post/leetcode-539-minimum-time-difference)中说的那样，一天其实只有`24*60=1440`分钟，`24*600*60=86400`秒），我们可以写出一种更简单的枚举方法：直接枚举每天的所有时间，然后计算该时间表示中`1`的数量是否符合要求就可以了。

于是我们又可以用`__builtin_popcount`了。除此之外，在[Straight-forward 6-line c++ solution, no need to explain](https://leetcode.com/problems/binary-watch/discuss/88465/Straight-forward-6-line-c++-solution-no-need-to-explain)中，我看到了另一种计算方法：

```cpp
int num = bitset<10>(x).count();
```

>类模板`bitset`表示一个`N`位的固定大小序列。可以用标准逻辑运算符操作位集，并将它与字符串和整数相互转换。[^bitset]

[^bitset]: [std::bitset](https://zh.cppreference.com/w/cpp/utility/bitset)

这也是个不错的方法（特别是记不住`__builtin_popcount`的全名的时候），不过我猜测效率会慢一些，毕竟是STL。

## 代码

```cpp
class Solution {
public:
    vector<string> readBinaryWatch(int num) {
        if (num > 8)
            return {};

        int hours[4] = {1, 2, 4, 8};
        int minutes[6] = {1, 2, 4, 8, 16, 32};

        vector<string> times;
        for (int hour = 0; hour < 12; hour++)
            for (int minute = 0; minute < 60; minute++) {
                if (__builtin_popcount(hour) + __builtin_popcount(minute) == num) {
                    string time = to_string(hour) + ":";
                    if (minute < 10)
                        time += "0";
                    time += to_string(minute);
                    times.push_back(time);
                }
            }

        return times;
    }
};
```
