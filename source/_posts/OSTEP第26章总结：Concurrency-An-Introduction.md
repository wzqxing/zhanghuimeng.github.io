---
title: 'OSTEP第26章总结：Concurrency: An Introduction'
urlname: ostep-ch-26-summary-concurrency-an-introduction
toc: true
date: 2018-04-16 15:17:58
tags: [操作系统, OSTEP]
---

本章对多线程程序面临的两个问题（竞争条件和等待）进行了初步介绍。包括以下关键词：
* **关键区**（critical section）：访问共享资源（通常是变量或数据结构）的代码段。
* **竞争条件**（race condition）：多个线程几乎同时访问关键区并尝试修改数据时会出现的状况，结果很难预测。
* **不确定**（indeterminate）程序：包含竞争条件的程序，每次的运行结果是不确定的。
* **互斥访问**（mutual exclusion）：保证一次最多有一个进程能进入关键区，防止竞争出现。

## 为何需要线程
首先比较一下线程与进程。

相同点：
* PC：指令指针
* 通用寄存器
* 上下文切换：需要保存寄存器的状态
* 需要TCB（线程控制块）保存线程的状态
不同点：
* 不需要切换地址空间（页表）
* 单线程进程通常只有一个栈（位于地址空间的底部，向低地址增长），而多线程进程的每个线程都需要一个栈

需要线程的两个主要原因包括：
* 并行性（parallelism）：对于多CPU计算机，如果将每个线程分配给不同的CPU，则可以提高程序效率
* 防止慢速I/O导致程序阻塞：由于I/O是很慢的，当某个线程等待I/O结果而被阻塞时，其他线程就可以继续运行

虽然上述应用场景也可以用进程来实现，但线程有着可以方便地共享数据这一优势。

## 多线程的问题
书中首先举了一个例子，用来说明，多线程程序中线程的执行顺序是不可控的，因此可能会出现各种各样的问题。

```
#include <stdio.h>
#include "mythreads.h"
#include <stdlib.h>
#include <pthread.h>

void *
mythread(void *arg) {
    printf("%s\n", (char *) arg);
    return NULL;
}

int
main(int argc, char *argv[])
{                    
    if (argc != 1) {
	fprintf(stderr, "usage: main\n");
	exit(1);
    }

    pthread_t p1, p2;
    printf("main: begin\n");
    Pthread_create(&p1, NULL, mythread, "A");
    Pthread_create(&p2, NULL, mythread, "B");
    // join waits for the threads to finish
    Pthread_join(p1, NULL);
    Pthread_join(p2, NULL);
    printf("main: end\n");
    return 0;
}
```

在上述代码中，主程序会创建两个线程，每个线程都会运行函数`mythread()`，但是参数不同。创建线程之后，它可能立刻开始运行或进入就绪态（取决于调度器）。当然，在多处理器系统上，也可能有多个线程同时运行。

创建了两个线程（T1和T2）之后，主程序调用`pthread_join()`，等待线程运行结束。T1和T2都结束之后，主线程会重新开始运行，打印信息并退出。如上所述，执行过程涉及了三个线程：T1、T2和主线程。

事实上线程的执行过程有各种不同的可能性，比如：
* 主线程创建完两个线程之后，切换到T1，然后再切换到T2，最后回到主线程
  * 先打印A，后打印B
* 主线程创建完T1之后，T1开始运行，结束之后，回到主线程，创建T2，T2开始运行，最后回到主线程
  * 先打印A，后打印B
* 主线程创建完T1和T2之后，T2先开始运行，结束之后切换到T1，最后回到主线程
  * 先打印B，后打印A

这种执行的不确定性就是万恶之源了。

### 共享资源导致的竞争条件
```
#include <stdio.h>
#include "mythreads.h"
#include <stdlib.h>
#include <pthread.h>

int max;
volatile int counter = 0; // shared global variable

void *
mythread(void *arg)
{
    char *letter = arg;
    int i; // stack (private per thread)
    printf("%s: begin [addr of i: %p]\n", letter, &i);
    for (i = 0; i < max; i++) {
	counter = counter + 1; // shared: only one
    }
    printf("%s: done\n", letter);
    return NULL;
}

int
main(int argc, char *argv[])
{                    
    if (argc != 2) {
	fprintf(stderr, "usage: main-first <loopcount>\n");
	exit(1);
    }
    max = atoi(argv[1]);

    pthread_t p1, p2;
    printf("main: begin [counter = %d] [%x]\n", counter,
	   (unsigned int) &counter);
    Pthread_create(&p1, NULL, mythread, "A");
    Pthread_create(&p2, NULL, mythread, "B");
    // join waits for the threads to finish
    Pthread_join(p1, NULL);
    Pthread_join(p2, NULL);
    printf("main: done\n [counter: %d]\n [should: %d]\n",
	   counter, max*2);
    return 0;
}
```

上述代码的运行结果是变化的，有时候是正确的：
```
prompt> gcc -o main main.c -Wall -pthread
prompt> ./main
main: begin (counter = 0)
A: begin
B: begin
A: done
B: done
main: done with both (counter = 20000000)
```
有时候是错误的：
```
prompt> ./main
main: begin (counter = 0)
A: begin
B: begin
A: done
B: done
main: done with both (counter = 19345221)
```
而且运行结果还会不断变化。这种性质被称为**不确定性**（indeterminate）。出现这种情况的原因是，修改共享变量`counter`的指令不是原子的，因此可能在执行的任意阶段被调度器打断，此时运行结果就会取决于线程的调度顺序。这一状况被称为**争用条件**（race condition）。下图对此作了很好的说明：

![争用条件的结果](race-condition.png)

由于多个线程执行这段代码时会导致争用条件发生，我们称这段代码为**关键区**（critical section）。关键区包含了一段访问共享变量（或者说共享资源）的代码，不能被多于一个进程并发执行。为了正确执行代码，我们此时需要的是**互斥访问**（mutual exclusion），也就是保证一个线程在关键区内执行时，其他线程都无法进入关键区。

解决问题的方法之一是创造一种更强大的指令，它可以把我们需要完成的任务浓缩在一条指令内执行，这样就不会遭遇中断了。这种性质称为**原子性**（atomic）。这是一种美好的理想，但是对于一些很复杂却仍然需要保证原子性的操作，这种指令是不太可能实现的。

因此，我们转而尝试通过几条关键指令，实现**同步原语**（synchronization primitives），进而实现原子性。通过这些同步原语和OS的帮助，我们就可以在多线程代码中安全地访问关键区了。

接下来我们将解决这些问题：
* 实现同步原语需要怎样的硬件支持？
* OS需要提供怎样的支持？
* 如何正确高效地实现同步原语？
* 程序如何使用这些程序原语以得到希望的结果？

### 进程的相互等待问题
多线程面临的另一类问题是，线程有时需要等待其他线程结束或I/O请求完成。为此，我们不仅会学习同步原语的使用，也会学习**条件变量**（condition variables）。

## 作业题
暂时没做。作业题程序可见于<https://github.com/asnr/ostep/tree/master/concurrency/26_threads_intro>。
