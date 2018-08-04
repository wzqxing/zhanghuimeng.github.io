---
title: 'OSTEP第05章总结：Interlude: Process API'
urlname: ostep-ch-05-summary-interlude-process-api
toc: true
date: 2018-07-07 16:49:54
updated: 2018-08-04 19:37:00
tags: [OSTEP, OS]
---

本章课本见[http://pages.cs.wisc.edu/~remzi/OSTEP/cpu-api.pdf](http://pages.cs.wisc.edu/~remzi/OSTEP/cpu-api.pdf)。

---

本章的内容是“幕间休息”（interlude）：这些章节讲的是OS中与具体API相关的内容，和原理关系不大，如果不想了解这些具体内容，可以跳过。（但是最好还是不跳过，因为实践出真知，对吧？）本章主要介绍了以下三个与进程创建相关的UNIX系统调用，以及它们的设计原理：

* `fork()`
* `wait()`
* `exec()`

## fork系统调用：通过复制来创建新进程

下面的例子说明了`fork()`系统调用的使用方法：

```
// p1.c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int
main(int argc, char *argv[])
{
    printf("hello world (pid:%d)\n", (int) getpid());
    int rc = fork();
    if (rc < 0) { // fork failed; exit
        fprintf(stderr, "fork failed\n");
        exit(1);
    } else if (rc == 0) { // child (new process)
        printf("hello, I am child (pid:%d)\n", (int) getpid());
    } else { // parent goes down this path (main)
        printf("hello, I am parent of %d (pid:%d)\n",
        rc, (int) getpid());
    }
    return 0;
}
```

上述程序的输出如下：

```
prompt> ./p1
hello world (pid:29146)
hello, I am parent of 29147 (pid:29146)
hello, I am child (pid:29147)
prompt>
```

可以看出，程序开始执行的时候打印了一条信息，其中包含了**进程标识符**（process identifier，PID）。该进程的PID是29146。在UNIX系统中，PID是进程的唯一标识。然后进程调用了`fork()`系统调用，通过拷贝当前进程创建了一个新进程。有趣的是，这两个进程几乎相同，都正准备从`fork()`系统调用返回。新进程（称为子进程；原来的进程称为父进程）不会从`main()`开始运行（因为`hello world`只被打印了一次），而是好像自己已经调用了`fork()`一样。这样设计的原因，将在后面进行解释。

子进程和父进程几乎相同（地址空间、寄存器、PC），只有一点区别：`fork()`调用的返回值不同。父进程的返回值是子进程的PID，而子进程的返回值是0。这一区别使得我们可以撰写代码分别处理这两种情况。

值得注意的另一点是，p1.c的输出是**不确定的**（nondeterminism）：当子进程创建的时候，系统中出现了两个活跃进程，而CPU调度器选择哪一个先开始运行是不确定的。因此，如果子进程被创建之后立刻开始运行，上述程序的输出就会变成这样：

```
prompt> ./p1
hello world (pid:29146)
hello, I am child (pid:29147)
hello, I am parent of 29147 (pid:29146)
prompt>
```

## wait系统调用：等待子进程退出

下面的例子说明了`wait()`系统调用的使用方法：

```
// p2.c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>

int
main(int argc, char *argv[])
{
    printf("hello world (pid:%d)\n", (int) getpid());
    int rc = fork();
    if (rc < 0) { // fork failed; exit
        fprintf(stderr, "fork failed\n");
        exit(1);
    } else if (rc == 0) { // child (new process)
        printf("hello, I am child (pid:%d)\n", (int) getpid());
    } else { // parent goes down this path (main)
        int wc = wait(NULL);
        printf("hello, I am parent of %d (wc:%d) (pid:%d)\n",
        rc, wc, (int) getpid());
    }
    return 0;
}
```

输出结果如下：

```
prompt> ./p2
hello world (pid:29266)
hello, I am child (pid:29267)
hello, I am parent of 29267 (wc:29267) (pid:29266)
prompt>
```

在这个例子中，父进程调用`wait()`，使得它在子进程结束执行之后才继续执行。当子进程结束之后，`wait()`调用才返回。此时，上述代码的输出显然是确定（deterministic）的了。如果父进程先运行，它会立刻调用`wait()`，等待子进程运行结束；因此子进程必然先运行。

## exec系统调用：通过覆盖改变当前进程的内容

下面的例子说明了`exec()`系统调用的使用方法，它一般和`fork()`一起使用，用于创建新进程：

```
// p3.c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/wait.h>

int
main(int argc, char *argv[])
{
    printf("hello world (pid:%d)\n", (int) getpid());
    int rc = fork();
    if (rc < 0) { // fork failed; exit
        fprintf(stderr, "fork failed\n");
        exit(1);
    } else if (rc == 0) { // child (new process)
        printf("hello, I am child (pid:%d)\n", (int) getpid());
        char *myargs[3];
        myargs[0] = strdup("wc"); // program: "wc" (word count)
        myargs[1] = strdup("p3.c"); // argument: file to count
        myargs[2] = NULL; // marks end of array
        execvp(myargs[0], myargs); // runs word count
        printf("this shouldn’t print out");
    } else { // parent goes down this path (main)
        int wc = wait(NULL);
        printf("hello, I am parent of %d (wc:%d) (pid:%d)\n",
        rc, wc, (int) getpid());
    }
    return 0;
}
```

输出结果如下：

```
prompt> ./p3
hello world (pid:29383)
hello, I am child (pid:29384)
29 107 1030 p3.c
hello, I am parent of 29384 (wc:29384) (pid:29383)
prompt>
```

事实上，在Linux中，`exec()`是一类系统调用的总称，一共有6个变种：`execl()`，`execlp()`，`execle()`，`execv()`，`execvp()`和`execvpe()`。详情见[exec(3)](http://man7.org/linux/man-pages/man3/exec.3.html)

在这个例子中，在调用`fork()`创建子进程后，子进程调用了`execvp()`，用程序`wc`覆盖自己并开始执行该程序。`wc`是字数统计（word counting）程序，此处它被用来统计源文件`p3.c`中行、词和字节的数量。

`fork()`系统调用的设计固然很怪，它的“同伙”`exec()`也有够怪的。事实上，`exec()`所做的事情是这样的：给定一个可执行文件的名字（如`wc`）和一些参数（如`p3.c`），它会**加载**（load）这个可执行文件的代码（和静态数据），覆盖当前进程的代码段和静态数据，并且重新初始化进程的堆栈和其他内存空间。然后OS把参数作为新进程的`argv`数组，直接开始运行新程序。所以`exec()`调用并没有创建一个新进程；它只是把当前正在运行的进程（`p3`）换成了一个新的程序（`wc`）。在子进程执行`exec()`调用之后，`p3.c`就好像从未运行过一样了；对`exec()`的成功调用是不会返回的。

## fork和exec的设计原因：方便shell和管道的实现

我们为什么要这样设计创建新进程的API呢？事实上，对于UNIX shell来说，在创建新进程的过程中把`fork()`和`exec()`分开是非常必要的，因为这样shell才能在调用`fork()`之后，调用`exec()`的过程之前运行一些代码来改变即将运行的程序的环境，这就使得我们可以创造很多有趣的特性。

shell是一个帮助你执行程序（命令）的用户程序。它显示一个**命令提示符**（prompt），然后等待你在里面打字。你在里面打一个命令（比如可执行程序的名字和参数）；然后，shell一般会找到这个可执行程序在文件系统中的位置，调用`fork()`创建一个新的子进程，然后（子进程）调用`exec()`的某个变种开始执行命令，最后（父进程）调用`wait()`等待命令执行结束。当子进程运行结束之后，shell（父进程）从`wait()`返回，再次打印出命令提示符，等待你的下一条指令。

把`fork()`和`exec()`分开使得shell能在其间够做很多有用的东西。比如，我们执行如下命令：

```
prompt> wc p3.c > newfile.txt
```

在这个例子中，`wc`的输出被**重定向**（redirect）到输出文件`newfile.txt`中。shell完成这个任务的方法很简单：在创建子进程之后，调用`exec()`之前，shell关闭**标准输出**（standard output）并打开文件`newfile.txt`。这样，即将被执行的程序`wc`的任何输出都会被发送到这个文件而不是屏幕。

下面的代码实现了子进程输出的重定向。

```
// p4.c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <fcntl.h>
#include <sys/wait.h>

int
main(int argc, char *argv[])
{
    int rc = fork();
    if (rc < 0) { // fork failed; exit
        fprintf(stderr, "fork failed\n");
        exit(1);
    } else if (rc == 0) { // child: redirect standard output to a file
        close(STDOUT_FILENO);
        open("./p4.output", O_CREAT|O_WRONLY|O_TRUNC, S_IRWXU);

        // now exec "wc"...
        char *myargs[3];
        myargs[0] = strdup("wc"); // program: "wc" (word count)
        myargs[1] = strdup("p4.c"); // argument: file to count
        myargs[2] = NULL; // marks end of array
        execvp(myargs[0], myargs); // runs word count
    } else { // parent goes down this path (main)
        int wc = wait(NULL);
    }
    return 0;
}
```

该程序的输出如下：

```
prompt> ./p4
prompt> cat p4.output
32 109 846 p4.c
prompt>
```

这种方法的工作原理与OS管理文件描述符的方法相关。UNIX系统在分配文件描述符时，会从0开始寻找空闲的文件描述符。[上一章](/post/ostep-ch-04-summary-the-abstraction-the-process)中曾经讲过，对于一个进程，默认有三个文件描述符是开启的：标准输入（`STDIN_FILENO=0`）、标准输出（`STDOUT_FILENO=1`）和标准错误输出（`STDERR_FILENO=2`）。关闭标准输出之后，在调用`open()`分配新的文件描述符时，`STDOUT_FILENO=1`就成了第一个可用的文件描述符，于是它就指向了我们需要的输出文件`./p4.output`。于是，子进程之后对标准输出文件描述符的写操作会被透明地指向新打开的文件。（真是有趣的设计啊）

UNIX**管道**（pipe）机制的实现方法类似。通过`pipe()`系统调用，一个进程的输出被连接到一个内核管道（pipe）中，另一个进程的输入也连接到这个相同的管道；这样，一个进程的输出就无缝连接到另一个进程的输入了。下面的例子通过管道命令实现了在文件中查找词并计算这个词出现次数的功能：

```
grep -o foo file | wc -l

```

## 温馨提示

### RTFM

我们刚才只是大概介绍了这些系统调用的基本原理，还有许多细节没有涉及到。为了了解这些细节，你应当去阅读手册。作为一个系统程序员，阅读**手册**（manual/man pages）是非常重要的，因为里面提供了很多细节，而且还可以帮助你减少烦你的同事的次数。如果你直接去问他们细节问题，他们可能会回答你：“[RTFM](https://zh.wikipedia.org/zh/RTFM)。”（Read the fucking manual！）

### Get it right

兰普森（[Butler W. Lampson](https://zh.wikipedia.org/wiki/%E5%B7%B4%E7%89%B9%E5%8B%92%C2%B7%E8%98%AD%E6%99%AE%E6%A3%AE)）在他那篇广受好评的论文“[Hints for Computer Systems Design](https://microsoft.com/en-us/research/wp-content/uploads/2016/02/acrobat-17.pdf)”中这样说：“做正确的事。（**Get it right.**）抽象和简化都不能代替正确的做法。”

实际上，设计进程创建API有很多方法，但是UNIX的设计者选择了正确的那一种。（虽然我觉得本章中并没有充分论述它的正确性）
