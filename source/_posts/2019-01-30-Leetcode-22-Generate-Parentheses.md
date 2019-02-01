---
title: Leetcode 22. Generate Parentheses
urlname: leetcode-22-generate-parentheses
toc: true
date: 2019-01-30 16:50:27
updated: 2019-02-01 14:41:00
tags: [Leetcode, alg:Backtracking]
---

题目来源：[https://leetcode.com/problems/generate-parentheses/description/](https://leetcode.com/problems/generate-parentheses/description/)

标记难度：Medium

提交次数：2/2

代码效率：

* DFS：100.00%（0ms）
* DP：100.00%（0ms）

## 题意

生成所有括号对数为`n`的合法小括号组合。

## 分析

我最开始以为用普通的迭代就可以解决（每次在已经生成的组合之后加一对括号或者套上一对括号），后来发觉好像不太对，所以就改为用DFS（或者说回溯）：记录已经添加的左括号和右括号的数量，然后加上左括号，或者在右括号总数不超过左括号总数的前提下加上右括号。这样就可以不重不漏地生成所有结果了。

另一种比较有趣的方法类似于递推（大概也更容易计算数量）：对于每个合法括号序列，考虑和左侧第一个左括号配对的右括号的位置。这对括号内部一定是一个合法括号序列，后面跟着的也是一个合法括号序列。然后枚举就可以了。[^sln]

这样肯定可以推导出合法括号序列的数量的……好像就是卡特兰数。[Leetcode 894. All Possible Full Binary Trees](/post/leetcode-894-all-possible-full-binary-trees)的思路和这道题很类似，而且确实是卡特兰数……

[^sln]: [https://leetcode.com/problems/generate-parentheses/solution/](Leetcode Official Solution for 22. Generate Parentheses)

## 代码

### DFS

```cpp
class Solution {
private:
    void dfs(int left, int right, string s, vector<string>& ans, int& n) {
        if (left == n && right == n) {
            ans.push_back(s);
            return;
        }
        if (left < n) dfs(left + 1, right, s + '(', ans, n);
        if (right < left) dfs(left, right + 1, s + ')', ans, n);
    }
    
public:
    vector<string> generateParenthesis(int n) {
        if (n == 0) return {};
        vector<string> ans;
        dfs(0, 0, "", ans, n);
        return ans;
    }
};
```

### DP

```cpp
class Solution {
private:
    map<int, vector<string>> pMap;
    
public:
    vector<string> generateParenthesis(int n) {
        if (pMap.find(n) != pMap.end()) return pMap[n];
        if (n == 0) {
            pMap[n] = {""};
            return pMap[n];
        }
        vector<string> a;
        for (int i = 0; i <= n - 1; i += 1) {
            vector<string> in = generateParenthesis(i);
            vector<string> back = generateParenthesis(n - i - 1);
            for (string x: in)
                for (string y: back)
                    a.push_back("(" + x + ")" + y);
        }
        pMap[n] = a;
        return a;
    }
};
```
