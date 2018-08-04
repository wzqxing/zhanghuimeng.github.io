---
title: OSTEP第02章总结：Introduction to Operating Systems
urlname: ostep-ch-02-summary-introduction-to-operating-systems
toc: true
date: 2018-07-01 16:09:21
updated: 2018-08-04 19:37:00
tags: [OSTEP, OS]
---

本章课本见[http://pages.cs.wisc.edu/~remzi/OSTEP/intro.pdf](http://pages.cs.wisc.edu/~remzi/OSTEP/intro.pdf)。

---

这一章对全书内容做了一些简单的概括：
* 用三个程序概括了本书将讲到的三个基本概念（虚拟化、并发、持久化）
* 讲了一些OS的设计目标和OS的历史

## 基本概念概述

### 虚拟化

虚拟化是本书第一部分的主题。

#### CPU的虚拟化

```
#include <stdio.h>
#include <stdlib.h>
#include "common.h"

int main(int argc, char *argv[]) {
    if (argc != 2) {
        fprintf(stderr, "usage: cpu <string>\n");
        exit(1);
    }
    char *str = argv[1];

    while (1) {
        printf("%s\n", str);
        Spin(1);
    }
    return 0;
}
```

上述程序中调用的`Spin()`函数（具体程序见本章附带的[代码](http://pages.cs.wisc.edu/~remzi/OSTEP/Code/code.intro.tgz)）会在运行1秒后返回。这个程序会不断运行，每秒输出一个`A`，如下：

```
prompt> gcc -o cpu cpu.c -Wall
prompt> ./cpu "A"
A
A
A
A
ˆC
prompt>
```

如果同时运行不同的程序实例，CPU会不断在程序之间切换，使得每个程序都认为自己独占了CPU，这被称为CPU的**虚拟化**。

```
prompt> ./cpu A & ; ./cpu B & ; ./cpu C & ; ./cpu D &
[1] 7353
[2] 7354
[3] 7355
[4] 7356
A
B
D
C
A
B
D
C
A
C
B
D
...
```

而OS选择什么程序来运行是一种**策略**，之后我们会学到相关内容（进程调度）。

#### 内存的虚拟化

```
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include <assert.h>
#include "common.h"

int
main(int argc, char *argv[])
{
    if (argc != 2) {
        fprintf(stderr, "usage: mem <value>\n");
        exit(1);
    }
    int *p;                   // memory for pointer is on "stack"
    p = malloc(sizeof(int));  // malloc'd memory is on "heap"
    assert(p != NULL);
    printf("(pid:%d) addr of p:        %llx\n", (int) getpid(),
        (unsigned long long) &p);
    printf("(pid:%d) addr stored in p: %llx\n", (int) getpid(),
        (unsigned long long) p);
    *p = atoi(argv[1]);       // assign value to addr stored in p
    while (1) {
        Spin(1);
        *p = *p + 1;
        printf("(pid:%d) value of p: %d\n", getpid(), *p);
    }

    return 0;
}
```

现代机器的内存模型就是一个数组，读写都需要给出地址。

上述程序所做的事很简单：
* 分配一些内存
* 打印内存的地址，以及程序的PID
* 将数0放入新分配的内存的第一个位置（4字节的int）中
* 循环，将p中存储的值+1

运行结果如下：
```
prompt> ./mem
(2134) address pointed to by p: 0x200000
(2134) p: 1
(2134) p: 2
(2134) p: 3
(2134) p: 4
(2134) p: 5
ˆC
```

如果仍然同时运行几个程序的实例，则会发现，每个程序都在同一地址处分配内存，而且对这一内存处存储的值的更新是相互独立的。

```
prompt> ./mem &; ./mem &
[1] 24113
[2] 24114
(24113) address pointed to by p: 0x200000
(24114) address pointed to by p: 0x200000
(24113) p: 1
(24114) p: 1
(24114) p: 2
(24113) p: 2
(24113) p: 3
(24114) p: 3
(24113) p: 4
(24114) p: 4
...
```

这说明OS对内存进行了虚拟化，每个进程访问的是自己的虚拟地址空间。

### 并发
```
#include <stdio.h>
#include <stdlib.h>
#include "common_threads.h"

volatile int counter = 0;
int loops;

void *worker(void *arg) {
    int i;
    for (i = 0; i < loops; i++) {
        counter = counter + 1;
    }
    pthread_exit(NULL);
}

int main(int argc, char *argv[]) {
    if (argc != 2) {
        fprintf(stderr, "usage: threads <loops>\n");
        exit(1);
    }
    loops = atoi(argv[1]);
    pthread_t p1, p2;
    printf("Initial value : %d\n", counter);
    Pthread_create(&p1, NULL, worker, NULL);
    Pthread_create(&p2, NULL, worker, NULL);
    Pthread_join(p1, NULL);
    Pthread_join(p2, NULL);
    printf("Final value   : %d\n", counter);
    return 0;
}
```

这一程序通过调用`Pthread_create`创建了两个线程，分别将一个计数器自增N次，然后将计数器的结果输出。显然，正常情况下程序输出应该为2N。

```
prompt> gcc -o thread thread.c -Wall -pthread
prompt> ./thread 1000
Initial value : 0
inal value : 2000
```

但如果两个线程执行的次数N比较大，则程序的输出可能不再会为2N，而且会不太稳定。这是由于+1的操作不是原子的。

```
prompt> ./thread 100000
Initial value : 0
Final value : 143012 // huh??
prompt> ./thread 100000
Initial value : 0
Final value : 137298 // what the??
```

这体现了并发中可能出现的一些问题，我们将要在本书的第二部分讲到这些问题。

### 持久化
```
#include <stdio.h>
#include <unistd.h>
#include <assert.h>
#include <fcntl.h>
#include <sys/types.h>
#include <string.h>

void do_work() {
    int fd = open("/tmp/file", O_WRONLY | O_CREAT | O_TRUNC,
		  S_IRUSR | S_IWUSR);
    assert(fd >= 0);
    char buffer[20];
    sprintf(buffer, "hello world\n");
    int rc = write(fd, buffer, strlen(buffer));
    assert(rc == strlen(buffer));
    printf("wrote %d bytes\n", rc);
    fsync(fd);
    close(fd);
}

int main(int argc, char *argv[]) {
    do_work();
    return 0;
}
```

上述代码会创建文件`/tmp/file`，并在里面写入字符串`hello world`。

内存是一种易失性存储，所以需要能够持续性地存储程序的硬件和软件，也就是硬盘（SSD）和文件系统。

和虚拟化的内存地址空间相比，因为用户通常会使用文件来共享信息，所以OS并不会为每个应用进程创建虚拟化的磁盘。OS向硬盘写数据的过程是十分复杂的，但OS对此进行了抽象，可以通过系统调用来访问设备。因此OS可以被看成是一个标准库。

## OS的设计目标

* 抽象：使系统更易用
* 高性能
* 保护：在应用程序之间，以及应用程序和系统之间提供保护和隔离
* 可靠性
* 节能、安全性、可移动性……

## OS的历史

* 早期操作系统（大型机批处理系统）：基本只是标准库，需要操作员参与管理
* 中期操作系统：提供了文件系统和保护机制，系统调用和硬件特权级的概念出现了
* 多道程序系统：内存保护和并发的概念
* 现代操作系统：……
