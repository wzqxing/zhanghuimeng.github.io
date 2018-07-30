---
title: OSTEP第28章总结：Locks
urlname: ostep-ch-28-summary-locks
toc: true
date: 2018-04-18 16:51:47
updated: 2018-04-18 16:51:47
tags: [OS, OSTEP]
---

本章主要介绍了锁的基础知识，以及各种锁的实现方法和评价标准：
* 硬件支持
  * 关中断
  * TS指令实现自旋锁
  * CS指令实现自旋锁
  * LL/SC指令实现自旋锁
  * FA指令实现无饥饿的自旋锁
* 硬件和OS支持
  * 从自旋锁到yield
  * 使用队列和睡眠
  * Linux中的实际例子
  * 两阶段锁

## 锁的基本定义和用法
锁是一个变量。在任意时刻，该变量中都存储着锁的状态：
* 可获得（或者说释放，开锁的，说明没有线程持有该锁）
* 不可获得（未被释放，上锁的，说明有一个线程持有该锁，很可能处于关键区内）

锁的使用方法：
```
lock_t mutex; // some globally-allocated lock 'mutex'
...
lock(&mutex);
balance = balance + 1;  // 关键区
unlock(&mutex);
```

`lock()`的语义：
* 尝试获得锁
* 如果该锁未被获得，则获得该锁，进入关键区，该线程成为锁的拥有者
* 如果该锁已被获得，则在锁被释放之前始终阻塞，防止多个线程同时进入关键区
`unlock()`的语义：
* 释放线程当前持有的锁
* 如果还有其他线程在等待锁，则通知其中一个进程开始运行，获得锁，并进入关键区

锁能够帮助程序员更多地获得对线程的控制权。

## 锁的评价标准
1. 基本要求：提供互斥访问功能
2. 公平性：是否会出现线程饥饿的状况？
3. 性能：不同场景下的时间开销
  1. 单线程获取并释放锁
  2. 多线程请求同一个锁
  3. 多CPU上的多线程请求同一个锁

## 锁的实现方法
本章中给出了一种单纯用现有非特权指令（加载和存储）实现锁的方法，但是失败了。当然，其实这样是可以成功的（Dekker和Peterson算法），也做过一些研究，但是这些算法对硬件也有一些假设，而且人们已经发现，在硬件中提供一些支持会方便很多。所以就不赘述了。

### 用硬件原语实现锁
#### 关中断
代码实现如下：
```
void lock() {
	DisableInterrupts();
}
void unlock() {
	EnableInterrupts();
}
```

优点：简单

缺点：
* 需要线程执行特权指令，可能被滥用，OS无法得到控制权
* 不适用于多处理器系统
* 关中断太久可能导致重要的中断请求被丢失
* 效率低，因为现代CPU开关中断的速度比较慢

目前这一方法主要用于OS本身的原子性操作，因为这样就不存在滥用问题。

#### 使用TS（Test-And-Set）指令实现自旋锁
TS指令原子地完成以下操作：
* 读出某个地址处的值
* 将新的值写入该地址
* 返回旧值

用代码表示如下：
```
int TestAndSet(int *old_ptr, int new) {
    int old = *old_ptr; // fetch old value at old_ptr
    *old_ptr = new; // store 'new' into old_ptr
    return old; // return the old value
}
```

可以利用TS指令实现自旋锁：
```
typedef struct __lock_t {
    int flag;
} lock_t;

void init(lock_t *lock) {
    // 0 indicates that lock is available, 1 that it is held
    lock->flag = 0;
}

void lock(lock_t *lock) {
    while (TestAndSet(&lock->flag, 1) == 1)
        ; // spin-wait (do nothing)
}

void unlock(lock_t *lock) {
    lock->flag = 0;
}
```

工作原理是：
* `flag`变量表示锁是否被持有，0表示空闲
* 在`lock()`函数中，不断使用TS指令测试锁是否空闲，空闲则获得锁并将`flag`置为1；不空闲则自旋等待
* 在`unlock()`函数中，只需简单地将`flag`置为0

#### 对自旋锁的评价
1. 正确性：显然是正确的
2. 公平性：无法保证，可能会导致饥饿
3. 性能：
  1. 单CPU系统：性能很差
  2. 多CPU系统：还不错（如果线程的数量大致与CPU相等）

#### 使用CS（Compare-And-Swap）指令实现自旋锁
CS指令原子地完成以下操作：
* 读出某个地址处的值
* 判断它是否与期望的值相等，如果相等，则更新该地址处的值为某给定的值
* 返回该地址处原有的值

用代码表示如下：
```
int CompareAndSwap(int *ptr, int expected, int new) {
	int actual = *ptr;
	if (actual == expected)
		*ptr = new;
	return actual;
}
```

与TS指令类似，可以利用CS指令实现自旋锁（其他函数与TS指令的实现相同）：
```
void lock(lock_t *lock) {
	while (CompareAndSwap(&lock->flag, 0, 1) == 1)
		; // spin
}
```

工作原理也类似。

#### 用LL（Load-Linked）和SC（Store-Conditional）指令实现自旋锁

这两个指令是MIPS架构下提供的。LL指令类似于普通的加载指令，把某个值从内存中加载到寄存器中；但它会标记这个地址，SC指令更新该地址处的值时，仅在上次LL过后该值还没有被修改过时才会成功。

用代码描述如下：
```
int LoadLinked(int *ptr) {
    return *ptr;
}

int StoreConditional(int *ptr, int value) {
    if (no one has updated *ptr since the LoadLinked to this address) {
        *ptr = value;
        return 1; // success!
    } else {
        return 0; // failed to update
    }
}
```

锁的实现方法：
```
void lock(lock_t *lock) {
    while (1) {
        while (LoadLinked(&lock->flag) == 1)
            ; // spin until it's zero
        if (StoreConditional(&lock->flag, 1) == 1)
            return; // if set-it-to-1 was a success: all done
        // otherwise: try it all over again
    }
}

void unlock(lock_t *lock) {
    lock->flag = 0;
}
```

如果两个线程同时在请求这个锁，则先执行SC指令的线程可以获得锁，另一个线程执行SC指令时就会返回0，需要重新进入请求状态。

#### 用FA（Fetch-And-Add）指令实现标签锁（ticket lock）
FA指令原子地完成以下任务：
* 读出某个地址处的值
* 将这个值+1
* 重新写回该地址处
* 返回原来的值

用代码描述如下：
```
int FetchAndAdd(int *ptr) {
	int old = *ptr;
	*ptr = old + 1;
	return old;
}
```

用该指令可以实现一个思路稍有不同的锁：
```
typedef struct __lock_t {
    int ticket;
    int turn;
} lock_t;

void lock_init(lock_t *lock) {
    lock->ticket = 0;
    lock->turn = 0;
}

void lock(lock_t *lock) {
    int myturn = FetchAndAdd(&lock->ticket);
    while (lock->turn != myturn)
        ; // spin
}

void unlock(lock_t *lock) {
    lock->turn = lock->turn + 1;
}
```

其中，`ticket`变量表示曾经请求过锁的线程的个数，`turn`变量表示当前（应该）是第几个等待线程持有锁。`lock()`函数中，通过FA指令为本线程获得“排位”（`myturn`），并等待前面的线程执行完毕。`unlock()`函数中，只需简单地将`turn`变量+1（轮到下一个线程了）。

这个实现方法的优点是，不存在饥饿问题，请求同一个锁的线程按请求顺序获得锁。（但是这样会不会有死锁问题？？）

### 用硬件原语和OS的支持实现锁
#### 直接把自旋改为`yield`
代码实现如下：
```
void init() {
    flag = 0;
}

void lock() {
    while (TestAndSet(&flag, 1) == 1)
        yield(); // give up the CPU
}

void unlock() {
    flag = 0;
}
```

得不到锁的线程不再自旋，而是主动放弃控制权。虽然减少了自旋浪费的时间片，不过仍然耗费了一些切换上下文的时间。

#### 利用队列和睡眠实现锁
现在改为由OS控制当前的锁被释放之后，哪一个线程能够获得锁。为此引入睡眠机制：
* `park()`系统调用：将调用线程睡眠
* `unpark(ThreadID)`系统调用：唤醒某个进程

下面是一个使用队列、TS指令、`yield`、自旋和睡眠机制的代码实现：
```
typedef struct __lock_t {
    int flag;
    int guard;
    queue_t *q;
} lock_t;

void lock_init(lock_t *m) {
    m->flag = 0;
    m->guard = 0;
    queue_init(m->q);
}

void lock(lock_t *m) {
    while (TestAndSet(&m->guard, 1) == 1)
        ; //acquire guard lock by spinning
    if (m->flag == 0) {
        m->flag = 1; // lock is acquired
        m->guard = 0;
    } else {
        queue_add(m->q, gettid());
        setpark();  // avoid wakeup/waiting race
        m->guard = 0;
        park();
    }
}

void unlock(lock_t *m) {
    while (TestAndSet(&m->guard, 1) == 1)
        ; //acquire guard lock by spinning
    if (queue_empty(m->q))
        m->flag = 0; // let go of lock; no one wants it
    else
        unpark(queue_remove(m->q)); // hold lock (for next thread!)
    m->guard = 0;
}
```

该锁的内部嵌套了一个用TS指令实现的自旋锁，对应的变量是`guard`，保护的关键区是对锁的内部状态的修改和更新过程。

* `lock()`函数：
  * 首先尝试进入关键区
  * 如果能获得锁，则离开关键区
  * 如果不能获得锁，则把自己加入队列中，离开关键区，然后放弃CPU（注意，如果先放弃CPU，则guard永远不会变成0，别的进程就无法获得和释放锁了）
* `unlock()`函数：
  * 首先尝试进入关键区
  * 如果等待队列为空，则直接放弃锁，并离开关键区
  * 否则从队列中唤醒一个线程（注意，此时`flag`并不会被置为0，因为被唤醒的线程此时回到了`park()`执行完毕的地方，它已经离开了关键区，因此也不应该尝试把`flag`设为1；因此，我们直接把锁交给下一个线程，重甲不进行释放）

另一个可能存在的问题是，如果没有`setpark()`一句，且请求锁的线程在`park()`之前被打断，那么如果此时锁被释放了，重新被唤醒的请求线程会继续执行`park()`，并永远沉睡下去。`setpark()`可以解决这个问题：在调用它之后，如果线程在调用`park()`之前被打断，且其他的线程对它调用了`unpark()`，那么该线程随后对`park()`的调用会立即返回。

#### 一个实际的锁的实现
这个实现利用了Linux中的两个函数：
* `futex_wait(address, expected)`：如果address处的值与expected相等，则睡眠，否则返回
* `futex_wake(address)`：唤醒一个在该地址的队列中等待的线程

这个实现中，一个整数（mutex）用于跟踪锁是否被持有（最高位）和等待该锁的线程数（其他所有位）。所以，如果mutex为负数，则锁被持有。

```
void mutex_lock (int *mutex) {
	int v;
	/* Bit 31 was clear, we got the mutex (this is the fastpath) */
	if (atomic_bit_test_set (mutex, 31) == 0)
		return;
	atomic_increment (mutex);
	while (1) {
		if (atomic_bit_test_set (mutex, 31) == 0) {
			atomic_decrement (mutex);
			return;
		}
		/* We have to wait now. First make sure the futex value
		we are monitoring is truly negative (i.e. locked). */
		v = *mutex;
		if (v >= 0)
			continue;
		futex_wait (mutex, v);
	}
}

void mutex_unlock (int *mutex) {
	/* Adding 0x80000000 to the counter results in 0 if and only if there are not other interested threads */
	if (atomic_add_zero (mutex, 0x80000000))
		return;

	/* There are other threads waiting for this mutex,
	wake one of them up. */
	futex_wake (mutex);
}
```

* `mutex_lock`函数：
  * 如果该锁空闲，则返回
  * 否则等待线程数+1，并开始循环
  * 在每次循环中，如果发现该锁空闲，则获得该锁（使用的是TS指令），等待线程数-1，并返回
  * 如果发现该锁不空闲，在等待之前重新进行一次检查，如果发现锁空闲则继续循环；否则开始睡眠
* `mutex_unlock`函数：
  * 将最高位置0的同时检查是否有其他线程在等待，如果没有，则直接返回
  * 如果有，则唤醒一个等待中的线程

其中，在尝试睡眠的过程中，`futex_wait`函数的特性保证了“wakeup/waiting race”不会发生：首先读出`mutex`处的值，确保此时锁不空闲；即使此时被打断，`mutex`处的值被修改，`futex_wait`也会因为两个值不相等而返回，不会永远进入睡眠状态。

#### 两阶段锁
这个锁的原理是混合了自旋锁和队列睡眠机制。好像没有什么好说的。

## 作业
没有做。
