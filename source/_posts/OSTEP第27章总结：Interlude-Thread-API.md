---
title: 'OSTEP第27章总结：Interlude: Thread API'
urlname: ostep-ch-27-summary-interlude-thread-api
toc: true
date: 2018-04-16 19:43:44
tags: [OS, OSTEP]
---

本章主要介绍了POSIX标准下的线程API，包括：
* 创建线程
* 等待线程结束
* 创建和使用互斥锁
* 创建和使用条件变量

## 创建线程
```
int
pthread_create( pthread_t * thread,
		 const pthread_attr_t * attr,
			   void * (*start_routine)(void*),
               void * arg);
```

4个参数的含义如下：
* `thread`：指向`struct pthread_t`类型的变量的指针，我们需要这一变量用来控制线程，因此需要把它的指针传递过去进行初始化
* `attr`：指定线程的属性，如栈的大小和线程的调度优先级；属性是通过调用`pthread attr_init()`进行初始化的
* `start_routine`：指定线程运行的函数，它只有一个`void*`类型的参数，返回值也是`void*`类型
* `arg`：在进程执行的时候传递给函数的参数

例子：
```
#include <pthread.h>

typedef struct __myarg_t {
    int a;
    int b;
} myarg_t;

void *mythread(void *arg) {
    myarg_t *m = (myarg_t *) arg;
    printf("%d %d\n", m->a, m->b);
    return NULL;
}

int
main(int argc, char *argv[]) {
    pthread_t p;
    int rc;

    myarg_t args;
    args.a = 10;
    args.b = 20;
    rc = pthread_create(&p, NULL, mythread, &args);
    ...
}
```

## 等待线程结束
```
int pthread_join(pthread_t thread, void **value_ptr);
```

参数说明：
* `thread`：等待哪一个线程结束；这个变量是通过调用`pthread_create()`来初始化的
* `value_ptr`：指向将会得到的返回值的指针

### 使用示例
```
#include <stdio.h>
#include <pthread.h>
#include <assert.h>
#include <stdlib.h>
typedef struct __myarg_t {
    int a;
    int b;
} myarg_t;

typedef struct __myret_t {
    int x;
    int y;
} myret_t;

void *mythread(void *arg) {
    myarg_t *m = (myarg_t *) arg;
    printf("%d %d\n", m->a, m->b);
    myret_t *r = Malloc(sizeof(myret_t));
    r->x = 1;
    r->y = 2;
    return (void *) r;
}

int
main(int argc, char *argv[]) {
    pthread_t p;
    myret_t *m;

    myarg_t args = {10, 20};
    Pthread_create(&p, NULL, mythread, &args);
    Pthread_join(p, (void **) &m);
    printf("returned %d %d\n", m->x, m->y);
    free(m);
    return 0;
}
```

在上述代码中，创建了一个线程，传入了一些参数，在该线程返回之后，得到了它的返回值。当然，我们并不是总需要把参数和返回值包装成`struct`的。值得注意的是，不要返回指向在线程的调用栈上分配的内存的指针（而改为使用`malloc()`在堆上分配内存），因为在线程返回之后，它的栈就会被回收。

当然，在这个例子中，调用`pthread_create()`之后立刻调用`pthread_join()`的做法是没有什么意义的，因为函数调用也可以达到相同的效果。

并不是所有多线程程序都会用到`join`函数。有些多线程服务器可能会创建一些工作线程，然后让主线程不断接收请求并传递给其他线程。

## 互斥锁
最基本的一对上锁/解锁API是：
```
int pthread_mutex_lock(pthread_mutex_t *mutex);
int pthread_mutex_unlock(pthread_mutex_t *mutex);
```
较高级的API包括：
```
// 在无法获得锁时返回错误
int pthread_mutex_trylock(pthread_mutex_t *mutex);
// 在试图获得锁而不成功一段时间后返回错误
int pthread_mutex_timedlock(pthread_mutex_t *mutex,
                            struct timespec *abs_timeout);
```

### 使用示例
```
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;
pthread_mutex_lock(&lock);
x = x + 1; // or whatever your critical section is
pthread_mutex_unlock(&lock);
```

上述代码的含义是：
* 如果调用`pthread_mutex_lock()`时没有其他线程持有锁，则该线程可以获得锁并进入关键区
* 如果有其他的线程正在持有锁，则试图获得锁的进程将会阻塞，直到其他进程将锁释放，该进程获得锁为止

注意几点：
1. 只有拥有锁的线程才能调用`pthread_mutex_unlock()`
2. 需要检查上锁和开锁时可能发生的错误，最简单的方法是assert返回值
3. 除了静态初始化方法（使用`PTHREAD_MUTEX_INITIALIZER`进行默认初始化）

修改为动态初始化的示例如下：
```
pthread_mutex_t lock;
int rc = pthread_mutex_init(&lock, NULL);
assert(rc == 0); // always check success!
pthread_mutex_lock(&lock);
x = x + 1; // or whatever your critical section is
pthread_mutex_unlock(&lock);
pthread_mutex_destroy(&lock);
```

## 条件变量
```
int pthread_cond_wait(pthread_cond_t *cond, pthread_mutex_t *mutex);
int pthread_cond_signal(pthread_cond_t *cond);
```

需要在线程之间传递信号的时候（比如某个线程等待另一个完成），条件变量是很重要的。条件变量需要一个与它相关的锁。调用上述函数的前提是持有该锁。

`pthread_cond_wait()`函数的功能是，将调用该函数的线程睡眠，直到其他线程发出信号为止。通常的用法如下：
```
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;
Pthread_mutex_lock(&lock);
while (ready == 0)
Pthread_cond_wait(&cond, &lock);
Pthread_mutex_unlock(&lock);
```

上述代码在初始化之后，对`ready`变量进行检查，如果不满足要求，则继续睡眠，等待再次被唤醒。

在其他线程中唤醒该线程的代码如下：
```
Pthread_mutex_lock(&lock);
ready = 1;
Pthread_cond_signal(&cond);
Pthread_mutex_unlock(&lock);
```

值得注意的是：
* 在发出信号和修改全局变量`ready`时，为了防止竞争条件，我们总是确保已经获得了锁
* `wait`函数的第二个参数是锁，但`signal`函数的参数没有锁。这是因为`wait`函数在使线程进入睡眠的同时也释放了锁（否则其他线程就不可能获得锁并唤醒该线程）；在被唤醒之前，wait函数会重新获得锁
* 等待的线程会通过`while`循环检查条件是否满足，而不是`if`语句；这样做是最安全的，因为有些`pthread`的实现会使得条件未满足时线程仍被唤醒，因此唤醒只是一种提示

作者指出，最好不要自己尝试造轮子来实现条件变量的功能。

## 线程API使用指南
* 简化操作：线程之间复杂的互动会导致bug。
* 减少线程之间的互动
* 记得初始化锁和条件变量
* 检查返回值
* 小心传参的方式，不要返回指向线程栈上的指针
* 每个线程都拥有自己的栈，只有堆中的变量才是全局的
* 一定要通过条件变量在线程之间传递信号
* 多看手册

## 作业
仍然没做，这次的作业比较复杂，似乎要用到valgrind，还要配置一些环境。
