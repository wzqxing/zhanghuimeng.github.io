---
title: Leetcode 937. Reorder Log Files（排序），及周赛（110）总结
urlname: leetcode-937-reorder-log-files-and-weekly-contest-110
toc: true
date: 2018-11-11 20:02:09
updated: 2018-11-12 00:29:00
tags: [Leetcode, Leetcode Contest, alg:Sort]
---

题目来源：[https://leetcode.com/problems/reorder-log-files/description/](https://leetcode.com/problems/reorder-log-files/description/)

标记难度：Easy

提交次数：1/2

代码效率：12ms

## 题意

给定一堆log和排序方法，返回排序之后的结果。

## 分析

这次比赛的排名是528 / 3720。第一题错了一次，然后第四题没有想出正解。这次比赛比较迷的一点是，它推迟了，改成了10:30开始，然后我的电脑中间还没网了，不得不重启一次……

---

这道题的考点可以用“Custom Sort”来概括，也许考察语言特性和细节比算法本身要多。然后因为搞错了identifier具体的排序方法还WA了一次。没别的了。

## 代码

```cpp
class Solution {
private:
    
    struct Log {
        string s;
        string identifier;
        vector<string> words;
        string word;
        bool isLetter;
        int index;
        
        Log() {}
        
        Log(string s, int i) {
            this->s = s;
            this->index = i;
            stringstream lineStream(s);
            
            lineStream >> identifier;
            
            string token;
            isLetter = true;
            while (lineStream >> token)
            {
                words.push_back(token);
                word += token;
                if (isNumeric(token))
                    isLetter = false;
            }
        }
        
        bool isNumeric(string s) {
            for (char ch: s)
                if (ch < '0' || ch > '9') return false;
            return true;
        }
        
        friend bool operator < (const Log& l1, const Log& l2) {
            if (l1.isLetter != l2.isLetter) {
                if (l1.isLetter) return true;
                else return false;
            }
            if (l1.isLetter && l2.isLetter) {
                int i = 0, n = min(l1.words.size(), l2.words.size());
                for (i = 0; i < n; i++)
                    if (l1.words[i] != l2.words[i])
                        return l1.words[i] < l2.words[i];
                if (l1.words.size() != l2.words.size())
                    return l1.words.size() < l2.words.size();
                return l1.identifier < l2.identifier;
            }
            return l1.index < l2.index;
        }
    };
    
public:
    vector<string> reorderLogFiles(vector<string>& logs) {
        vector<Log> Logs;
        for (int i = 0; i < logs.size(); i++) {
            Logs.emplace_back(logs[i], i);
        }
        sort(Logs.begin(), Logs.end());
        vector<string> ans;
        for (Log log: Logs)
            ans.push_back(log.s);
        return ans;
    }
};
```
