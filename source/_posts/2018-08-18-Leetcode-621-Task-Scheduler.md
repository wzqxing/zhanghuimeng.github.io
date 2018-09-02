---
title: Leetcode 621. Task Scheduler（贪心）
urlname: leetcode-621-task-scheduler
toc: true
date: 2018-08-18 20:45:05
updated: 2018-08-18 21:48:00
tags: [Leetcode, alg:Greedy]
---

题目来源：[https://leetcode.com/problems/task-scheduler/description/](https://leetcode.com/problems/task-scheduler/description/)

标记难度：Medium

提交次数：2/5

代码效率：

* 排序+贪心：6.90%
* 直接贪心：98.84%

## 题意

有若干种任务，每种任务有若干个时间片，同一任务的两个时间片之间的间隔必须超过`n`。问执行完所有任务至少需要多少个时间片。

## 分析

很容易想到一种直接的贪心方法：每次选择已经到了冷却时间且剩余个数最多的任务来执行。

另一种思路则十分巧妙。事实上，上一种思路中我们浪费了过多的信息。从题意中可以看出，任务的名称和执行并没有明确的关系，因此不妨假设我们已经按任务所需时间片的数量从大到小排好序了，为`A`到`Z`。若`n >= 25`，则任务的执行必然呈现出周期性，因为：

* 第`1`个时间片：因为之前没有执行过A，且A所需时间片最多，所以执行A
* 第`2`个时间片：A未到冷却时间；之前没有执行过B，且B在剩余任务中所需时间片最多，所以执行B
* 第`3`个时间片：A、B未到冷却时间；之前没有执行过C，且C在剩余任务中所需时间片最多，所以执行C
* ……
* 第`26`个时间片：只有Z未到冷却时间，执行Z
* 空闲……
* 第`n+2`个时间片：由于A-Z已经各执行了一次，剩余时间片大小的顺序关系不变，因此仍然按上述顺序执行
* ……
* 第？个时间片：此时只有时间片总量最大的A（也许还有B，等等）还没有执行完，执行完A后，我们就可以结束执行了。

此时用图来表现执行顺序，就会变成这样：

```
ABCD......ZXXXX
ABCD......ZXXXX
ABCD......ZXXXX
......
A
```

显然我们应该把`n`的界再缩小一些：假设实际任务的总数为`m`，则`n >= m - 1`时，实际执行状况都是这样的。不妨设A的时间片个数为`maxIntervals`，时间片数量与A相等的任务个数为`maxCount`，则所需时间片总量为`(maxIntervals - 1) * (n + 1) + maxCount`。

当`n < m - 1`时，我们只需先把前`n + 1`个任务安排好：

```
ABCDE
ABCDE
ABCDE
ABCDE
...
ABC
```

然后把剩余的任务按顺序插进去，有多少个就插多少个：

```
ABCDEFGHI
ABCDEFGH
ABCDEFGH
ABCDEFG
...
ABC
```

此时没有空闲时间片，所以需要的时间片总量为全部任务时间片的总量。

特别地，当`n < m - 1`且`maxCount >= n - 1`时，我们不需要按照`n`的循环来安排任务，直接按照`m`来循环即可：

```
ABCD......Z
ABCD......Z
ABCD......Z
......
AB
```

上述思路参考了[concise Java Solution O(N) time O(26) space](https://leetcode.com/problems/task-scheduler/discuss/104496/concise-Java-Solution-O%28N%29-time-O%2826%29-space)和[Java O(n) time O(1) space 1 pass, no sorting solution with detailed explanation](https://leetcode.com/problems/task-scheduler/discuss/104500/Java-O%28n%29-time-O%281%29-space-1-pass-no-sorting-solution-with-detailed-explanation)。

## 代码

### 排序+贪心

```cpp
class Solution {
public:
    int leastInterval(vector<char>& tasks, int n) {
        int taskLeft[26];
        memset(taskLeft, 0, sizeof(taskLeft));
        int lastExecute[26];
        memset(lastExecute, -1, sizeof(lastExecute));

        for (char c: tasks)
            taskLeft[c - 'A']++;

        int now = 0;
        while (true) {
            int task = -1, left = -1;
            bool found = false;
            for (int i = 0; i < 26; i++) {
                if (taskLeft[i] > 0) {
                    found = true;
                    if (lastExecute[i] == -1 || now - lastExecute[i] > n) {
                        if (left == -1 || taskLeft[i] > left) {
                            left = taskLeft[i];
                            task = i;
                        }
                    }
                }
            }
            if (!found)
                break;
            // cout << now << ' ' << task << endl;
            if (found && task == -1) {
                now++;
                continue;
            }
            taskLeft[task]--;
            lastExecute[task] = now;
            now++;
        }

        return now;
    }
};
```

### 直接贪心

```cpp
class Solution {
public:
    int leastInterval(vector<char>& tasks, int n) {
        int taskLeft[26];
        memset(taskLeft, 0, sizeof(taskLeft));

        for (char c: tasks)
            taskLeft[c - 'A']++;

        sort(taskLeft, taskLeft + 26);
        int maxes = 0;
        for (int i = 25; i >= 0 && taskLeft[i] == taskLeft[25]; i--)
            maxes++;

        return max((int) tasks.size(), (taskLeft[25] - 1) * (n + 1) + maxes);
    }
};
```
