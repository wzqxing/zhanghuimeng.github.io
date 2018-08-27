---
title: Leetcode 893. Groups of Special-Equivalent Strings（map）
urlname: leetcode-893-groups-of-special-equivalent-strings
toc: true
date: 2018-08-26 23:24:39
updated: 2018-08-26 23:54:39
tags: [Leetcode, LeetcodeContest]
---

题目来源：[https://leetcode.com/problems/groups-of-special-equivalent-strings/description/](https://leetcode.com/problems/groups-of-special-equivalent-strings/description/)

标记难度：Easy

提交次数：2/3

代码效率：

* 排序：4ms
* 计数排序：36ms

## 题意

给定一系列字符串，对于每个字符串，可以随意交换模2同余的位置上的字符。如果经过交换后可以使得两个字符串相等，则称它们在同一组内。问这些字符串中共有几组。

## 分析

这是周赛（99）的第二题。这道题好像一共才做了5分钟，因为之前见过类似的题目了（[Leetcode 49](/post/leetcode-49-group-anagrams)），做法也差不多。我感觉，最好写的方法（对于C++而言）就是分别取出奇数和偶数位置上的字符，分别排序，然后组合起来，作为键值进行统计。

考虑到效率和字符串本身的特点，仍然可以用计数排序的做法[^solution]，但这对于C++来说并不好写，因为没有像Java或Python那样方便的`to_string`类函数。

[^solution]: [Java Concise Set Solution](https://leetcode.com/problems/groups-of-special-equivalent-strings/discuss/163413/Java-Concise-Set-Solution)

## 代码

### 排序

```cpp
class Solution {
public:
    int numSpecialEquivGroups(vector<string>& A) {
        // 把位置模2不同的两种串分开考虑
        // 实际上，因为只需要统计group的数量，这里用set就足够了
        map<string, int> groups;
        for (string str: A) {
            string odd;
            string even;
            for (int i = 0; i < str.length(); i++)
                if (i % 2 == 0)
                    odd += str[i];
                else
                    even += str[i];
            sort(odd.begin(), odd.end());
            sort(even.begin(), even.end());
            groups[odd + even]++;
        }
        return groups.size();
    }
};
```

### 计数排序

因为复杂的操作过程，常数太大，虽然复杂度减小了，速度反而慢了很多。

```cpp
class Solution {
private:
    string intToString(int x[], int len) {
        string str;
        for (int i = 0; i < len; i++)
            str += to_string(x[i]) + ",";
        return str;
    }

public:
    int numSpecialEquivGroups(vector<string>& A) {
        int hash[52];
        set<string> groups;
        for (string str: A) {
            memset(hash, 0, sizeof(hash));
            for (int i = 0; i < str.size(); i++) {
                hash[str[i] - 'a' + (i % 2) * 26]++;
                // 中间wa了一次，因为把26写成2了，就这样居然还过了31个点
            }
            string h = intToString(hash, 52);
            if (groups.find(h) == groups.end())
                groups.insert(h);
        }

        return groups.size();
    }
};
```
