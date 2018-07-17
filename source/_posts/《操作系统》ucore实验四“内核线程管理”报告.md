---
title: 《操作系统》ucore实验四“内核线程管理”报告
urlname: os-ucore-lab-4-report
toc: true
date: 2018-05-23 10:25:22
updated: 2018-05-23 10:25:22
tags: [OS, ucore]
---

## 实验目的
* 了解内核线程创建/执行的管理过程
* 了解内核线程的切换和基本调度过程

## 实验内容

实验2/3完成了物理和虚拟内存管理，这给创建内核线程（ 内核线程是一种特殊的进程） 打下了提供内存管理的基础。当一个程序加载到内存中运行时，首先通过ucore OS的内存管理子系统分配合适的空间，然后就需要考虑如何分时使用CPU来“并发”执行多个程序，让每个运行的程序（ 这里用线程或进程表示） “感到”它们各自拥有“自己”的CPU。

本次实验将首先接触的是内核线程的管理。内核线程是一种特殊的进程，内核线程与用户进程的区别有两个：
* 内核线程只运行在内核态
* 用户进程会在在用户态和内核态交替运行
* 所有内核线程共用ucore内核内存空间，不需为每个内核线程维护单独的内存空间
* 而用户进程需要维护各自的用户内存空间

相关原理介绍可看附录B：【原理】进程/线程的属性与特征解析。

### 练习0：填写已有实验
本实验依赖实验1/2/3。请把你做的实验1/2/3的代码填入本实验中代码中有“LAB1”,“LAB2”,“LAB3”的注释相应部分。

---

* Lab1：
  * `kdebug.c:print_stackframe`
  * `trap.c:idt_init`
  * `trap.c:trap_dispatch`
* Lab2：
  * `default_pmm.c:default_init`
  * `default_pmm.c:default_init_memmap`
  * `default_pmm.c:default_alloc_pages`
  * `default_pmm.c:default_free_pages`
  * `pmm.c:get_pte`
  * `pmm.c:page_remove_pte`
* Lab3：
  * `vmm.c:do_pgfault`
  * `swap_fifo.c:__fifo_map_swappable`
  * `swap_fifo.c:__fifo_swap_out_victim`

### 练习1：分配并初始化一个进程控制块（需要编码）
alloc_proc函数（位于kern/process/proc.c中）负责分配并返回一个新的struct proc_struct结构，用于存储新建立的内核线程的管理信息。ucore需要对这个结构进行最基本的初始化，你需要完成这个初始化过程。
>【提示】在alloc_proc函数的实现中，需要初始化的proc_struct结构中的成员变量至少包括：state/pid/runs/kstack/need_resched/parent/mm/context/tf/cr3/flags/name。

请在实验报告中简要说明你的设计实现过程。请回答如下问题：
* 请说明`proc_struct`中`struct context context`和`struct trapframe *tf`成员变量含义和在本实验中的作用是啥？（提示通过看代码和编程调试可以判断出来）

---

#### 1.1 具体实现
注释中给出了以下域的说明，其中有些是不需要在这个函数中进行分配的：
* `enum proc_state state`：表示进程状态，在此函数中应赋值为`PROC_UNINIT`，表示该进程的初始化尚未完成（对进程状态的修改在`do_fork`函数的最后，通过调用`sched.c:wakeup_proc`函数完成）
* `int pid`：初始赋值为-1，表示尚未分配（`pid`在`do_fork`函数中通过调用`get_pid`进行分配）
* `int runs`：已运行次数，此处赋值为0
* `uintptr_t kstack`：内核堆栈起始地址，此时堆栈尚未分配，因此置为0；实际在`do_fork`函数中通过调用`setup_kstack`进行分配
* `volatile bool need_resched`：当前进程是否需要调度；初始化为不需要（0）
* `struct proc_struct *parent`：当前进程的父进程，初始化为`NULL`；在`do_fork`中初始化为调用`do_fork`的当前进程
* `struct mm_struct *mm`：内存管理，初始化为`NULL`；在`do_fork`中通过调用`copy_mm`进行初始化（虽然实际上直接使用了内核的mm，因为是内核线程）
* `struct context context`：在Lab5中发现context需要清零，但在此处似乎不初始化也能正常运行；在`do_fork`中通过调用`copy_thread`函数进行初始化
* `struct trapframe *tf`：当前的中断帧，初始化为`NULL`；在`do_fork`中通过调用`copy_thread`函数进行初始化
* `uintptr_t cr3`：当前进程的页表基地址；直接初始化为kernel的页表基地址`boot_cr3`
* `uint32_t flags`：当前进程属性，因为是初始化，所以置为0了
* `char name[PROC_NAME_LEN + 1]`：进程的名称，此处初始化似乎不是很重要，不过还是清零了
```
// alloc_proc - alloc a proc_struct and init all fields of proc_struct
static struct proc_struct *
alloc_proc(void) {
    struct proc_struct *proc = kmalloc(sizeof(struct proc_struct));
    if (proc != NULL) {
    //LAB4:EXERCISE1 YOUR CODE
    	proc->state = PROC_UNINIT;  // 正在创建和初始化状态中
    	proc->pid = -1;  // 参考了答案：未初始化的进程id为-1
    	proc->runs = 0;  // 还没有运行过
    	proc->kstack = 0;  // 参考了答案：初始化内核堆栈似乎是在do_fork()中进行的？
    	proc->need_resched = 0;  // 初始化为不需要调度
    	proc->parent = NULL;
    	proc->mm = NULL;  // 之后也不会分配，因为都是内核态线程，所以直接使用内核的mm
    	memset(&(proc->context), 0, sizeof(struct context));  // 在LAB5中发现，忘了清零context了
    	// proc->tf = kmalloc(sizeof(struct trapframe));  // tf似乎不需要在此处设置
    	proc->cr3 = boot_cr3;  // 参考了答案：内核态线程不需要分配新的页表地址；这对于正确执行是必需的
    	proc->flags = 0;  // 参考了答案：标志位置为0
    	memset(proc->name, 0, PROC_NAME_LEN);  // 参考了答案：将进程名清零，不过不是必需的
    }
    return proc;
}
```

#### 1.2 `context`和`trapframe`的含义和用途

实验指导书中指出：
* `context`：进程的上下文，用于进程切换（参见`switch.S`）。在 uCore中，所有的进程在内核中也是相对独立的（例如独立的内核堆栈以及上下文等等）。使用`context`保存寄存器的目的就在于在内核态中能够进行上下文之间的切换。实际利用`context`进行上下文切换的函数是`kern/process/switch.S:switch_to`
* `tf`：中断帧的指针，总是指向内核栈的某个位置：当进程从用户空间跳到内核空间时，中断帧记录了进程在被中断前的状态。当内核需要跳回用户空间时，需要调整中断帧以恢复让进程继续执行的各寄存器值。除此之外，uCore内核允许嵌套中断。因此为了保证嵌套中断发生时`tf`总是能够指向当前的`trapframe`，uCore 在内核栈上维护了`tf`的链，可以参考`trap.c::trap`函数做进一步的了解。

经过阅读代码，我认为，`switch_to`的主要工作是把被切换的进程的各个通用寄存器（eip、esp、ebx、ecx、edx、esi、edi、ebp，但不包括段寄存器，因为kernel进程使用的段是相同的）保存到进程的`context`结构中，然后加载即将开始运行的进程的`context`结构中保存的通用寄存器。而`trapframe`就是我们在Lab1中已经了解的中断保存现场。对于内核线程，`trapframe`的意义似乎并不大，因为不需要进行用户空间到内核空间的切换。

---

今天我的某个叫wenj的同学指出了一个很有趣的问题，这使得我重新翻出了实验报告：`context`和`trapframe`中为何都存储了EIP？这两种结构的功能是否重复了？翻了翻实验指导书，发现其实这个问题已经有比较明确的解答了：`trapframe`一般来说是用户态切换到内核态用的，而`context`是内核态自己切换上下文用的（因为特权级不变，所以不需要存储页表基地址、段寄存器等内容）；不过用户态跳转到内核态的时候也需要保存`context`中的通用寄存器，因为`trapframe`不存通用寄存器。

以及，lab4中构建进程的过程是这样的：
* “硬”构造出第一个内核线程idleproc
* 调用`do_fork`函数，fork idleproc，生成initproc

事实上initproc返回时会假装自己是通过系统调用`do_fork`生成的，所以返回过程会比较复杂：

> uCore会执行进程切换，让initproc执行。在对initproc进行初始化时，设置了`initproc->context.eip = (uintptr_t)forkret`，这样，当执行`switch_to`函数并返回后，initproc将执行其实际上的执行入口地址forkret。而forkret会调用位于kern/trap/trapentry.S中的forkrets函数执行，具体代码如下：

```
.globl __trapret
__trapret:
# restore registers from stack
popal
# restore %ds and %es
popl %es
popl %ds
# get rid of the trap number and error code
addl $0x8, %esp
iret
.globl forkrets
forkrets:
# set stack to this new process's trapframe
movl 4(%esp), %esp //把esp指向当前进程的中断帧
jmp __trapret
```

> 可以看出，forkrets函数首先把esp指向当前进程的中断帧，从_trapret开始执行到iret前，esp指向了current->tf.tf_eip，而如果此时执行的是initproc，则current->tf.tf_eip=kernel_thread_entry，initproc->tf.tf_cs = KERNEL_CS，所以当执行完iret后，就开始在内核中执行kernel_thread_entry函数了，而initproc->tf.tf_regs.reg_ebx = init_main，所以在kernl_thread_entry中执行“call %ebx”后，就开始执行initproc的主体了。Initprocde的主体函数很简单就是输出一段字符串，然后就返回到kernel_tread_entry函数，并进一步调用do_exit执行退出操作了。本来do_exit应该完成一些资源回收工作等，但这些不是实验四涉及的，而是由后续的实验来完成。至此，实验四中的主要工作描述完毕。

### 练习2：为新创建的内核线程分配资源（需要编码）
创建一个内核线程需要分配和设置好很多资源。kernel_thread函数通过调用do_fork函数完成具体内核线程的创建工作。do_kernel函数会调用alloc_proc函数来分配并初始化一个进程控制块，但alloc_proc只是找到了一小块内存用以记录进程的必要信息，并没有实际分配这些资源。ucore一般通过do_fork实际创建新的内核线程。do_fork的作用是，创建当前内核线程的一个副本，它们的执行上下文、代码、数据都一样，但是存储位置不同。在这个过程中，需要给新内核线程分配资源，并且复制原进程的状态。你需要完成在kern/process/proc.c中的do_fork函数中的处理过程。它的大致执行步骤包括：
* 调用alloc_proc，首先获得一块用户信息块。
* 为进程分配一个内核栈。
* 复制原进程的内存管理信息到新进程（但内核线程不必做此事）
* 复制原进程上下文到新进程
* 将新进程添加到进程列表
* 唤醒新进程
* 返回新进程号
请在实验报告中简要说明你的设计实现过程。请回答如下问题：
* 请说明ucore是否做到给每个新fork的线程一个唯一的id？请说明你的分析和理由。

---

#### 2.1 具体代码实现

函数的大致执行步骤与题目中列出的相同。

```
int
do_fork(uint32_t clone_flags, uintptr_t stack, struct trapframe *tf) {
    int ret = -E_NO_FREE_PROC;
    struct proc_struct *proc;
    if (nr_process >= MAX_PROCESS) {
        goto fork_out;
    }
    ret = -E_NO_MEM;
    //    1. call alloc_proc to allocate a proc_struct
    proc = alloc_proc();
    if (proc == NULL) {  // 参考答案添加了错误处理
    	goto fork_out;
    }
    proc->parent = current;  // 参考了答案
    //    2. call setup_kstack to allocate a kernel stack for child process
    if (setup_kstack(proc) != 0) {  // 参考答案添加了错误处理
    	goto bad_fork_cleanup_proc;
    }
    //    3. call copy_mm to dup OR share mm according clone_flag
    // CLONE_VM表示分享；实际上因为都在内核态所以什么都没做，只是assert NULL了
    if (copy_mm(clone_flags, proc) != 0) {  // 参考答案添加了错误处理
    	goto bad_fork_cleanup_kstack;
    }
    //    4. call copy_thread to setup tf & context in proc_struct
    copy_thread(proc, stack, tf);
    //    5. insert proc_struct into hash_list && proc_list
    // 参考了答案：关中断的原因是，进程号要求唯一性，此操作需要为原子操作，防止被打断而重复添加
    // 所以参考答案是很有必要的。但是我认为在实验指导书中也应该说明一下。
    bool intr_flag;
	local_intr_save(intr_flag);
	{
    	proc->pid = get_pid();
		hash_proc(proc);
		nr_process++;  // 参考答案添加在此处（我本来以为用了get_pid()就不需要这句了
		list_add_before(&proc_list, &proc->list_link);
	}
	local_intr_restore(intr_flag);
    //    6. call wakeup_proc to make the new child process RUNNABLE
    wakeup_proc(proc);
    //    7. set ret vaule using child proc's pid
    ret = proc->pid;
fork_out:
    return ret;

bad_fork_cleanup_kstack:
    put_kstack(proc);
bad_fork_cleanup_proc:
    kfree(proc);
    goto fork_out;
}
```

#### 2.2 能否为每个新线程赋值唯一ID

阅读代码可以得知，`idleproc`的PID是由`proc_init`函数设置的，但`initproc`的PID应该怎么设置呢？我的解决方法是把分配PID的过程移到`do_fork`函数中，这样至少能通过测试了。参考答案的做法也类似，不过在分配PID和将进程插入队列的过程中进行了关中断处理，保证原子操作。

PID的唯一性是通过关中断和`get_pid`函数保证的，该函数查看当前的全部进程，在不发生中断的情况下，可以保证新分配的PID与之前的PID是不冲突的。
```
// get_pid - alloc a unique pid for process
static int
get_pid(void) {
    static_assert(MAX_PID > MAX_PROCESS);
    struct proc_struct *proc;
    list_entry_t *list = &proc_list, *le;
    static int next_safe = MAX_PID, last_pid = MAX_PID;
    if (++ last_pid >= MAX_PID) {
        last_pid = 1;
        goto inside;
    }
    if (last_pid >= next_safe) {
    inside:
        next_safe = MAX_PID;
    repeat:
        le = list;
        while ((le = list_next(le)) != list) {
            proc = le2proc(le, list_link);
            if (proc->pid == last_pid) {
                if (++ last_pid >= next_safe) {
                    if (last_pid >= MAX_PID) {
                        last_pid = 1;
                    }
                    next_safe = MAX_PID;
                    goto repeat;
                }
            }
            else if (proc->pid > last_pid && next_safe > proc->pid) {
                next_safe = proc->pid;
            }
        }
    }
    return last_pid;
}
```

### 练习3：阅读代码，理解 proc_run 函数和它调用的函数如何完成进程切换的。（无编码工作）
请在实验报告中简要说明你对proc_run函数的分析。并回答如下问题：
* 在本实验的执行过程中，创建且运行了几个内核线程？
* 语句`local_intr_save(intr_flag);....local_intr_restore(intr_flag);`在这里有何作用？请说明理由。

完成代码编写后，编译并运行代码：`make qemu`

如果可以得到如附录A所示的显示内容（仅供参考，不是标准答案输出），则基本正确。

---

#### 3.1 内核线程
分析一下`proc_init`函数的调用过程：
* 初始化`proc_list`和`hash_list`
* 调用`alloc_proc`函数分配`idleproc`所需的TCB块，检验是否分配成功
* 对`idleproc`进行基本设置（所以`alloc_proc`函数其实不需要干啥？）：
  * PID=0
  * state=PROC_RUNNABLE
  * kstack=bootstack
  * need_resched=1
  * name="idle"
* 将`current`变量置为`idleproc`
* 调用`kernel_thread`函数，用`init_main`函数创建一个内核线程
  * 创建所需的`trapframe`
  * 调用`do_fork`函数，创建新进程
    * 调用`alloc_proc`，首先获得一块用户信息块。
    * 为进程分配一个内核栈。
    * 复制原进程的内存管理信息到新进程（但内核线程不必做此事）
    * 复制原进程上下文到新进程
    * 将新进程添加到进程列表
    * 唤醒新进程
    * 返回新进程号
* 验证创建`initproc`线程成功（PID不为0）

`proc_init`函数是由`kern_init`函数调用的。在`kern_init`完成其余初始化之后，它调用`cpu_idle`函数，使得当前的`idle_proc`进程让出控制权，交给`initproc`线程，进行上下文的切换；执行完之后，回到`kernel_thread_entry`，退出。

由以上分析可知，在本实验的执行过程中，一共只创建了两个内核线程（`idleproc`和`initproc`）。

#### 3.2 关中断的必要性
以下回答来自[Piazza](https://piazza.com/class/i5j09fnsl7k5x0?cid=309)：
> 由于进程号要求唯一性，进程号分配时可能需要查看进程列表中全部进程以避免发生冲突。若进程号已分配而进程尚未添加进进程列表时被中断，则该进程号可能会被重复分配，故进程号分配与进程添加应为原子操作。因而在进行上述操作时需关闭中断。

### 3.3 `proc_run`函数如何完成进程切换
对进程切换的控制是通过`sched.c:schedule`函数完成的。一旦当前进程的`need_resched`变量被置为1，就调用`schedule`函数选择下一个要运行的进程，调用`proc_run`函数开始运行。
```
void
schedule(void) {
    bool intr_flag;
    list_entry_t *le, *last;
    struct proc_struct *next = NULL;
    local_intr_save(intr_flag);
    {
        current->need_resched = 0;
        last = (current == idleproc) ? &proc_list : &(current->list_link);
        le = last;
        do {
            if ((le = list_next(le)) != &proc_list) {
                next = le2proc(le, list_link);
                if (next->state == PROC_RUNNABLE) {
                    break;
                }
            }
        } while (le != last);
        if (next == NULL || next->state != PROC_RUNNABLE) {
            next = idleproc;
        }
        next->runs ++;
        if (next != current) {
            proc_run(next);
        }
    }
    local_intr_restore(intr_flag);
}
```

然后就进入了`proc_run`函数。
```
void
proc_run(struct proc_struct *proc) {
    if (proc != current) {
        bool intr_flag;
        struct proc_struct *prev = current, *next = proc;
        local_intr_save(intr_flag);
        {
            current = proc;
            load_esp0(next->kstack + KSTACKSIZE);
            lcr3(next->cr3);
            switch_to(&(prev->context), &(next->context));
        }
        local_intr_restore(intr_flag);
    }
}
```
代码的执行过程如下：
1. 关闭中断，保证原子操作
2. 调用`load_esp0`函数，设置任务状态段`ts`中特权态0下的栈顶指针`esp0`为要切换到的内核线程的内核栈的栈顶，即`next->kstack + KSTACKSIZE`（建立指针的目的是，进行特权态切换时能够正确定位处于特权态0时进程的内核栈的栈顶）
3. 设置CR3寄存器的值为要切换到的内核线程的页目录表起始地址，这实际上是完成进程间的页表切换，不过在内核中的内存切换下其实用不到
4. 由 `switch_to`函数完成具体的两个线程的执行现场切换，即切换各个寄存器，当`switch_to`函数执行完`ret`指令后，就切换到 `initproc`执行了

在切换现场时，倒数第二条汇编指令`pushl 0(%eax)`其实把`context`中保存的下一个进程要执行的指令地址`context.eip`放到了堆栈顶，这样接下来执行最后一条指令`ret`时，会把栈顶的内容赋值给EIP寄存器，这样就切换到下一个进程执行了。

事实上，在对initproc进行初始化时，设置了`initproc->context.eip = (uintptr_t)forkret`（见`copy_thread`函数），这样，当执行`switch_to`函数并返回后，将进入实际的执行入口地址`forkret`。而`forkret`会调用位于`kern/trap/trapentry.`S中的`forkrets`（参数是切换之后的进程的中断帧地址，该地址位于内核中线程对应的栈中，如果我没理解错的话）。

```
.globl forkrets
forkrets:
    # set stack to this new process's trapframe
    movl 4(%esp), %esp
    jmp __trapret
```

可以看出，`forkrets`函数首先把`esp`指向当前进程的中断帧，然后跳转到`__trapret`，从中恢复中断帧的各个寄存器。这些寄存器是在`kernel_thread`函数中设置的，包括：
* `tf.tf_cs = KERNEL_CS`：和内核使用同一代码段（这是合理的，因为`initproc`对应的代码也是内核的一部分）
* `tf.tf_ds = tf.tf_es = tf.tf_ss = KERNEL_DS`：和内核使用同一数据（堆栈）段
* `tf.tf_regs.reg_ebx = (uint32_t)fn`：寄存器中的`ebx`为函数起始地址
* `tf.tf_regs.reg_edx = (uint32_t)arg`：寄存器中的`edx`指向函数参数地址
* `tf.tf_eip = (uint32_t)kernel_thread_entry`：中断后的实际起始地址是`kernel_thread_entry`

```
.globl __trapret
__trapret:
    # restore registers from stack
    popal

    # restore %ds, %es, %fs and %gs
    popl %gs
    popl %fs
    popl %es
    popl %ds

    # get rid of the trap number and error code
    addl $0x8, %esp
    iret
```

对于`initproc`，`current->tf.tf_eip=kernel_thread_entry`，`initproc->tf.tf_cs = KERNEL_CS`，所以执行完`iret`后，就开始在内核中执行`kernel_thread_entry`函数了。

```
.text
.globl kernel_thread_entry
kernel_thread_entry:        # void kernel_thread(void)

    pushl %edx              # push arg
    call *%ebx              # call fn

    pushl %eax              # save the return value of fn(arg)
    call do_exit            # call do_exit to terminate current thread
```

首先把进程的参数入栈，然后调用起始地址（现在是实际的起始地址了，对于`initproc`，这个起始地址就是`init_main`函数的开头）。所以执行`call %ebx`后，就开始执行`initproc`的主体了。

执行结束后，返回到`kernel_tread_entry`函数，它会进一步调用`proc.c:do_exit`函数，执行退出操作。目前这个函数除了打印一点字符串没有做别的工作。

### 扩展练习Challenge：实现支持任意大小的内存分配算法
这不是本实验的内容，其实是上一次实验内存的扩展，但考虑到现在的slab算法比较复杂，有必要实现一个比较简单的任意大小内存分配算法。可参考本实验中的slab如何调用基于页的内存分配算法（注意，不是要你关注slab的具体实现）来实现first-fit/best-fit/worst-fit/buddy等支持任意大小的内存分配算法。

【注意】下面是相关的Linux实现文档，供参考
SLOB
http://en.wikipedia.org/wiki/SLOB http://lwn.net/Articles/157944/
SLAB
https://www.ibm.com/developerworks/cn/linux/l-linux-slab-allocator/

---

没写。

## 分析参考答案
* `alloc_proc`：和参考答案的实现类似，参考了答案中的一些我忘了写的部分，如初始化内核堆栈和将context清零
* `do_fork`：参考了答案，添加了一些错误处理，以及分配pid时关中断

## 知识点
* 进程状态
* 进程控制块
* 内核栈和用户栈
