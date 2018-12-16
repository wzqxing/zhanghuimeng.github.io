---
title: Leetcode 957. Prison Cells After N Days
urlname: leetcode-957-prison-cells-after-n-days
toc: true
date: 2018-12-16 15:42:09
updated: 2018-12-16 15:42:09
tags: [Leetcode, Leetcode Contest]
---

题目来源：[https://leetcode.com/problems/prison-cells-after-n-days/description/](https://leetcode.com/problems/prison-cells-after-n-days/description/)

标记难度：Medium

提交次数：1/3

代码效率：8ms

## 题意

给定一行8个格子，用类似于生命游戏的方法进行更新：

* 如果相邻有2个格子被占用或有**2**个格子为空，则变为被占用
* 否则变为空

问`N`步后这8个格子的状态。`N <= 1e9`。

## 分析

比赛的时候看到这道题我就开始慌了，然后错了2次。这次比赛居然有三道Medium，一道Hard，可以说是偏难了。

---

一个很显然的观察是，两侧的格子在1天后都会变成空并且保持这个状态，所以实际上可以只考虑6个格子的状态。所以状态数量最多只有`1+2^6`种。不过当成`2^8`种来做也没什么问题。

显然用线性算法在`N=1e9`的时候必然会超时。不过，观察到状态必然会重复之后，可以通过找到循环来解决问题。如果第`i`天的状态与第`start`天相同，那么循环节的长度为`i - start`，可以令`N = (N - i) % (i - start)`，然后找到对应的天数的状态。

我比赛的时候没有写`N--`，而是`i = 0 to N-1`，显然前一种比较利于取模，后一种比较麻烦。所以我在给`N`取模这一点上错了两次。

另一个观察是[^lee215]，因为格子的数量是偶数，所以对于每一种状态，都可以找到一种合法的之前的状态，所以初始状态后一步就可以进入循环；而且循环节长度必然是1、7或14，所以可以在一步之后就按照14为循环节进行操作。

[^lee215]: [lee215's Solution for 957 - \[Java/Python\] Find the Loop, Mod 14](https://leetcode.com/problems/prison-cells-after-n-days/discuss/205684/JavaPython-Find-the-Loop-Mod-14)

## 代码

```cpp
class Solution {
private:
    int encode() {
       int state = 0;
        for (int i = 0; i < 8; i++)
            state = (state << 1) | s[i];
        return state;
    }
    
    void decode(int x) {
        for (int i = 0; i < 8; i++) {
            s[7 - i] = (x & 1) > 0;
            x >>= 1;
        }
    }
    
    int s[8];
    
public:
    vector<int> prisonAfterNDays(vector<int>& cells, int N) {
        int state = 0;
        for (int i = 0; i < 8; i++) {
            s[i] = cells[i];
        }
        state = encode();
        unordered_map<int, int> state2num, num2state;
        for (int i = 0; i < N; i++) {
            if (state2num.find(state) != state2num.end()) {
                int start = state2num[state];
                int x = (N - i) % (i - start) + start;  // 这是对的，但是比较麻烦
                state = num2state[x];
                decode(state);
                break;
            }
            state2num[state] = i;
            num2state[i] = state;
            // 进入下一个状态
            int s2[8];
            for (int i = 0; i < 8; i++) {
                int vacuum = 0, occupied = 0;
                if (i > 0) {
                    if (s[i-1] == 0) vacuum++;
                    else occupied++;
                }
                if (i < 7) {
                    if (s[i+1] == 0) vacuum++;
                    else occupied++;
                }
                if (vacuum == 2 || occupied == 2) s2[i] = 1;
                else s2[i] = 0;
            }
            memcpy(s, s2, sizeof(s));
            state = encode();
        }
        // 返回
        for (int i = 0; i < 8; i++)
            cells[i] = s[i];
        return cells;
    }
};
```