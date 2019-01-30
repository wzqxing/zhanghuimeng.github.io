---
title: 'USACO 2.3.1: Longest Prefix（DP）'
urlname: usaco-2-3-1-longest-prefix
toc: true
date: 2019-01-30 20:25:13
updated: 2019-01-30 20:25:13
tags: [USACO, alg:Dynamic Programming]
---

## 题意

见[洛谷 P1470 最长前缀 Longest Prefix](https://www.luogu.org/problemnew/show/P1470)。

给定若干个字符串和另一个字符串`S`，问`S`的能由上述字符串组成的最长的前缀的长度。

## 分析

普通的动态规划。刚才好像刚刚在Leetcode上做了一道很相似的题：[139. Word Break](https://leetcode.com/problems/word-break/description/)

一种转移方法是从前转移：对于当前的字符串结尾，枚举所有前缀，找到能够和它匹配的，判断减去匹配部分之后的字符串能否被构成。

另一种转移方法则是向后转移：对于当前（可行的）字符串末尾，枚举所有前缀，找到能够和后面的字符串匹配的，然后记减去前缀后的字符串为可行。这种方法可以利用Trie来进行优化，比前一种更快。

## 代码

### 普通的DP

```cpp
/*
ID: zhanghu15
TASK: prefix
LANG: C++14
*/

#include <iostream>
#include <fstream>
#include <sstream>
#include <vector>
#include <algorithm>
#include <cstring>
#include <cmath>

using namespace std;
typedef long long int LL;

vector<string> prefix;

bool ok[200000];

int main() {
    ofstream fout("prefix.out");
    ifstream fin("prefix.in");

    string p;
    while (fin >> p) {
        if (p == ".") break;
        prefix.push_back(p);
    }

    string s;
    while (fin >> p)
        s += p;
    
    int n = s.length(), m = prefix.size();
    int ans = 0;
    // ok[i]：以i为结尾的字符串是否可行
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            if (prefix[j].size() - 1 > i) continue;
            if (s.substr(i - prefix[j].size() + 1, prefix[j].size()) != prefix[j])
                continue;
            if (i == prefix[j].size() - 1 || ok[i - prefix[j].size()]) {
                ok[i] = true;
                break;
            }
        }
        ans = ok[i] ? i + 1 : ans;
    }
    fout << ans << endl;
    return 0;
}
```

### 普通的Trie

```cpp
/*
ID: zhanghu15
TASK: prefix
LANG: C++14
*/

#include <iostream>
#include <fstream>
#include <sstream>
#include <vector>
#include <algorithm>
#include <cstring>
#include <cmath>

using namespace std;
typedef long long int LL;

vector<string> prefix;

bool f[200005];

struct TrieNode {
    TrieNode* ch[26];
    bool isEnd;
    TrieNode() {
        memset(ch, 0, sizeof(ch));
        isEnd = false;
    }
};

int main() {
    ofstream fout("prefix.out");
    ifstream fin("prefix.in");

    string pre;
    TrieNode* root = new TrieNode();
    while (fin >> pre) {
        if (pre == ".") break;
        prefix.push_back(pre);
        TrieNode* p = root;
        for (char c: pre) {
            if (p->ch[c - 'A'] == NULL)
                p->ch[c - 'A'] = new TrieNode();
            p = p->ch[c - 'A'];
        }
        p->isEnd = true;
    }

    string s, t;
    while (fin >> t)
        s += t;

    int n = s.length();
    int ans = 0;
    // f[i]：以i结尾的字符串是否可行
    for (int i = 0; i < n; i++) {
        if (i != 0 && !f[i - 1]) continue;
        TrieNode* p = root;
        for (int j = i; j < n; j++) {
            p = p->ch[s[j] - 'A'];
            if (p == NULL) break;
            if (p->isEnd) f[j] = true;
        }
    }
    for (int i = 0; i < n; i++)
        if (f[i])
            ans = max(ans, i + 1);
    fout << ans << endl;
    return 0;
}
```