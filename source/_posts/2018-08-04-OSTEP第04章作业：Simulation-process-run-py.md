---
title: 'OSTEP第04章作业：Simulation: process-run.py'
urlname: ostep-ch-04-homework-simulation-process-run-py
toc: true
date: 2018-08-04 19:30:57
updated: 2018-08-04 19:30:57
tags: [OSTEP, OS]
---

本章作业见[http://pages.cs.wisc.edu/~remzi/OSTEP/cpu-intro.pdf](http://pages.cs.wisc.edu/~remzi/OSTEP/cpu-intro.pdf)和[http://pages.cs.wisc.edu/~remzi/OSTEP/Homework/HW-CPU-Intro.tgz](http://pages.cs.wisc.edu/~remzi/OSTEP/Homework/HW-CPU-Intro.tgz)。

---

作业说明和模拟器代码见<https://github.com/zhanghuimeng/ostep-hw-translation/tree/master/ch04_Process-Intro>。

这次作业没有明确地说明调度策略——这也是因为现在还没有介绍到调度策略这么复杂的东西。通过阅读代码，我发现选择下一个运行进程的策略是通过PID进行循环查找：从当前进程的PID开始，如果这个进程处于就绪态，则选择它；否则`PID = (PID + 1) % 进程总数`。这个策略不太实际（有点类似于FIFO，但是优先级依赖于PID），但显然对于手动模拟比较方便。

## 1
用以下参数运行程序：`./process-run.py -l 5:100,5:100`。CPU利用率（CPU处于使用状态的时间比例）是多少？为什么？用参数`-c`和`-p`验证你的回答的正确性。

---

我猜测Process0会先运行5个时间片；Process0运行完之后，Process1再运行5个时间片，也会结束。因为CPU运行完Process0就运行了Process1，中间没有空闲的时间，因此CPU利用率为100%。

运行结果如下（因为是win环境，所以用的是python）：
```
prompt> python process-run.py -l 5:100,5:100 -c -p
Time     PID: 0     PID: 1        CPU        IOs
  1     RUN:cpu      READY          1
  2     RUN:cpu      READY          1
  3     RUN:cpu      READY          1
  4     RUN:cpu      READY          1
  5     RUN:cpu      READY          1
  6        DONE    RUN:cpu          1
  7        DONE    RUN:cpu          1
  8        DONE    RUN:cpu          1
  9        DONE    RUN:cpu          1
 10        DONE    RUN:cpu          1

Stats: Total Time 10
Stats: CPU Busy 10 (100.00%)
Stats: IO Busy  0 (0.00%)
```

可以看出这个推测是正确的。

## 2
用以下参数运行程序：`./process-run.py -l 4:100,1:0`。这些参数给定了两个进程，其中一个包含4条CPU指令，另一个只发出一个I/O请求并等待请求结束。两个进程都结束执行需要多长时间？用参数`-c`和`-p`验证你的回答的正确性。

---

Process0不发出I/O请求，因此它会一直执行到结束，共花费4个时间片。Process1发出一个I/O请求（1个时间片），I/O请求执行完毕需要5个时间片；因此，一共需要10个时间片结束执行。

运行结果如下（实际上刚才的结果参考了运行结果，因为我不知道所谓的“I/O请求花费5个时间片”算不算发出请求的指令的时间……当然从情理上和实验上来说都是不算的）：

```
prompt> python process-run.py -l 4:100,1:0 -c -p
Time     PID: 0     PID: 1        CPU        IOs
  1     RUN:cpu      READY          1
  2     RUN:cpu      READY          1
  3     RUN:cpu      READY          1
  4     RUN:cpu      READY          1
  5        DONE     RUN:io          1
  6        DONE    WAITING                     1
  7        DONE    WAITING                     1
  8        DONE    WAITING                     1
  9        DONE    WAITING                     1
 10*       DONE       DONE

Stats: Total Time 10
Stats: CPU Busy 5 (50.00%)
Stats: IO Busy  4 (40.00%)
```

## 3
现在切换进程的顺序：`./process-run.py -l 1:0,4:100`。切换顺序对于结束执行的时间有影响吗？继续用参数`-c`和`-p`验证你的回答的正确性。

---

显然有影响，因为Process0发出一个I/O请求（花费1个时间片）后，就会切换到Process1开始执行（4个时间片）；与此同时，I/O请求需要5个时间片完成。因此执行时间减少到了6个时间片。这说明了进程切换的必要性，因为增加了并行性，可以增加各种设备的利用率。

运行结果如下：
```
prompt> python process-run.py -l 1:0,4:100 -c -p
Time     PID: 0     PID: 1        CPU        IOs
  1      RUN:io      READY          1
  2     WAITING    RUN:cpu          1          1
  3     WAITING    RUN:cpu          1          1
  4     WAITING    RUN:cpu          1          1
  5     WAITING    RUN:cpu          1          1
  6*       DONE       DONE

Stats: Total Time 6
Stats: CPU Busy 5 (83.33%)
Stats: IO Busy  4 (66.67%)
```

## 4
下面我们探索一下其他的参数。参数`-S`指定了进程发出I/O请求时系统的反应策略。当该参数的值为`SWITCH_ON_END`时，系统不会在当前进程发出I/O请求时切换到另一个进程，而是等待进程结束之后再切换。如果你用以下参数运行程序（`-l 1:0,4:100 -c -S SWITCH_ON_END`），会发生什么？

---

这道题的进程配置和上一道题一样，但是如果在进程发出I/O请求时不切换，就相当于必须执行完当前进程才能切换到下一个，这样设备利用率显然会下降，而运行时间会增加。Process0执行完需要6个时间片，Process1需要4个时间片，因此总时间为10个时间片，与进程顺序颠倒时相同。

运行结果如下（好吧，这说明我的分析有一点问题，这似乎与结束的总时间如何计算有关）：
```
prompt> python process-run.py -l 1:0,4:100 -c -S SWITCH_ON_END -p
Time     PID: 0     PID: 1        CPU        IOs
  1      RUN:io      READY          1
  2     WAITING      READY                     1
  3     WAITING      READY                     1
  4     WAITING      READY                     1
  5     WAITING      READY                     1
  6*       DONE    RUN:cpu          1
  7        DONE    RUN:cpu          1
  8        DONE    RUN:cpu          1
  9        DONE    RUN:cpu          1

Stats: Total Time 9
Stats: CPU Busy 5 (55.56%)
Stats: IO Busy  4 (44.44%)
```

## 5
现在把`-S`参数的值置为SWITCH_ON_IO，此时只要进程发出I/O请求，就会切换到别的进程。（参数为`-l 1:0,4:100 -c -S SWITCH_ON_IO`）。那么，会发生什么呢？用参数`-c`和`-p`验证你的回答的正确性。

---

把参数设置成这样似乎就是默认值。结论是和第4题的分析相同吧？

运行结果如下（是的，的确如此）：
```
prompt> python process-run.py -l 1:0,4:100 -c -S SWITCH_ON_IO -p
Time     PID: 0     PID: 1        CPU        IOs
  1      RUN:io      READY          1
  2     WAITING    RUN:cpu          1          1
  3     WAITING    RUN:cpu          1          1
  4     WAITING    RUN:cpu          1          1
  5     WAITING    RUN:cpu          1          1
  6*       DONE       DONE

Stats: Total Time 6
Stats: CPU Busy 5 (83.33%)
Stats: IO Busy  4 (66.67%)
```

## 6
I/O请求结束时系统的执行策略也很重要。如果将参数`-I`的值置为`IO_RUN_LATER`，则I/O请求完成时，发出请求的进程不会立刻开始执行，当前运行中的进程会继续运行。如果使用以下参数组合，会发生什么？（`./process-run.py -l 3:0,5:100,5:100,5:100 -S SWITCH_ON_IO -I IO_RUN_LATER`）

---

`IO_RUN_LATER`似乎就是默认值。Process0首先发出一个I/O请求（1个时间片），之后Process1，Process2和Process3依次执行完毕（共15个时间片）；最后Process0再重新开始执行，花费10个时间片进行两个I/O请求。然后，似乎因为最后一条指令为I/O指令，所以需要多花一个时间片确认为DONE（这一点是根据运行结果凑出来的）。因此总时间为27个时间片。

这个调度方法比较愚蠢，如果把Process0的I/O请求分散开，可以提高CPU和I/O设备的利用率。这说明在设计调度策略的时候，我们应当考虑到I/O密集型进程和CPU密集型进程的不同特点。

运行结果如下：
```
prompt> python process-run.py -l 3:0,5:100,5:100,5:100 -S SWITCH_ON_IO -I IO_RUN_LATER -c -p
Time     PID: 0     PID: 1     PID: 2     PID: 3        CPU        IOs
  1      RUN:io      READY      READY      READY          1
  2     WAITING    RUN:cpu      READY      READY          1          1
  3     WAITING    RUN:cpu      READY      READY          1          1
  4     WAITING    RUN:cpu      READY      READY          1          1
  5     WAITING    RUN:cpu      READY      READY          1          1
  6*      READY    RUN:cpu      READY      READY          1
  7       READY       DONE    RUN:cpu      READY          1
  8       READY       DONE    RUN:cpu      READY          1
  9       READY       DONE    RUN:cpu      READY          1
 10       READY       DONE    RUN:cpu      READY          1
 11       READY       DONE    RUN:cpu      READY          1
 12       READY       DONE       DONE    RUN:cpu          1
 13       READY       DONE       DONE    RUN:cpu          1
 14       READY       DONE       DONE    RUN:cpu          1
 15       READY       DONE       DONE    RUN:cpu          1
 16       READY       DONE       DONE    RUN:cpu          1
 17      RUN:io       DONE       DONE       DONE          1
 18     WAITING       DONE       DONE       DONE                     1
 19     WAITING       DONE       DONE       DONE                     1
 20     WAITING       DONE       DONE       DONE                     1
 21     WAITING       DONE       DONE       DONE                     1
 22*     RUN:io       DONE       DONE       DONE          1
 23     WAITING       DONE       DONE       DONE                     1
 24     WAITING       DONE       DONE       DONE                     1
 25     WAITING       DONE       DONE       DONE                     1
 26     WAITING       DONE       DONE       DONE                     1
 27*       DONE       DONE       DONE       DONE

Stats: Total Time 27
Stats: CPU Busy 18 (66.67%)
Stats: IO Busy  12 (44.44%)
```

## 7
将参数`-I`的值换成`IO_RUN_IMMEDIATE`，重新执行上述命令，此时，当I/O请求完成时，发出请求的进程会立刻抢占CPU。现在程序的运行结果有何不同？为什么让刚刚执行完I/O的进程立刻开始运行可能是个好主意？

---

就像上一题所分析的那样，如果把I/O请求分散开来，则可以提高设备的利用率。在设计调度策略的时候应当考虑到进程的I/O密集程度，这个思路见于多级反馈队列调度算法中——如果进程用完了时间片则下移一个队列；否则，如果在时间片结束之前就发出了I/O请求，则保留在当前队列中。此时花费的总时间就是3条I/O指令加上15条CPU指令的时间。

运行结果如下：
```
prompt> python process-run.py -l 3:0,5:100,5:100,5:100 -S SWITCH_ON_IO -I IO_RUN_IMMEDIATE -c -p
Time     PID: 0     PID: 1     PID: 2     PID: 3        CPU        IOs
  1      RUN:io      READY      READY      READY          1
  2     WAITING    RUN:cpu      READY      READY          1          1
  3     WAITING    RUN:cpu      READY      READY          1          1
  4     WAITING    RUN:cpu      READY      READY          1          1
  5     WAITING    RUN:cpu      READY      READY          1          1
  6*     RUN:io      READY      READY      READY          1
  7     WAITING    RUN:cpu      READY      READY          1          1
  8     WAITING       DONE    RUN:cpu      READY          1          1
  9     WAITING       DONE    RUN:cpu      READY          1          1
 10     WAITING       DONE    RUN:cpu      READY          1          1
 11*     RUN:io       DONE      READY      READY          1
 12     WAITING       DONE    RUN:cpu      READY          1          1
 13     WAITING       DONE    RUN:cpu      READY          1          1
 14     WAITING       DONE       DONE    RUN:cpu          1          1
 15     WAITING       DONE       DONE    RUN:cpu          1          1
 16*       DONE       DONE       DONE    RUN:cpu          1
 17        DONE       DONE       DONE    RUN:cpu          1
 18        DONE       DONE       DONE    RUN:cpu          1

Stats: Total Time 18
Stats: CPU Busy 18 (100.00%)
Stats: IO Busy  12 (66.67%)
```

## 8
用下列随机生成的参数组合运行程序，比如`-s 1 -l 3:50,3:50`，`-s 2 -l 3:50,3:50`和`-s 3 -l 3:50,3:50`。你能否预测程序运行结果？将`-I`参数的值分别置为`IO_RUN_IMMEDIATE`和`IO_RUN_LATER`时，运行结果有何区别？将`-S`参数的值分别置为`SWITCH_ON_IO`和`SWITCH_ON_END`时，运行结果有何区别？

---

### 8.1

使用参数`-s 1 -l 3:50,3:50`，输出如下：
```
prompt> python process-run.py -s 1 -l 3:50,3:50
Produce a trace of what would happen when you run these processes:
Process 0
  cpu
  io
  io

Process 1
  cpu
  cpu
  cpu

Important behaviors:
  System will switch when the current process is FINISHED or ISSUES AN IO
  After IOs, the process issuing the IO will run LATER (when it is its turn)
```

`-I IO_RUN_LATER`时：
```
prompt> python process-run.py -s 1 -l 3:50,3:50 -I IO_RUN_LATER -c -p
Time     PID: 0     PID: 1        CPU        IOs
  1     RUN:cpu      READY          1
  2      RUN:io      READY          1
  3     WAITING    RUN:cpu          1          1
  4     WAITING    RUN:cpu          1          1
  5     WAITING    RUN:cpu          1          1
  6     WAITING       DONE                     1
  7*     RUN:io       DONE          1
  8     WAITING       DONE                     1
  9     WAITING       DONE                     1
 10     WAITING       DONE                     1
 11     WAITING       DONE                     1
 12*       DONE       DONE

Stats: Total Time 12
Stats: CPU Busy 6 (50.00%)
Stats: IO Busy  8 (66.67%)
```

`-I IO_RUN_IMMEDIATE`时结果相同，因为Process0发出的I/O请求在Process1完全执行结束之后才完成：

`-S SWITCH_ON_IO`时结果也相同，因为这个是默认值。

`-S SWITCH_ON_END`时运行时间变长，如下：
```
prompt> python process-run.py -s 1 -l 3:50,3:50 -S SWITCH_ON_END -c -p
Time     PID: 0     PID: 1        CPU        IOs
  1     RUN:cpu      READY          1
  2      RUN:io      READY          1
  3     WAITING      READY                     1
  4     WAITING      READY                     1
  5     WAITING      READY                     1
  6     WAITING      READY                     1
  7*     RUN:io      READY          1
  8     WAITING      READY                     1
  9     WAITING      READY                     1
 10     WAITING      READY                     1
 11     WAITING      READY                     1
 12*       DONE    RUN:cpu          1
 13        DONE    RUN:cpu          1
 14        DONE    RUN:cpu          1

Stats: Total Time 14
Stats: CPU Busy 6 (42.86%)
Stats: IO Busy  8 (57.14%)
```

### 8.2

使用参数`-s 2 -l 3:50,3:50`，输出如下：
```
prompt> python process-run.py -s 2 -l 3:50,3:50
Produce a trace of what would happen when you run these processes:
Process 0
  io
  io
  cpu

Process 1
  cpu
  io
  io

Important behaviors:
  System will switch when the current process is FINISHED or ISSUES AN IO
  After IOs, the process issuing the IO will run LATER (when it is its turn)
```

`-I IO_RUN_LATER`时，输出如下，出现了两个进程同时I/O的情况（不过此时我们认为I/O是可以并行的，不存在等待I/O设备的问题）：
```
prompt> python process-run.py -s 2 -l 3:50,3:50 -I IO_RUN_LATER -p -c
Time     PID: 0     PID: 1        CPU        IOs
  1      RUN:io      READY          1
  2     WAITING    RUN:cpu          1          1
  3     WAITING     RUN:io          1          1
  4     WAITING    WAITING                     2
  5     WAITING    WAITING                     2
  6*     RUN:io    WAITING          1          1
  7     WAITING    WAITING                     2
  8*    WAITING     RUN:io          1          1
  9     WAITING    WAITING                     2
 10     WAITING    WAITING                     2
 11*    RUN:cpu    WAITING          1          1
 12        DONE    WAITING                     1
 13*       DONE       DONE

Stats: Total Time 13
Stats: CPU Busy 6 (46.15%)
Stats: IO Busy  11 (84.62%)
```

`-I IO_RUN_IMMEDIATE`时，输出完全相同（因为I/O请求很多，所以当前I/O结束之后，CPU处于空闲状态，可以直接开始执行下一条指令）。

`-S SWITCH_ON_IO`时输出完全相同（因为这是默认值……）。

`-S SWITCH_ON_END`时运行时间大大增加了：
```
prompt> python process-run.py -s 2 -l 3:50,3:50 -S SWITCH_ON_END -p -c
Time     PID: 0     PID: 1        CPU        IOs
  1      RUN:io      READY          1
  2     WAITING      READY                     1
  3     WAITING      READY                     1
  4     WAITING      READY                     1
  5     WAITING      READY                     1
  6*     RUN:io      READY          1
  7     WAITING      READY                     1
  8     WAITING      READY                     1
  9     WAITING      READY                     1
 10     WAITING      READY                     1
 11*    RUN:cpu      READY          1
 12        DONE    RUN:cpu          1
 13        DONE     RUN:io          1
 14        DONE    WAITING                     1
 15        DONE    WAITING                     1
 16        DONE    WAITING                     1
 17        DONE    WAITING                     1
 18*       DONE     RUN:io          1
 19        DONE    WAITING                     1
 20        DONE    WAITING                     1
 21        DONE    WAITING                     1
 22        DONE    WAITING                     1
 23*       DONE       DONE

Stats: Total Time 23
Stats: CPU Busy 6 (26.09%)
Stats: IO Busy  16 (69.57%)
```

### 8.3

使用参数`-s 3 -l 3:50,3:50`，输出如下：
```
prompt> python process-run.py -s 3 -l 3:50,3:50
Produce a trace of what would happen when you run these processes:
Process 0
  cpu
  io
  cpu

Process 1
  io
  io
  cpu

Important behaviors:
  System will switch when the current process is FINISHED or ISSUES AN IO
  After IOs, the process issuing the IO will run LATER (when it is its turn)
```

`-I IO_RUN_LATER`时：
```
prompt> python process-run.py -s 3 -l 3:50,3:50 -I IO_RUN_LATER -c -p
Time     PID: 0     PID: 1        CPU        IOs
  1     RUN:cpu      READY          1
  2      RUN:io      READY          1
  3     WAITING     RUN:io          1          1
  4     WAITING    WAITING                     2
  5     WAITING    WAITING                     2
  6     WAITING    WAITING                     2
  7*    RUN:cpu    WAITING          1          1
  8*       DONE     RUN:io          1
  9        DONE    WAITING                     1
 10        DONE    WAITING                     1
 11        DONE    WAITING                     1
 12        DONE    WAITING                     1
 13*       DONE    RUN:cpu          1

Stats: Total Time 13
Stats: CPU Busy 6 (46.15%)
Stats: IO Busy  9 (69.23%)
```

`-I IO_RUN_IMMEDIATE`时输出不变，因为I/O请求结束的时间又一次恰好和CPU被占用的时间错开了。

`-S SWITCH_ON_IO`时输出不变（因为这仍然是默认值）。

`-S SWITCH_ON_END`时运行时间仍然会增加。
```
python process-run.py -s 3 -l 3:50,3:50 -S SWITCH_ON_END -c -p
Time     PID: 0     PID: 1        CPU        IOs
  1     RUN:cpu      READY          1
  2      RUN:io      READY          1
  3     WAITING      READY                     1
  4     WAITING      READY                     1
  5     WAITING      READY                     1
  6     WAITING      READY                     1
  7*    RUN:cpu      READY          1
  8        DONE     RUN:io          1
  9        DONE    WAITING                     1
 10        DONE    WAITING                     1
 11        DONE    WAITING                     1
 12        DONE    WAITING                     1
 13*       DONE     RUN:io          1
 14        DONE    WAITING                     1
 15        DONE    WAITING                     1
 16        DONE    WAITING                     1
 17        DONE    WAITING                     1
 18*       DONE    RUN:cpu          1

Stats: Total Time 18
Stats: CPU Busy 6 (33.33%)
Stats: IO Busy  12 (66.67%)
```
