---
title: Leetcode 936. Stamping The Sequence（贪心）
urlname: leetcode-936-stamping-the-sequence
toc: true
date: 2018-11-04 23:07:59
updated: 2018-11-04 23:07:59
tags: [Leetcode, Leetcode Contest, alg:String, alg:Greedy, alg:Depth-first Search]
---

题目来源：[https://leetcode.com/problems/stamping-the-sequence/description/](https://leetcode.com/problems/stamping-the-sequence/description/)

标记难度：Hard

提交次数：1/1

代码效率：36ms

## 题意

有一个stamp序列，可以通过不断使它覆盖一个序列的某一部分构造新的序列。问某个序列能否以这种方式构造。

## 分析

比赛的时候我觉得这道题应该用Trie之类的方法来做。没想到正解是贪心……？

---

题目里限定的是，最多可以进行`10 * target.length`次stamp，但是事实上每个位置最多只需要进行一次stamp（如果stamp两次，则后来的stamp会彻底覆盖前一次），因此最多需要的stamp次数是`source.length - target.length`。

一种方法是贪心：每次尝试unstamp掉一块序列，直到整个序列被unstamp完为止。[^zym3008]不过我不知道怎么证明贪心的正确性……

[^zym3008]: [Leetcode 936 Solution by zym3008 - \[Python\] Reverse + Greedy O(N^2M)  (w/ Explanation)](https://leetcode.com/problems/stamping-the-sequence/discuss/189314/Python-Reverse-+-Greedy-O%28N2M%29-%28w-Explanation%29)

另一种方法来自lee215，因为他现在转移到简书写中文题解了，所以我就不再在这里赘述了，不过的确是一种很好的做法。[^lee]

[^lee]: [Leetcode 936 Stamping The Sequence - 题解](https://www.jianshu.com/p/cb4cbd11511b)

## 代码

```cpp
class Solution {
    bool checkStamp(int index, const string& stamp, const string& target) {
        for (int i = 0; i < stamp.size(); i++) {
            if (index + i > target.size()) return false;
            if (stamp[i] != target[index + i] && target[index + i] != '?') return false;
        }
        return true;
    }

    bool checkIsOk(const string& target) {
        for (char ch: target)
            if (ch != '?')
                return false;
        return true;
    }

public:
    vector<int> movesToStamp(string stamp, string target) {
        vector<int> ops;
        int N = target.size(), M = stamp.size();
        int cnt = 0;
        bool stamped[N - M + 1];
        memset(stamped, 0, sizeof(stamped));
        bool found, ok;
        while (cnt <= N - M) {
            found = false, ok = false;
            for (int i = 0; i <= N - M; i++)
                if (!stamped[i] && checkStamp(i, stamp, target)) {
                    for (int j = 0; j < M; j++) target[i + j] = '?';
                    stamped[i] = true;
                    ops.push_back(i);
                    cnt++;
                    found = true;
                    if (checkIsOk(target)) {
                        ok = true;
                        break;
                    }
                }
            if (ok) break;
            if (!found) break;
        }
        if (ok) {
            reverse(ops.begin(), ops.end());
            return ops;
        }
        else
            return {};
    }
};
```
