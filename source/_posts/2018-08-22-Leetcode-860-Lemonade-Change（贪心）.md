---
title: Leetcode 860. Lemonade Change（贪心）
urlname: leetcode-860-lemonade-change
toc: true
mathjax: true
date: 2018-08-22 00:40:44
updated: 2018-08-22 09:25:00
tags: [Leetcode]
---

题目来源：[https://leetcode.com/problems/lemonade-change/description/](https://leetcode.com/problems/lemonade-change/description/)

标记难度：Easy

提交次数：1/1

代码效率：15.43%

## 题意

你在卖一种5元的柠檬水，顾客会分别用5元、10元和20元的纸币购买，你在开始的时候没有零钱。给定顾客使用的纸币类型顺序，问你能否顺利完成找零过程。

## 分析

看起来是道水题，模拟就行，不过中间有一个选择的问题：当顾客使用20元纸币的时候，你应该找3张5元的纸币，还是找1张5元和1张10元呢？直觉告诉我们，应该尽量找1张5元和1张10元的，因为5元纸币的用途更广，而10元纸币除了在收到20元时找零，其他时候是花不出去的。

我在考虑如何形式化的证明这一问题。比如说，用stay-ahead方法。

把原问题的目标扩大一下，优化目标变为“尽可能的服务最多的顾客”。记我们的算法给出的解（给每个顾客找零的纸币）为$A = \\{a_1, a_2, ..., a_k \\}$，最优解（之一）为$O = \\{ o_1, o_2, ..., o_m \\}$。令$f(i)$表示按顺序服务完第$i$个顾客后剩余的每种纸币的数量。由于我们的算法中会尽可能的节省5元纸币，所以对于任意$i$，都满足$f_{A5}(i) \geq f_{O5}(i)$，$f_{A10}(i) \leq f_{O10}(i)$。因为总钱数不变，且只能用5元和10元纸币找零，所以这两种零钱的总数是一定的，$5f_{A5}(i) + 10f_{A10}(i) = 5f_{O5}(i) + 10f_{O10}(i)$。

下面证明$A$必然是一个最优算法。若$A$不是最优的，则必然$k < m$，也就是$A$无法为第$k + 1$位顾客找零，但$O$可以。若第$k + 1$位顾客使用的是5元或10元纸币，由于$f_{A5}(k + 1) \geq f_{O5}(k + 1)$，只要$O$满足要求，$A$必然满足要求，所以这显然是不可能的。若该顾客使用的是20元纸币，由于$O$可以为他找零，说明$f_{O5}(k + 1) \geq 3$，或$f_{O5}(k + 1) \geq 1, f_{O10}(k + 1) \geq 1$。如果$f_{O5}(k + 1) \geq 3$，和刚才的情况相同，$A$必然也满足要求。如果$f_{O5}(k + 1) \geq 1, f_{O10}(k + 1) \geq 1$，由于两种算法剩余的5元和10元纸币的总钱数必然相等，必有$f_{A5}(k + 1) \geq 3$或$f_{A5}(k + 1) \geq 1, f_{A10}(k + 1) \geq 1$，$A$仍然可以找零，推出矛盾。

综上，我们的算法可以给出最优解。

## 代码

```cpp
class Solution {
public:
    bool lemonadeChange(vector<int>& bills) {
        // 收5元：没有问题
        // 收10元：需要找一张5元
        // 收20元：需要找3张5元，或1张5元+1张10元
        // 所以从贪心的角度，我觉得对于收20元，优先找10元+5元是合理的
        int fiveCnt = 0, tenCnt = 0, twentyCnt = 0;
        for (int bill: bills) {
            if (bill == 5)
                fiveCnt++;
            else if (bill == 10) {
                if (fiveCnt == 0)
                    return false;
                fiveCnt--;
                tenCnt++;
            }
            else if (bill == 20) {
                if (tenCnt > 0 && fiveCnt > 0) {
                    tenCnt--;
                    fiveCnt--;
                    twentyCnt++;
                }
                else if (fiveCnt >= 3) {
                    fiveCnt -= 3;
                    twentyCnt++;
                }
                else
                    return false;
            }
        }

        return true;
    }
};
```
