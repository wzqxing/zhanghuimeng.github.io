---
title: Leetcode 720. Longest Word in Dictionary（枚举；Trie）
urlname: leetcode-720-longest-word-in-dictionary
toc: true
mathjax: true
date: 2018-08-27 19:22:43
updated: 2018-08-27 20:46:43
tags: [Leetcode]
---

题目来源：[https://leetcode.com/problems/longest-word-in-dictionary/description/](https://leetcode.com/problems/longest-word-in-dictionary/description/)

标记难度：Easy

提交次数：2/2

代码效率：

* 枚举：61.98%
* Trie：30.09%

## 题意

给定一系列字符串，找出满足这一条件的最长且字典序最小的字符串：这个字符串的所有前缀都包含在字符串集中。

## 分析

因为字符串数量只有1000，长度只有30，所以完全可以采用枚举的方法：先把所有字符串排序（这样可以保证每个字符串的前缀都排在它前面），按顺序把满足要求的字符串加入到一个`set`中；对于每个字符串，检查它的`substr(1, length-1)`子串是否包含在`set`中，如果是则满足要求。（所以里面也含有一些动态规划的思想）这一做法和[题解](https://leetcode.com/problems/longest-word-in-dictionary/solution/)中给出的做法相比有些差异，更类似于[^javasol]，复杂度（假设在HashSet中插入和查询字符串的时间正比于字符串长度）大约是$O(n * \log{(n)} + \sum w_i)$。

[^javasol]: [Java/C++ Clean Code](https://leetcode.com/problems/longest-word-in-dictionary/discuss/109114/JavaC++-Clean-Code)

如果数据量更大的话，更好的方法是使用Trie。把所有的字符串都插入到一棵Trie中，然后通过DFS或BFS找到最长的连续路径。这样复杂度只有$O(\sum w_i)$。

## 代码

### 枚举

```cpp
class Solution {
public:
    string longestWord(vector<string>& words) {
        set<string> okToBuild;
        string longest;
        sort(words.begin(), words.end());
        for (string word: words) {
            if (word.length() == 1) {
                okToBuild.insert(word);
                // 考虑到经过了排序，后一个判断条件其实是不必要的
                if (word.length() > longest.length() || (word.length() == longest.length() && word < longest))
                    longest = word;
            }
            else {
                if (okToBuild.find(word.substr(0, word.length() - 1)) != okToBuild.end()) {
                    okToBuild.insert(word);
                    if (word.length() > longest.length() || (word.length() == longest.length() && word < longest))
                        longest = word;
                }
            }
        }

        return longest;
    }
};
```

### Trie

```cpp
class Solution {
private:
    struct TrieNode {
        string str;
        TrieNode* next[26];

        TrieNode() {
            memset(next, 0, sizeof(next));
        }

        TrieNode(string s): str(s) {
            memset(next, 0, sizeof(next));
        }
    };

    void insert(TrieNode* root, string str) {
        TrieNode* cur = root;
        for (int i = 0; i < str.length(); i++) {
            char ch = str[i];
            int index = ch - 'a';
            if (cur->next[index] == NULL) {
                cur->next[index] = new TrieNode();
            }
            cur = cur->next[index];
        }
        cur->str = str;
    }

    string longest;

    void dfs(TrieNode* cur, int depth) {
        if (cur == NULL)
            return;
        if (depth > longest.length() && cur->str.length() == depth)
            longest = cur->str;
        for (int i = 0; i < 26; i++)
            if (cur->next[i] != NULL && cur->next[i]->str.length() > 0)
                dfs(cur->next[i], depth+1);
    }

public:
    string longestWord(vector<string>& words) {
        TrieNode* root = new TrieNode();
        for (string word: words)
            insert(root, word);
        dfs(root, 0);
        return longest;
    }
};
```
