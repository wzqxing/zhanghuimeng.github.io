---
title: 'OSTEP第05章总结：Interlude: Process API'
urlname: ostep-ch-05-summary-interlude-process-api
toc: true
date: 2018-07-07 16:49:54
updated: 2018-07-11 00:22:00
tags: [OSTEP, OS]
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

## 作业

在本次作业中，你将熟悉刚讲到的进程管理API的用法。这可是很有趣的，代码写得越多越好。所以赶紧去写吧。

（以下代码运行结果来自Ubuntu 16.04）

### 1

写一个程序，调用`fork()`。在调用`fork()`之前，让主进程设置一个变量（如`x`）的值（如100）。子进程中这个变量的值是多少？当子进程和父进程都修改`x`的值时，会发生什么？

---

程序代码见[hw01.c](https://github.com/zhanghuimeng/ostep-hw-translation/blob/master/ch05_Process-API/hw01.c)。

程序输出如下：
```
prompt> ./hw01
x = 100 (pid:2597)
parent: x = 100 (pid:2597)
parent: x = 101 (pid:2597)
child: x = 100 (pid:2598)
child: x = 99 (pid:2598)
prompt>
```

可以看出，子进程中`x`的值仍然为100，且父进程和子进程对`x`的修改是互相独立的。这是因为`fork()`系统调用把内存空间复制了一份（或者大概用了COW机制，不过这个暂时并不重要）。

### 2

写一个程序，通过`open()`系统调用打开一个文件，并调用`fork()`创建一个新的进程。子进程和父进程可以同时访问`open()`返回的文件描述符吗？如果它们同时写这个文件，会发生什么？

---

程序代码见[hw02.c](https://github.com/zhanghuimeng/ostep-hw-translation/blob/master/ch05_Process-API/hw02.c)。

程序输出如下：
```
prompt> ./hw02
prompt> cat hw02.output
I am parent
I am child
prompt>
```

可以发现，子进程和父进程可以同时访问这个文件，且输出结果基本是正常的。Linux实际上并不会保证并发的文件操作不出问题。不过，其实Linux的内部实现保证了`read()`和`write()`操作是串行执行的。详情可见[How do filesystems handle concurrent read/write?](https://stackoverflow.com/questions/2751734/how-do-filesystems-handle-concurrent-read-write)。

### 3

再写一个调用`fork()`的程序。令子进程打印`"hello"`，父进程打印`"goodbye"`。你能否在父进程中不调用`wait()`的情况下保证子进程总是先打印？

---

我感觉上一道题暗示我们，让子进程打印一点东西到文件中，然后在父进程中不断查看该文件中是否含有希望的信息。但我感觉这个做法可能不太可取。查了一些资料之后，我尝试让父进程用[kill(2)](https://linux.die.net/man/2/kill)检查子进程是否正在运行，但是我发现这也不可行，因为对僵尸进程进行这一检查也会返回`0`。事实上最科学的做法就是`wait()`系统调用了……所以我干脆用`waitpid()`好了。

程序代码见[hw03.c](https://github.com/zhanghuimeng/ostep-hw-translation/blob/master/ch05_Process-API/hw03.c)。

程序输出如下：

```
prompt> ./hw03
Hello, I am child
Goodbye, I am parent
prompt>
```

### 4

写一个程序，调用`fork()`，然后通过某种形式的`exec()`运行`/bin/ls`。你能否尝试使用`exec()`的所有变形，包括`execl()`、`execle()`、`execlp()`、`execv()`、`execp()`和`execvpe()`？这个调用为何有这么多种形式？

---

程序代码见[hw04.c](https://github.com/zhanghuimeng/ostep-hw-translation/blob/master/ch05_Process-API/hw04.c)。

程序输出如下：
```
prompt> ./hw04
Parent: ready for execl(const char *path, const char *arg, ... /* (char  *) NULL */)
bin    etc	   lib32       media  root  sys  vmlinuz
boot   home	   lib64       mnt    run   tmp
cdrom  initrd.img  libx32      opt    sbin  usr
dev    lib	   lost+found  proc   srv   var
Parent: ready for execlp(const char *file, const char *arg, ... /* (char  *) NULL */)
bin    etc	   lib32       media  root  sys  vmlinuz
boot   home	   lib64       mnt    run   tmp
cdrom  initrd.img  libx32      opt    sbin  usr
dev    lib	   lost+found  proc   srv   var
Parent: ready for execle(const char *path, const char *arg, ... /*, (char *) NULL, char * const envp[] */)
bin    etc	   lib32       media  root  sys  vmlinuz
boot   home	   lib64       mnt    run   tmp
cdrom  initrd.img  libx32      opt    sbin  usr
dev    lib	   lost+found  proc   srv   var
Parent: ready for execv(const char *path, char *const argv[])
bin    etc	   lib32       media  root  sys  vmlinuz
boot   home	   lib64       mnt    run   tmp
cdrom  initrd.img  libx32      opt    sbin  usr
dev    lib	   lost+found  proc   srv   var
Parent: ready for execvp(const char *file, char *const argv[])
bin    etc	   lib32       media  root  sys  vmlinuz
boot   home	   lib64       mnt    run   tmp
cdrom  initrd.img  libx32      opt    sbin  usr
dev    lib	   lost+found  proc   srv   var
Parent: ready for execvpe(const char *file, char *const argv[], char *const envp[])
bin    etc	   lib32       media  root  sys  vmlinuz
boot   home	   lib64       mnt    run   tmp
cdrom  initrd.img  libx32      opt    sbin  usr
dev    lib	   lost+found  proc   srv   var
prompt>
```

我猜这些形式主要是为了满足用户不同的需求。实际上，`exec`后面的那些后缀的含义是这样的（参见了[linux系统编程之进程（五）：exec系列函数（execl,execlp,execle,execv,execvp)使用](http://www.cnblogs.com/mickole/p/3187409.html)）：

* `l`：参数以可变参数列表的形式给出，且以`NULL`结束（`execl()`，`execle()`，`execlp()`）
* 没有`l`：参数以`char *arg[]`形式给出，且`arg`最后一个元素必须为`NULL`（`execv()`，`execp()`，`execvpe()`）
* `p`：第一个参数不用输入完整路径，给出命令名即可，程序会在环境变量PATH当中查找命令（`execlp()`，`execp()`，`execvpe()`）
* 没有`p`：第一个参数需要输入完整路径（`execl()`，`execle()`，`execv()`）
* `e`：将环境变量传递给新进程（`execle()`，`execvpe()`）
* 没有`e`：不传递环境变量（`execl()`，`execlp()`，`execv()`，`execp()`）

### 5

写一个程序，让父进程调用`wait()`，等待子进程完成。`wait()`将返回什么？如果在子进程中调用`wait()`，会发生什么？

---

程序代码见[hw05.c](https://github.com/zhanghuimeng/ostep-hw-translation/blob/master/ch05_Process-API/hw05.c)。

程序输出如下：

```
prompt> ./hw05
I am parent (pid:4154)
I am child (pid:4155)
Child: wait() returns -1
Parent: wait() returns 4155
prompt>
```

如果[wait(2)](http://man7.org/linux/man-pages/man2/waitpid.2.html)找到了至少一个状态已经变化的子进程，则它会返回这个子进程的PID（因此此处父进程返回了子进程的PID，4155）；而子进程自己没有子进程，因此调用失败，返回`-1`。

### 6

稍微修改一下上一题中的程序，改为使用`waitpid()`，而不是`wait()`。`waitpid()`何时是有用的？

---

程序代码见[hw06.c](https://github.com/zhanghuimeng/ostep-hw-translation/blob/master/ch05_Process-API/hw06.c)。

程序输出如下：

```
prompt> ./hw06
I am parent (pid:4368)
I am child (pid:4369)
Child: waitpid(-1) returns -1
Parent: waitpid(4369) returns 4369
prompt>
```

在需要等待某一个子进程执行完毕时，可以使用`waitpid()`。调用`waitpid(-1)`的效果与`wait()`基本是类似的。

### 7

写一个程序，创建一个子进程，并在子进程中关闭标准输出（`STDOUT_FILENO`）。在关闭该文件描述符之后，如果子进程调用`printf()`来打印输出，会发生什么？

---

程序代码见[hw07.c](https://github.com/zhanghuimeng/ostep-hw-translation/blob/master/ch05_Process-API/hw07.c)。

程序输出如下：
```
prompt> ./hw07
I am parent
prompt>
```

结果，无论是关闭之前还是关闭之后的`printf`都没有在屏幕打印出结果。我并没有查到为什么……

### 8

写一个程序，创建两个子进程，通过`pipe()`系统调用，把其中一个进程的标准输出连接到另一个的标准输入。

---

程序代码见[hw08.c](https://github.com/zhanghuimeng/ostep-hw-translation/blob/master/ch05_Process-API/hw08.c)，参考了[UNIX管道编程——使用pipe函数，dup函数，dup2函数](http://sealbird.iteye.com/blog/867908)。

程序输出如下：

```
prompt> ./hw08
Child 1 (pid=4727), writing to pipe.
Child 2 (pid=4728), reading from pipe.
Hello world , this is transported by pipe.
prompt>
```
