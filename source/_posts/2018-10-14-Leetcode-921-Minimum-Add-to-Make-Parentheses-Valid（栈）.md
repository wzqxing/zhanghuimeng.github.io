---
title: Leetcode 921. Minimum Add to Make Parentheses Valid（栈），及周赛（106）总结
urlname: leetcode-921-minimum-add-to-make-parentheses-valid-and-weekly-contest-106
toc: true
date: 2018-10-14 15:23:44
updated: 2018-10-14 16:00:00
tags: [Leetcode, Leetcode Contest, Alg:Stack]
---

题目来源：[https://leetcode.com/problems/minimum-add-to-make-parentheses-valid/description/](https://leetcode.com/problems/minimum-add-to-make-parentheses-valid/description/)

标记难度：Easy

提交次数：3/3

代码效率：

* 栈：4ms
* 计数器：0ms

## 题意

给定一个只由括号组成的字符串，问如果需要在字符串中添加括号使它变成合法的字符串，最少需要添加多少个括号？

## 分析

这次比赛的排名是632 / 2700。（人好少。）排名很低的原因一是第三题罚时太多了（靠WA来debug毕竟不可取，还是要好好考虑corner case？），二是这次第四题非常简单，但我却不仅看错了题，暴力也没写对……

---

这道题很简单，但还稍微有点意思。在何种情况下需要添加括号？如何打印出一种添加括号的方法？

我的第一反应是直接统计左括号和右括号各自的数量，但这么做显然不可取，反例：`))((`。那么不妨采用我们一般对括号表达式进行思考的方式——栈——来考虑这个问题。在括号入栈的过程中，一个右括号总是和栈中最顶端的还没有配对的左括号配对；如果当前栈中是空的，则这个右括号没有被配对——可以直接在它的左边加上一个左括号；如果最后栈中还剩下一些左括号，则直接在字符串的右侧加上对应数量的右括号。

这个问题也可以简化到不用栈来解决。直接用一个计数器维护当前栈中尚未配对的左括号数量即可。[^java]

[^java]: [rock's solution - \[Java\] two one pass 7 liners - space O(n) and O(1), respectively](https://leetcode.com/problems/minimum-add-to-make-parentheses-valid/discuss/181086/Java-two-one-pass-7-liners-space-O%28n%29-and-O%281%29-respectively)

## 代码

### 栈

```cpp
class Solution {
public:
    int minAddToMakeValid(string S) {
        stack<char> s;
        for (char ch: S) {
            if (!s.empty() && s.top() == '(' && ch == ')')
                s.pop();
            else
                s.push(ch);
        }
        return s.size();
    }
};
```

### 实际地添加括号

写了一个实际添加括号的程序，大概是对的吧。

```cpp
class Solution {
public:
    int minAddToMakeValid(string S) {
        stack<int> s;
        vector<int> toInsBefore;
        int ans = 0;
        for (int i = 0; i < S.size(); i++) {
            if (S[i] == '(')
                s.push(i);
            else {
                if (s.empty()) {
                    // 记录未配对右括号的位置
                    toInsBefore.push_back(i);
                    ans++;
                }
                else
                    s.pop();
            }
        }
        for (int i = toInsBefore.size() - 1; i >= 0; i--)
            S = S.substr(0, toInsBefore[i]) + '(' + S.substr(toInsBefore[i], S.size() - toInsBefore[i]);
        for (int i = 0; i < s.size(); i++) {
            ans++;
            S += ')';
        }
        cout << S << endl;
        return ans;
    }
};
```

### 计数器

```cpp
class Solution {
public:
    int minAddToMakeValid(string S) {
        int counter = 0, ans = 0;
        for (char ch: S) {
            if (ch == '(') counter++;
            else {
                if (counter > 0) counter--;
                else ans++;
            }
        }
        return ans + counter;
    }
};
```
