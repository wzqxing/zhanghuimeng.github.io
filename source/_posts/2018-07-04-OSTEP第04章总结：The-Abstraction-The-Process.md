---
title: 'OSTEP第04章总结：The Abstraction: The Process'
urlname: ostep-ch-04-summary-the-abstraction-the-process
toc: true
date: 2018-07-04 01:30:08
updated: 2018-08-04 19:37:00
tags: [OSTEP, OS]
---

本章课本见[http://pages.cs.wisc.edu/~remzi/OSTEP/cpu-intro.pdf](http://pages.cs.wisc.edu/~remzi/OSTEP/cpu-intro.pdf)。

---

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
