---
title: 深入理解Linux系统 —— 进程（上）
date: 2021-08-05 13:01:20
index_img: /img/OS/banner.png
category: [OS]
tags: [kernel, process]
math: true
---



## 一、进程、轻量级进程和线程

### 1.1 进程 - Process

​	进程无疑是现代操作系统中最为成功的概念之一，其是程序执行时的实例抽象，现代操作系统以进程为抽象概念，使得每一个进程在运行时都好似独立的拥有CPU、内存等硬件。使得软件开发者不用过多关心底层硬件的调度和实现，便能够在较高的层次上进行程序开发。

- **概念：**
  - **程序角度：**程序运行时的实例
  - **内核观点：**分配系统资源（CPU时间、内存）的实体

**父子进程**

​    当一个子进程被创建时，其拥有和父进程相同的代码段、数据段和用户堆栈，就像父进程把自己克隆了一遍。事实上，父进程只复制了自己的PCB *(Process Control Block)*。而代码段，数据段和用户堆栈内存空间并没有复制一份，而是与子进程共享。只有当子进程在运行中出现写操作时，才会产生中断，并为子进程分配内存空间。即**写时复制**

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gt3l9prc3lj30i10b7js6.jpg" style="zoom:80%;" />

### 1.2 轻量级进程 - Lightweight Process

​	Linux使用了轻量级进程，对多线程应用提供了更好的支持，其户型可以共享一些资源。例如地址空间和打开的文件，其中一个修改共享资源，另一个立即查看并且同步更新。

​	不过这需要依赖进程间通信（IPC）机制，肯定会比线程更慢

### 1.3 线程 - Thread

​	线程是操作系统进行运算调度的最小单位，一个进程中可以有多个线程，其被共同维护在一个线程池中进行统一调度。可用线程数量应该取决于可用的并发处理器、处理器内核、内存、网络sockets等的数量。任务调度以执行线程的常见方法是使用同步队列，称作任务队列。池中的线程等待队列中的任务，并把执行完的任务放入完成队列中。	

​	线程池一般使用生产者消费者模式或领导者追随者模式

- **生产者消费者：**即一个简单buffer，主线程存入工作队列，工作线程从工作队列取出任务进入工作状态，如果工作队列为空则挂起。
- **领导者追随者模式：**当事件到达，领导者线程将自身变为工作状态处理事件，并从追随者中选择一个来当继任领导者，处理完后将自身置为追随者模式。其避免了线程之间的任务数据交换，提高了性能

## 二、进程描述符

​	进程描述符顾名思义就是用来描述一个进程的，从其数据结构中我们可以了解该进程有哪些权限、允许访问哪些文件等等。

​	其数据结构是 task_struct 类型，其不仅包含了许多进程属性，还包含了一些指向另外数据结构的指针。

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gt3mn4emosj30u00udtan.jpg" style="zoom:50%;" />

​	  下面我们讨论进程的状态和进程的父|子之间关系

### 2.1 进程状态

​	进程描述符中的state描述进程所处的状态，其由一组标志组成，并且是互斥的

**可运行状态 TASK_RUNNING**

​	进程要么在CPU上执行，要么准备执行

**可中断的等待状态 TASK_INTERRUPTIBLE**

​	进程被挂起，直到某条件为真。产生一个硬件中断，释放系统资源，或通过信号唤醒进程

**不可中断的等待状态 TASK_UNINTERRUPTIBLE**

​	即不可被信号中断，常常出现在需要原子性操作的过程（例如进程打开设备文件，相应的设备驱动开始探查硬件设备时）

**暂停状态 TASK_STOPPED**

​	收到 SIGSTOP \ SIGTSTP \ SIGTTIN \ SIGTTOU 时

**跟踪状态 TASK_TRACED**

​	进程被debugger程序暂停，任何信号都可以把进程置为 TASK_TRACED 状态

*另外两种状态即可在state也可在exit_state中*

**僵死状态 EXIT_ZOMBIE**

 	子进程结束但还未被父进程回收

**僵死撤销状态 EXIT_DEAD**

​	已经被父进程waitpid回收，在系统回收过程中，而防止其他进程调用wait产生冲突而置于 EXIT_DEAD 状态



### 2.2 进程标识符 Process ID

​	内核对进程的大部分引用是通过进程描述符指针完成的，而类UNIX操作系统允许用户采用一个进程标识符 pid 来标识进程，PID按顺序编号且进程数量有一定上限，其上限为 4194303 （PID_MAX_DEFAULT - 1）PID编号循环使用，内核通过管理一个 pidmap_array 位图来表示已经分配的PID和闲置的PID号。

​	很自然地我们知道不同的进程有着不同的pid号，而同一进程的线程的pid号相同，这是在POSIX 1003.1c 标准中就规定了的

为此引入了线程组的表示，一个线程组中的所有线程和该线程组中的领头线程拥有相同的PID，其被存入了**tgid字段中**，我们调用getpid()返回的是当前进程的 tgid值而不是pid的值。由于绝大多数进程都属于一个线程组，包含单一成员，线程组的tgid值与进程pid值相同，因此getpid()系统调用对此类进程所起到的作用和一般进程相同。



### 2.3 内核栈与thread_info数据结构

​	每个进程都有一个独立的内核栈，其位置被进程的task_struct中的stack指针所标记，长得像下图。内核栈和thread_info存放在连续的页框中，内核栈从高地址向低地址增长，底部存放thread_info数据结构。

**结构大小：**一般来说这样一个结构所占为一个页框，在80x86结构中其占据两个页框即32位系统中的8kb，但如果当内存碎片较多时，可能无法找到这样的连续两个页框，导致效率低下。

**指针关系：**其中task_struct的thread_info指针指向，内核态下esp栈指针指向内核栈栈顶，thread_info中的task指针指向进程的task_stuct

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gt5qzama9uj319z0u0q5n.jpg" style="zoom:45%;" />

**语言实现：**

C语言用以下一个简单的union结构来实现上述数据结构

```c
// 80x86 32位系统
union thread_union {
  struct thread_info thread_info
  unsigned long stack[2048]	/* 对4k的栈数组下标 1024*/
}
```

内核使用alloc_thread_info和free_thread_info宏分配和释放储存thread_info结构和内核栈的内存区



**效率考量：**从效率角度看，所说的thread_info结构与内核栈连续存放的好处是我们可以轻松的通过栈指针esp寄存器的值获得当前CPU上正在运行进程的thread_info结构的地址。其根据thread_union结构的大小，屏蔽掉esp低12位或13位的地址来获得thread_info结构的基地址，调用current_thread_info()产生以下汇编代码

```assembly
movl $0xffffe000, %ecx /*或是用于4k堆栈的0xfffff000*/
andl %esp, %ecx
movl %ecx, p
```

执行完后p就包含thread_info结构的指针，进而我们可以通过thread_info结构中的task指针获得进程描述符task_struct数据结构的地址

调用 macro current 其等价于 current_thread_info()->task，产生如下汇编指令

```assembly
movl $0xffffe000, %ecx /*或者是用于4k堆栈的0xfffff000*/
andl %esp, %ecx
movl (%ecx), p
```

因为task在thread_info结构中偏移量为0，即储存在thread_info中的基地址，所以我们直接引用内存 (%ecx) 便可得到进程描述符地址



### 2.4 进程链表

​	Linux操作系统通过进程链表把所有进程描述符连接起来，进程链表是一个循环双向链表，在linux内核中体现为list_head数据结构。每一个进程的task_struct结构都包含一个list_head类型的tasks字段，其prev前向指针和next后向指针分别指向前面和后面的的进程描述符数据结构

​	其头部是 init_task 描述符，即pid=0的0号进程，或swapper进程中的进程描述符，所以init_task中的prev指针指向最后插入 的进程描述符。

**TASK_RUNNING 状态的进程链表：**

​	内核寻找新进程在CPU上运行时，必须保证其是可运行的，回忆我们前面说到过的进程状态，其必须是TASK_RUNNING状态的进程。

**早期实现：**早期Linux将所有的可运行进程维护在一个运行队列的链表中，但由于按优先级排序维护的开销过大，因此每次选择最优可运行进程时都必须遍历整个链表，效率较低。

​	自Kernel 2.6版本开始，其做出来一些变动，使可以在固定时间内找出最优可运行进程，其方法是为每个优先级都建立一个可运行进程链表，list_head 优先级在 task_struct 中北 run_list 字段所标识，共分为140个list_head，被单独的一个prio_array_t数据结构来实现

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gt5sf9zvmqj31jg0dwmzf.jpg" style="zoom:50%;" />

通过 enqueue_task(p, array) 和 dequeue_task(p, array) 来从运行队列的链表中添加或删除 pd



### 2.5 进程间的关系

​	进程fork出来的进程和原进程具有父子关系，如果一个进程创建多个进程，那么子进程之间具有兄弟关系。

​	**进程0和进程1由内核创建，进程1是所有进程的祖先。**	<img src="https://tva1.sinaimg.cn/large/008i3skNly1gt5sj34gm9j31iq0t6juf.jpg" style="zoom:50%;" />



### 2.6 PIDHash表

​	在某些情况下，内核必须通过进程pid来索引其对应的进程描述符指针，其通过pidhash数据结构实现，即是一个pid与描述符指针对应的散列表。总共有四个。

![](https://tva1.sinaimg.cn/large/008i3skNly1gt5tkx7g64j61iu0esmzj02.jpg)

在Linux中很多散列函数都使用 hash_long()，在32位体系结构中其基本等价于

```c
unsigned long hash_long(unsigned long val, unsigned int bits) {
  unsigned long hash = val * 0x93270001UL
  return hash >> (32 - bits);
}
```

其原理是通过一个数乘大数 0x93270001 (= 2654404609) 溢出，把保留在32位中的值的结果作为模数的操作。

这个大数 0x93270001 是最接近与 2 的 32 次方黄金分割位置的一个素数。

其等于 $2^{31}+2^{29}+2^{25}+2^{22}-2^{19}-2^{16}+1$

利用散列表避免了直接映射带来的储存浪费，因为毕竟大多数时候系统中的进程数量都远远小于最大进程数量。对于 Hash 冲突的情况，Linux内核使用拉链法，且溢出链表为双向链表

<img src="https://tva1.sinaimg.cn/large/008i3skNly1gt5ttu88o4j30wa0u0q4z.jpg" style="zoom:50%;" />

Linux内核进程详解的上半部分就讲到这，下节我们来讲进程的context switch、创建和撤销。

## 参考文献

[1] Daniel P. Bovet, Marco Cesati Understanding the Linux Kernel, 3rd Edition

