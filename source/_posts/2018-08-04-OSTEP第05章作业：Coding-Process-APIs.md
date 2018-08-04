---
title: 'OSTEP第05章作业：Coding: Process APIs'
urlname: ostep-ch-05-homework-coding-process-apis
toc: true
date: 2018-08-04 19:39:20
updated: 2018-08-04 19:39:20
tags: [OSTEP, OS]
---

本章作业见[http://pages.cs.wisc.edu/~remzi/OSTEP/cpu-api.pdf](http://pages.cs.wisc.edu/~remzi/OSTEP/cpu-api.pdf)。

---

在本次作业中，你将熟悉刚讲到的进程管理API的用法。这可是很有趣的，代码写得越多越好。所以赶紧去写吧。

（以下代码运行结果来自Ubuntu 16.04）

## 1

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

## 2

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

## 3

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

## 4

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

## 5

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

## 6

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

## 7

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

## 8

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
