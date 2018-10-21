---
title: Leetcode 925. Long Pressed Name（字符串），及周赛（107）总结
urlname: leetcode-925-long-pressed-name-and-weekly-contest-107
toc: true
date: 2018-10-21 14:48:23
updated: 2018-10-21 15:15:23
tags: [Leetcode, Leetcode Contest, alg:String]
---

题目来源：[https://leetcode.com/problems/long-pressed-name/description/](https://leetcode.com/problems/long-pressed-name/description/)

标记难度：Easy

提交次数：2/2

代码效率：

* 暴力的做法：4ms
* 不太暴力的做法：0ms

## 题意

有一个叫`name`的字符串，你在把它录入电脑的时候可能会把某个字母连续重复若干遍（相当于你“长按”了那个按键），问`typed`串是否有可能是这样形成的。

## 分析

应当庆祝一下，这次比赛中第一次把所有题都做出来了（当然也是因为题水）。最后得到的排名是128 / 3209，充分说明是题太水了。而且我第三题因为MLE的罚时太多了。不过今天不知道Leetcode是不是出了什么问题，现在官方题解仍未出现。

（虽然我知道我还是很菜，但是我很久以前就写过了，要接受自己的菜然后努力去提升，别人怎么样我是不管的。）

---

我比赛的时候的做法就是把两个字符串都做了一个连续字符的压缩，然后对压缩之后的结果进行比较。这个方法比较暴力。当然，也可以不那么暴力，也就是不把这个过程显式地写出来。[^sol]

[^sol]: [Leetcode 925 Solution by votrubac - C++ 2 lines accepted and 5 lines accurate](https://leetcode.com/problems/long-pressed-name/discuss/183929/C++-2-lines-accepted-and-5-lines-accurate)

## 代码

### 暴力的做法

```cpp
class Solution {
public:
    bool isLongPressedName(string name, string typed) {
        vector<pair<char, int>> press1, press2;
        for (char ch: name) {
            if (press1.size() == 0 || press1.back().first != ch)
                press1.emplace_back(ch, 1);
            else
                press1.back().second++;
        }
        for (char ch: typed) {
            if (press2.size() == 0 || press2.back().first != ch)
                press2.emplace_back(ch, 1);
            else
                press2.back().second++;
        }
        if (press1.size() != press2.size()) return false;
        for (int i = 0; i < press1.size(); i++)
            if (press1[i] > press2[i])
                return false;
        return true;
    }
};
```

### 不太暴力的做法

参考了[^sol]中的代码。

```cpp
class Solution {
public:
    bool isLongPressedName(string name, string typed) {
        int n = name.size(), m = typed.size();
        if (n > m) return false;
        int i = 0, j = 0;
        while (i < n || j < m) {
            if (i < n && typed[j] == name[i]) i++, j++;
            else if (i > 0 && typed[j] == name[i - 1]) j++;
            else break;
        }
        if (i == n && j == m) return true;
        return false;
    }
};
```
