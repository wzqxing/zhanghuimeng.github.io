---
title: Leetcode 289. Game of Life（模拟）
urlname: leetcode-289-game-of-life
toc: true
date: 2018-07-31 20:34:30
updated: 2018-07-31 21:52:00
tags: [Leetcode]
---

题目来源：[https://leetcode.com/problems/game-of-life/description/](https://leetcode.com/problems/game-of-life/description/)

标记难度：Medium

提交次数：1/1

代码效率：100%

## 题意

给定生命游戏（元胞自动机）的现有状态，根据规则计算下一个状态。

进阶：

* 能否就地解决这个问题？
* 如果棋盘无穷大，如何解决该问题？

## 分析

如果不要求就地解决，这就是一个模拟大水题。但是，

### What is in-place?

我本来以为就地解决会用到什么高级的思路，因此我没有想出来。结果别人给出的“[就地解法](https://leetcode.com/problems/game-of-life/discuss/73223/Easiest-JAVA-solution-with-explanation)”竟然只是把前后两个状态通过位运算的方法硬塞进一个int里而已……虽然我脑子在这种方面是不太灵光啦，但我认为这只是一种hack的方法，它利用的是编程语言的性质，而非真正的数学思路；从抽象的角度来说，每个格子只有位置信息和1 bit的状态信息，仅此而已。

但是我很快就不得不收回自己说的话。我刚才看了[另一份题解](http://www.cnblogs.com/grandyang/p/4854466.html)，其实也可以从另一个角度来解释这种做法：其实我们在这个元胞自动机的每一个格子里又塞了一个有限状态自动机，像下图这样的。

![一个压缩了前后两种状态的有限状态自动机](simple-automata.jpg)

我原来的想法可能有一定的道理，但显然是狭隘的。

### 数据的实际状况，以及状态压缩

读了[Chapter 17 – The Game of Life](http://www.jagregory.com/abrash-black-book/#chapter-17-the-game-of-life)和[Chapter 18 – It’s a plain Wonderful Life](http://www.jagregory.com/abrash-black-book/#chapter-18-its-a-plain-wonderful-life)之后，我意识到了一些甚至更有趣的事情。这本书中给出了对模拟生命游戏进行进一步的速度优化的两种主要思路（我决定不把对指针的巧妙操作列为主要思路之一）：

* 关注数据的实际状态。通过生命游戏的规则，我们很容易看出，活细胞的数量不会太多，而且细胞状态的变化并不频繁。因此，我们可以维护一张实际变化了的细胞位置的表，并在更新时主要处理这张表指向的细胞。
* 状态压缩。除了像上面说的那样，把当前和未来状态打包起来以外，我们还可以把相邻活细胞的数量也记为状态的一部分，甚至还可以把多个细胞的状态一起压缩。除了减少空间占用之外，查表的复杂度也减小了。

书中David Stafford的解法把这几种思路有机地结合在了一起，创造了一份非常精妙的代码，以至于我很难分别描述每种思路各自的好处。总之他的思路非常棒。

![同时存储三个细胞](cell-triplet.jpg)

### 无穷多个细胞？

即使细胞有无穷多个，活细胞的数目也应该是有限的，所以我们只需分别更新每个活细胞及其周围细胞的状态即可。参见[Infinite board solution](https://leetcode.com/problems/game-of-life/discuss/73217/Infinite-board-solution)。

## 代码

这时候我的代码反而显得微不足道了呢……

```
class Solution {
public:
    void gameOfLife(vector<vector<int>>& board) {
        int n = board.size();
        if (n < 1)
            return;
        int m = board[0].size();
        // 直接将原状态保存下来
        vector<vector<int>> original;
        for (int i = 0; i < n; i++) {
            vector<int> tmp;
            for (int j = 0; j < m; j++)
                tmp.push_back(board[i][j]);
            original.push_back(tmp);
        }

        // 更新board
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                int cnt = 0;
                if (i > 0) cnt += original[i-1][j];
                if (i < n-1) cnt += original[i+1][j];
                if (j > 0) cnt += original[i][j-1];
                if (j < m-1) cnt += original[i][j+1];
                if (i > 0 && j > 0) cnt += original[i-1][j-1];
                if (i > 0 && j < m-1) cnt += original[i-1][j+1];
                if (i < n-1 && j > 0) cnt += original[i+1][j-1];
                if (i < n-1 && j < m-1) cnt += original[i+1][j+1];

                if (original[i][j] == 1) {
                    if (cnt < 2 || cnt > 3)
                        board[i][j] = 0;
                    else
                        board[i][j] = 1;
                }
                else if (cnt == 3)
                    board[i][j] = 1;
            }
        }
    }
};
```
