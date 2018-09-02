---
title: Leetcode 394. Decode String（栈）
urlname: leetcode-394-decode-string
toc: true
date: 2018-09-02 20:25:12
updated: 2018-09-02 21:49:00
tags: [Leetcode, alg:Recursive, alg:Stack, alg:String]
---

题目来源：[https://leetcode.com/problems/decode-string/description/](https://leetcode.com/problems/decode-string/description/)

标记难度：Medium

提交次数：1/1

代码效率：

* 递归版本：100.00%
* 栈版本：100.00%
* 正则表达式版本：0.00%

## 题意

给定一个形如`"3[a2[c]]"`的字符串，要求将其展开为`accaccacc`格式。

## 分析

递归寻找`[]`中间的字符串，将其递归展开后，将返回的字符串重复规定的次数。大致就是这么一个思路。我觉得需要注意的细节包括：

* 注意一些不需要递归展开的部分
* 注意字符串操作中的边界

用递归的方法相当于用栈。也许用栈直接做会更直接和节省时间一些。[^javastack]

[^javastack]: [Simple Java Solution using Stack](https://leetcode.com/problems/decode-string/discuss/87534/Simple-Java-Solution-using-Stack)

另一种神奇的做法是用正则表达式，直接找出形如`cnt[str]`的字符串并展开。[^regex]我尝试用C++写了一下，发现效率比较低，但这种想法是很妙的。

[^regex]: [3 lines Python, 2 lines Ruby, regular expression](https://leetcode.com/problems/decode-string/discuss/87536/3-lines-Python-2-lines-Ruby-regular-expression/171372)

## 代码

### 递归版本

```cpp
class Solution {
private:
    bool isDigit(char x) {
        return '0' <= x && x <= '9';
    }

public:
    string decodeString(string s) {
        if (s.length() == 0)
            return "";
        int leftBrackets = 0, rightBrackets = 0, k = 0, start = -1, end = -1;
        string startStr, multStr, addStr;
        int i = 0, n = s.length();
        while (!isDigit(s[i]) && s[i] != '[' && i < n)
            i++;
        if (i >= n) return s;
        startStr = s.substr(0, i);
        while (isDigit(s[i]) && i < n) {
            k = k * 10 + s[i] - '0';
            i++;
        }
        start = i;
        while ((leftBrackets == 0 || leftBrackets != rightBrackets) && i < n) {
            if (s[i] == '[')
                leftBrackets++;
            if (s[i] == ']')
                rightBrackets++;
            i++;
        }
        end = i - 1;

        // cout << "s=" << s << " k=" << k << " startStr=" << startStr << " start=" << start << " end=" << end << endl;

        multStr = s.substr(start + 1, end - start - 1);
        addStr = s.substr(i, n - i);
        multStr = decodeString(multStr);
        addStr = decodeString(addStr);

        string str(startStr);
        for (int i = 0; i < k; i++)
            str += multStr;
        str += addStr;
        return str;
    }
};
```

### 栈版本

```cpp
class Solution {
public:
    string decodeString(string s) {
        // 把这两个栈分开的思路不错
        stack<int> cntStack;
        stack<string> strStack;
        int cnt = 0;
        string str;
        for (char ch: s) {
            if ('0' <= ch && ch <= '9') {
                cnt = cnt * 10 + ch - '0';
                // 把之前不在重复范围内的字符串入栈
                if (str.length() != 0) {
                    strStack.push(str);
                    str = "";
                }
            }
            else if (ch == '[') {
                cntStack.push(cnt);
                cnt = 0;
                strStack.push("[");  // 需要了解右括号对应的范围
            }
            else if (ch == ']') {
                // 把范围内的字符串都弹出来，可能有不止一块
                while (true) {
                    if (strStack.top() == "[") {
                        strStack.pop();
                        break;
                    }
                    str = strStack.top() + str;
                    strStack.pop();
                }
                cnt = cntStack.top();
                cntStack.pop();
                string rep;
                for (int i = 0; i < cnt; i++) rep += str;

                cnt = 0;
                str = "";
                strStack.push(rep);
            }
            else
                str += ch;
        }

        while (!strStack.empty()) {
            str = strStack.top() + str;
            strStack.pop();
        }
        return str;
    }
};
```

### 正则表达式版本

```cpp
class Solution {
public:
    string decodeString(string s) {
        smatch m;
        regex r("(\\d+)\\[([^\\[^\\]]*)\\]");
        while (regex_search(s, m, r)) {
            int cnt = stoi(m.format("$1"));
            string str = m.format("$2");
            string rep;
            for (int i = 0; i < cnt; i++)
                rep += str;
            s = m.prefix().str() + rep + m.suffix().str();
        }
        return s;
    }
};
```
