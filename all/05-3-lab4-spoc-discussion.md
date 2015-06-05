# lab4 spoc 思考题

- 有"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的ucore_code和os_exercises的git repo上。

## 个人思考题

### 总体介绍

(1) ucore的线程控制块数据结构是什么？

### 关键数据结构

(2) 如何知道ucore的两个线程同在一个进程？

(3) context和trapframe分别在什么时候用到？

(4) 用户态或内核态下的中断处理有什么区别？在trapframe中有什么体现？

### 执行流程

(5) do_fork中的内核线程执行的第一条指令是什么？它是如何过渡到内核线程对应的函数的？
```
tf.tf_eip = (uint32_t) kernel_thread_entry;
/kern-ucore/arch/i386/init/entry.S
/kern/process/entry.S
```

(6)内核线程的堆栈初始化在哪？
```
tf和context中的esp
```

(7)fork()父子进程的返回值是不同的。这在源代码中的体现中哪？

(8)内核线程initproc的第一次执行流程是什么样的？能跟踪出来吗？

## 小组练习与思考题

(1)(spoc) 理解内核线程的生命周期。

> 需写练习报告和简单编码，完成后放到git server 对应的git repo中

### 掌握知识点
1. 内核线程的启动、运行、就绪、等待、退出
2. 内核线程的管理与简单调度
3. 内核线程的切换过程

### 练习用的[lab4 spoc exercise project source code](https://github.com/chyyuu/ucore_lab/tree/master/related_info/lab4/lab4-spoc-discuss)


请完成如下练习，完成代码填写，并形成spoc练习报告

### 1. 分析并描述创建分配进程的过程

> 注意 state、pid、cr3，context，trapframe的含义

- 调用kernel_thread()函数，在此函数中，先开辟一段空间给进程的trapframe,然后对trapframe中应保存的寄存器信息进行初始化。然后调用do_fork（）函数。    
- 在do_fork()函数中先对该进程创建一个PCB（进程控制块poc_struct) ，PCB中有该进程的一些状态信息和空间信息，其中state初始化为PROC_UNINIT，进程号pid为-1，cr3为内核堆栈的cr3（cr3为进程的虚拟地址空间一级页表的初始位置）。  
- 对该进（线）程的pid赋值，创建堆栈空间，调用copy_thread()函数对该进程的trapframe保存栈顶的寄存器进行赋值，同时也进行上下文context的复制（context是进程的上下文，表示进程运行过程中的状态，主要是一些寄存器的值）。  
- 此时进程以及创建好，加入进程队列，并调用wakeup函数将进程的state设为RUNNABLE，在之后的schedule函数调用时可以转换状态。

### 练习2：分析并描述新创建的内核线程是如何分配资源的

> 注意 理解对kstack, trapframe, context等的初始化

- 在内核线程创建时进行初始化的过程中进行的资源分配
- 在do_fork函数中调用`setup_kstack`函数进行堆栈的空间的创建  
- context是TCB创建过程中创建的，主要保存进程运行过程中的寄存器信息。如eip、esp以及通用的ebx、ecx等等,是在do_fork函数中的copy_thread()函数里进行创建的。  
- trapframe是进程中断的时候保存到内核堆栈里的数据结构，保存了当前被打断时候一些信息，以便于后续能够恢复。进程中trapframe的建立是在kernel_thread函数的开始就完成的。主要是分配空间、以及对该进程函数入口地址、段寄存器和eip等值的设置。

当前进程中唯一，操作系统的整个生命周期不唯一，在get_pid中会循环使用pid，耗尽会等待

### 练习3：阅读代码，在现有基础上再增加一个内核线程，并通过增加cprintf函数到ucore代码中
能够把内核线程的生命周期和调度动态执行过程完整地展现出来

### 练习4 （非必须，有空就做）：增加可以睡眠的内核线程，睡眠的条件和唤醒的条件可自行设计，并给出测试用例，并在spoc练习报告中给出设计实现说明

### 扩展练习1: 进一步裁剪本练习中的代码，比如去掉页表的管理，只保留段机制，中断，内核线程切换，print功能。看看代码规模会小到什么程度。
