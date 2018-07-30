---
title: 'OSTEP第04章总结：The Abstraction: The Process'
urlname: ostep-ch-04-summary-the-abstraction-the-process
toc: true
date: 2018-07-04 01:30:08
updated: 2018-07-04 21:42:00
tags: [OSTEP, OS]
---

本章课本见<http://pages.cs.wisc.edu/~remzi/OSTEP/cpu-intro.pdf>。

这一章主要讲了OS的一种基本抽象模型：进程（Process）。
* 进程的定义
* 与进程相关的API
* 进程的创建过程和状态（生命周期）
* 进程的数据结构（进程控制块和进程状态队列）

## CPU的虚拟化
我们首先给出一个进程的非正式定义：进程是一个正在运行的程序。

我们通常希望同时运行多个（甚至成百上千个）程序。这样用起来很方便，但是我们遇到的挑战是：如何创造这样由很多个虚拟CPU的假象？

OS通过对CPU进行虚拟化创造这一假象。它不断地运行一个程序，暂停，然后再运行下一个。这被称作CPU的分时共享（time sharing），这种技术允许程序的并发，同时也会牺牲一定的性能，因为一个程序的运行时间肯定会慢得多。

我们可以把实现CPU的虚拟化的过程拆分成两个步骤：

* 机制（mechanism）：这是实现一种功能的底层方法或协议；如上下文切换（context switch），这是一种分时机制（time-sharing mechanism），用于帮助OS实现暂停一个程序后切换到下一个的功能
* 策略（policy）：这是利用底层机制，在OS内进行决策的算法；如调度（scheduling）算法会利用历史信息、程序信息和性能评价方式进行决策

这是一种模块化的程序设计思路。不过，本节中我们似乎既没有讲机制也没有讲策略，而是讲了一些抽象内容（进程）的基本概念。

## 进程的正式定义
上面给出的那个定义是正确的，然而并不全面。如何更具体地描述一个进程？一般来说，我们可以通过记录进程在运行过程中访问或影响的系统部分来描述一个进程。于是我们可以定义进程的机器状态（machine state）：

* 内存（地址空间）：包含指令和程序读写的数据
* 寄存器：通用寄存器和一些特殊寄存器，如PC（program counter）和栈指针、帧指针
* I/O信息：进程开启的文件列表

## 与进程相关的API
既然进程是OS为我们提供的一种抽象，那么显然需要一些使用它的方法。因此下面介绍了一些与进程相关的API，一般来说，任何现代OS都提供了这些API：

* 创建（create）：创建新进程
* 销毁（destroy）：强制终止进程，对于失控的进程来说是很必要的
* 等待（wait）：等待其他进程结束运行
* 其他控制（miscellaneous control）：除了杀死或等待进程以外的控制方式，如将进程挂起后恢复运行的机制
* 状态（status）：通常会提供一些能够获得进程状态信息的接口，包括运行时间和状态

## 进程的生命周期

### 进程的创建过程

为了启动一个进程，OS需要做以下事情：

1. 将程序的代码和静态数据（初始化了的变量）加载到进程地址空间中。通常程序以某种可执行文件格式存储在磁盘（或者SSD）中，所以OS需要从磁盘读取数据。
  * 此时有一个加载策略的选择问题：积极加载（eager load）还是懒惰加载（lazy load）
  * 积极加载：将所有代码和数据在运行之前就全部装入内存中，是早期和简单OS的做法
  * 懒惰加载：用到对应的代码和数据时才加载，是现代OS的做法。
  * 这个问题与分页（paging）和交换（swapping）有关，之后还会谈到。
2. 为程序的运行栈分配内存；对栈进行初始化（比如把`main`函数的`argc`和`argv`放进去）
3. 为程序的堆分配内存。堆用于在程序运行中动态分配内存，程序调用`malloc()`请求内存，调用`free()`释放内存。
4. 对I/O进行初始化：对于类UNIX系统，每个进程默认有三个打开的文件描述符（file descriptor），用于标准输入、标准输出和标准错误输出，这使得进程能够从终端读入输入并打印到屏幕。我们将在本书的第三部分（持久化）中更多的谈到这个问题。
5. 跳转到`main()`函数的起始点，将CPU的控制权交给新创建的进程。

### 进程的状态模型

创建了一个进程之后，它的状态（state）会不断变化。一个简化的进程三状态模型如下：

* 运行态（running）：进程正在处理器上运行，正在执行指令。
* 就绪态（ready）：进程已经准备好执行了，但由于某些原因，OS现在并没有运行它。
* 等待态（blocked）：进程执行了一些操作，使得它不能继续运行，直到发生某些其他事件为止。例如，进程启动对磁盘的I/O请求时，它就被阻塞了，此时其他的进程可以使用处理器。

![进程在状态之间的迁移](fig4-2_process-state-transitins.png)

上图说明了这个简化模型中进程如何在状态间迁移：

* OS可以通过调度使进程在运行态和就绪态之间转换
* 如果进程进入了等待态，则只有对应的事件发生时才会进入就绪态

下面举两个例子进行说明。在第一个例子中，两个进程只使用CPU而不进行I/O；调度策略是进程0执行完之后进程1才能开始执行。

![只使用CPU的两个进程的状态变化](fig4-3_tracing-process-state-cpu-only.png)

在第二个例子中，进程会进行I/O操作。当进程0进行I/O之后，它进入阻塞状态。于是进程1开始运行。进程0的I/O完成之后，它进入就绪态，等待进程1执行结束之后继续执行。

![使用CPU并进行I/O的两个进程的状态变化](fig4-4_tracing-process-state-cpu-and-io.png)

事实上，在上述过程中，OS采用了以下两种策略：
* 在进程0进行I/O时切换到进程1：这个决策看起来是明智的，因为可以增加CPU使用率
* 在进程0的I/O操作结束之后，没有立即切换回切换0：这个策略的好坏很难说

（上述内容将出现在作业中）

## 进程的数据结构
上面的内容可以说是非常抽象了。所以下面会讲到，这些抽象的内容如何用OS中具体的数据结构来表示。

### 进程控制块
我们把进程信息对应的数据结构称为进程控制块（Process Control Block，PCB）。下面是xv6教学OS中一个实际的进程控制块的定义（也就是说，会涉及大量**底层机制**内容）：

```
// the registers xv6 will save and restore
// to stop and subsequently restart a process
struct context {
    int eip;
    int esp;
    int ebx;
    int ecx;
    int edx;
    int esi;
    int edi;
    int ebp;
};

// the different states a process can be in
enum proc_state { UNUSED, EMBRYO, SLEEPING,
                  RUNNABLE, RUNNING, ZOMBIE };

// the information xv6 tracks about each process
// including its register context and state
struct proc {
    char *mem;                  // Start of process memory
    uint sz;                    // Size of process memory
    char *kstack;               // Bottom of kernel stack
                                // for this process
    enum proc_state state;      // Process state
    int pid;                    // Process ID
    struct proc *parent;        // Parent process
    void *chan;                 // If non-zero, sleeping on chan
    int killed;                 // If non-zero, have been killed
    struct file *ofile[NOFILE]; // Open files
    struct inode *cwd;          // Current directory
    struct context context;     // Switch here to run process
    struct trapframe *tf;       // Trap frame for the
                                // current interrupt
};
```

下面对其中一些比较重要的内容给出说明。首先可以看到，`context`中存储了通用寄存器的内容，用于上下文切换（保存一个暂停的进程的寄存器的值；在进程恢复运行时，这些值将被重新加载到寄存器中），这是之前讲到的。其次可以发现，这里定义了6种状态，和之前讲到的三状态模型不完全相符（这可能也体现了抽象策略和底层机制的区别）。除了运行态、就绪态和等待态以外，此处还定义了其他的状态，如初始（initial）态（进程刚被创建时的状态，对应的大概是上述代码中的EMBRYO态）和终止（final）态（进程已经退出，但尚未被完全清理，在类UNIX系统中，这一状态被称为僵尸（zombie）态）。进入终止态的进程允许其他进程（通常是创建这个进程的父进程）检查进程的返回值，查看它是否成功结束。父进程在自己结束之前会执行最后一个调用（`wait()`），等待子进程执行完成（进入终止态），此时OS可以清理子进程相关的数据结构。

（我不知道我有没有理解清楚关于僵尸态的内容）

### 进程状态队列
为了管理进程的状态，OS会维护一些进程状态队列，用来跟踪就绪进程、正在运行的进程和阻塞进程，并且在发生事件的时候唤醒正确的进程。

## 作业

作业说明和模拟器代码见<https://github.com/zhanghuimeng/ostep-hw-translation/tree/master/ch04_Process-Intro>。

这次作业没有明确地说明调度策略——这也是因为现在还没有介绍到调度策略这么复杂的东西。通过阅读代码，我发现选择下一个运行进程的策略是通过PID进行循环查找：从当前进程的PID开始，如果这个进程处于就绪态，则选择它；否则`PID = (PID + 1) % 进程总数`。这个策略不太实际（有点类似于FIFO，但是优先级依赖于PID），但显然对于手动模拟比较方便。

### 1
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

### 2
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

### 3
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

### 4
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

### 5
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

### 6
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

### 7
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

### 8
用下列随机生成的参数组合运行程序，比如`-s 1 -l 3:50,3:50`，`-s 2 -l 3:50,3:50`和`-s 3 -l 3:50,3:50`。你能否预测程序运行结果？将`-I`参数的值分别置为`IO_RUN_IMMEDIATE`和`IO_RUN_LATER`时，运行结果有何区别？将`-S`参数的值分别置为`SWITCH_ON_IO`和`SWITCH_ON_END`时，运行结果有何区别？

---

#### 8.1

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

#### 8.2

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

#### 8.3

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
