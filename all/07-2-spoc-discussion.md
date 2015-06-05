# 同步互斥(lec 18) spoc 思考题


- 有"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的ucore_code和os_exercises的git repo上。

## 个人思考题

### 基本理解
 - 什么是信号量？它与软件同步方法的区别在什么地方？
 - 什么是自旋锁？它为什么无法按先来先服务方式使用资源？
 - 下面是一种P操作的实现伪码。它能按FIFO顺序进行信号量申请吗？
```
 while (s.count == 0) {  //没有可用资源时，进入挂起状态；
        调用进程进入等待队列s.queue;
        阻塞调用进程;
}
s.count--;              //有可用资源，占用该资源； 
```

> 参考回答： 它的问题是，不能按FIFO进行信号量申请。
> 它的一种出错的情况
```
一个线程A调用P原语时，由于线程B正在使用该信号量而进入阻塞状态；注意，这时value的值为0。
线程B放弃信号量的使用，线程A被唤醒而进入就绪状态，但没有立即进入运行状态；注意，这里value为1。
在线程A处于就绪状态时，处理机正在执行线程C的代码；线程C这时也正好调用P原语访问同一个信号量，并得到使用权。注意，这时value又变回0。
线程A进入运行状态后，重新检查value的值，条件不成立，又一次进入阻塞状态。
至此，线程C比线程A后调用P原语，但线程C比线程A先得到信号量。
```

### 信号量使用

 - 什么是条件同步？如何使用信号量来实现条件同步？
 - 什么是生产者-消费者问题？
 - 为什么在生产者-消费者问题中先申请互斥信息量会导致死锁？

### 管程

 - 管程的组成包括哪几部分？入口队列和条件变量等待队列的作用是什么？
 - 为什么用管程实现的生产者-消费者问题中，可以在进入管程后才判断缓冲区的状态？
 - 请描述管程条件变量的两种释放处理方式的区别是什么？条件判断中while和if是如何影响释放处理中的顺序的？

### 哲学家就餐问题

 - 哲学家就餐问题的方案2和方案3的性能有什么区别？可以进一步提高效率吗？

### 读者-写者问题

 - 在读者-写者问题的读者优先和写者优先在行为上有什么不同？
 - 在读者-写者问题的读者优先实现中优先于读者到达的写者在什么地方等待？
 
## 小组思考题

1. （spoc） 每人用python threading机制用信号量和条件变量两种手段分别实现[47个同步问题](07-2-spoc-pv-problems.md)中的一题。向勇老师的班级从前往后，陈渝老师的班级从后往前。请先理解[]python threading 机制的介绍和实例](https://github.com/chyyuu/ucore_lab/tree/master/related_info/lab7/semaphore_condition)

进程同步问题：生产者-消费者-扩展三。

```
#include <stdio.h>
#include <proc.h>
#include <sem.h>
#include <monitor.h>
#include <assert.h>

#define P 4 // 生产者数目
#define B 3 // 既是生产者又是消费者数目
#define C 2 // 消费者数目

#define IP 2 // 每个生产者的物品数
#define IB 4 // 每个既是生产者又是消费者的物品数
#define IC 4 // 每个消费者的物品数

#define BUFFERSIZE 2
#define SHORTSLEEPTIME 10
#define LONGSLEEPTIME 80

//---------- Producer-consumer problem using semaphore ----------------------

semaphore_t full;
semaphore_t empty;
semaphore_t mutex;
struct proc_struct *producer_proc_sema[P];
struct proc_struct *consumer_proc_sema[C];
struct proc_struct *both_proc_sema[B];

int s_head = 0, s_tail = 0;

int producer_using_semaphore(void * arg)
{
	int i = (int)arg, iter = 0;
    cprintf("I am No.%d producer_sema\n", i);
    while (iter++ < IP)
    {
    	cprintf("Iter %d, No.%d producer_sema prepares to produce \n", iter, i);
		do_sleep(SHORTSLEEPTIME);
		down(&empty);
		cprintf("Iter %d, No.%d producer_sema gets a vacancy\n", iter, i);
		down(&mutex);
		cprintf("Iter %d, No.%d producer_sema produces item %d\n", iter, i, s_head);
		s_head = (s_head + 1) % (BUFFERSIZE + 1);
		up(&mutex);
		up(&full);
		if (iter < IP / 2)
			do_sleep(SHORTSLEEPTIME);
		else
			do_sleep(LONGSLEEPTIME);
	}
    cprintf("No.%d producer_sema quit\n", i);
    return 0;
}

int consumer_using_semaphore(void * arg)
{
	int i = (int)arg, iter = 0;
    cprintf("I am No.%d consumer_sema\n", i);
    while (iter++ < IC)
    {
    	cprintf("Iter %d, No.%d consumer_sema prepares to consume \n", iter, i);
		do_sleep(SHORTSLEEPTIME);
		down(&full);
		cprintf("Iter %d, No.%d consumer_sema gets an order\n", iter, i);
		down(&mutex);
		cprintf("Iter %d, No.%d consumer_sema consumes item %d\n", iter, i, s_tail);
		s_tail = (s_tail + 1) % (BUFFERSIZE + 1);
		up(&mutex);
		up(&empty);
		do_sleep(SHORTSLEEPTIME);
	}
    cprintf("No.%d consumer_sema quit\n", i);
    return 0;
}

int both_using_semaphore(void * arg)
{
	int i = (int)arg, iter = 0;
    cprintf("I am No.%d both_sema\n", i);
    while (iter++ < IB)
    {
    	if (empty.value >= 1) {
    		cprintf("Iter %d, No.%d both_sema prepares to produce \n", iter, i);
			do_sleep(SHORTSLEEPTIME);
			down(&empty);
			cprintf("Iter %d, No.%d both_sema gets a vacancy\n", iter, i);
			down(&mutex);
			cprintf("Iter %d, No.%d both_sema produces item %d\n", iter, i, s_head);
			s_head = (s_head + 1) % (BUFFERSIZE + 1);
			up(&mutex);
			up(&full);
			if (iter < IP / 2)
				do_sleep(SHORTSLEEPTIME);
			else
				do_sleep(LONGSLEEPTIME);
    	}
    	if (full.value >= 1) {
	    	cprintf("Iter %d, No.%d both_sema prepares to produce \n", iter, i);
			do_sleep(SHORTSLEEPTIME);
			down(&full);
			cprintf("Iter %d, No.%d both_sema gets an order\n", iter, i);
			down(&mutex);
			cprintf("Iter %d, No.%d both_sema consumes item %d\n", iter, i, s_tail);
			s_tail = (s_tail + 1) % (BUFFERSIZE + 1);
			up(&mutex);
			up(&empty);
			do_sleep(SHORTSLEEPTIME);
    	}
    	
	}
    cprintf("No.%d both_sema quit\n", i);
    return 0;
}

monitor_t mt, *mtp = &mt;
struct proc_struct *producer_proc_condvar[P];
struct proc_struct *consumer_proc_condvar[C];
struct proc_struct *both_proc_condvar[B];

int m_head = 0, m_tail = 0;

#define M_FULL ((m_head + 1) % (BUFFERSIZE + 1) == m_tail)
#define M_EMPTY (m_tail == m_head)
#define M_NC 2
#define M_CFULL 0
#define M_CEMPTY 1

int producer_using_condvar(void * arg) {
    int i = (int) arg, iter=0;
    cprintf("I am No.%d producer_condvar\n", i);
    while (iter ++ < IP)
    {
        cprintf("Iter %d, No.%d producer_condvar prepares to produce\n", iter, i);
        do_sleep(SHORTSLEEPTIME);
        down(&mtp->mutex);
        cprintf("Iter %d, No.%d producer_condvar inside monitor\n", iter, i);
        while (M_FULL) {
            cprintf("Full buffer. Iter %d, No.%d producer_condvar is waiting\n", iter, i);
            cond_wait(&mtp->cv[M_CFULL]);
        }
        cprintf("Iter %d, No.%d producer_condvar produces item %d\n", iter, i, m_head);
        m_head = (m_head + 1) % (BUFFERSIZE + 1); 
        cond_signal(&mtp->cv[M_CEMPTY]);
        if (mtp->next_count > 0)
            up(&mtp->next);
        else
            up(&mtp->mutex);
        do_sleep(LONGSLEEPTIME);
    }
    cprintf("No.%d producer_condvar quit\n", i);
    return 0;
}

int consumer_using_condvar(void * arg) {
    int i = (int) arg, iter=0;
    cprintf("I am No.%d consumer_condvar\n", i);
    while (iter ++ < IC)
    {
        cprintf("Iter %d, No.%d consumer_condvar prepares to consume\n", iter, i);
        do_sleep(SHORTSLEEPTIME);
        down(&mtp->mutex);
        cprintf("Iter %d, No.%d consumer_condvar inside monitor\n", iter, i);
        while (M_EMPTY) {
            cprintf("Empty buffer. Iter %d, No.%d consumer_condvar is waiting\n", iter, i);
            cond_wait(&mtp->cv[M_CEMPTY]);
        }
        cprintf("Iter %d, No.%d consumer_condvar consumes item %d\n", iter, i, m_tail);
        m_tail = (m_tail + 1) % (BUFFERSIZE + 1); 
        cond_signal(&mtp->cv[M_CFULL]);
        if (mtp->next_count > 0)
            up(&mtp->next);
        else
            up(&mtp->mutex);
        do_sleep(LONGSLEEPTIME);
    }
    cprintf("No.%d consumer_condvar quit\n", i);
    return 0;
}

int both_using_condvar(void * arg) {
    int i = (int) arg, iter=0;
    cprintf("I am No.%d both_condvar\n", i);
    while (iter ++ < IB)
    {
        cprintf("Iter %d, No.%d both_condvar prepares to produce\n", iter, i);
        do_sleep(SHORTSLEEPTIME);
        down(&mtp->mutex);
        cprintf("Iter %d, No.%d both_condvar inside monitor\n", iter, i);
        while (M_FULL) {
            cprintf("Full buffer. Iter %d, No.%d both_condvar is waiting\n", iter, i);
            cond_wait(&mtp->cv[M_CFULL]);
        }
        cprintf("Iter %d, No.%d both_condvar produces item %d\n", iter, i, m_head);
        m_head = (m_head + 1) % (BUFFERSIZE + 1); 
        cond_signal(&mtp->cv[M_CEMPTY]);
        if (mtp->next_count > 0)
            up(&mtp->next);
        else
            up(&mtp->mutex);
        do_sleep(LONGSLEEPTIME);

        cprintf("Iter %d, No.%d both_condvar prepares to consume\n", iter, i);
        do_sleep(SHORTSLEEPTIME);
        down(&mtp->mutex);
        cprintf("Iter %d, No.%d both_condvar inside monitor\n", iter, i);
        while (M_EMPTY) {
            cprintf("Empty buffer. Iter %d, No.%d both_condvar is waiting\n", iter, i);
            cond_wait(&mtp->cv[M_CEMPTY]);
        }
        cprintf("Iter %d, No.%d both_condvar consumes item %d\n", iter, i, m_tail);
        m_tail = (m_tail + 1) % (BUFFERSIZE + 1); 
        cond_signal(&mtp->cv[M_CFULL]);
        if (mtp->next_count > 0)
            up(&mtp->next);
        else
            up(&mtp->mutex);
        do_sleep(LONGSLEEPTIME);

        
    }
    cprintf("No.%d both_condvar quit\n", i);
    return 0;
}


void check_sync(void){
    int i = 0;

    //check semaphore
    /*sem_init(&mutex, 1);
	sem_init(&full, 0);
	sem_init(&empty, BUFFERSIZE);
	for (i = 0; i < P; ++ i) {
		int pid = kernel_thread(producer_using_semaphore, (void *)i, 0);
        if (pid <= 0) {
            panic("create No.%d producer_using_semaphore failed.\n");
        }
        producer_proc_sema[i] = find_proc(pid);
        set_proc_name(producer_proc_sema[i], "producer_sema_proc");
	}
	for (i = 0; i < C; i++) {
		int pid = kernel_thread(consumer_using_semaphore, (void *)i, 0);
        if (pid <= 0) {
            panic("create No.%d consumer_using_semaphore failed.\n");
        }
        consumer_proc_sema[i] = find_proc(pid);
        set_proc_name(consumer_proc_sema[i], "consumer_sema_proc");
    }
	for (i = 0; i < B; i++) {
		int pid = kernel_thread(both_using_semaphore, (void *)i, 0);
        if (pid <= 0) {
            panic("create No.%d both_using_semaphore failed.\n");
        }
        both_proc_sema[i] = find_proc(pid);
        set_proc_name(both_proc_sema[i], "both_sema_proc");
    }*/

    //check condition variable
    monitor_init(&mt, M_NC);
    for (i = 0; i < P; ++ i) {
        int pid = kernel_thread(producer_using_condvar, (void *)i, 0);
        if (pid <= 0) {
            panic("create No.%d producer_using_condvar failed.\n");
        }
        producer_proc_condvar[i] = find_proc(pid);
        set_proc_name(producer_proc_condvar[i], "producer_proc_condvar");
    }
    for (i = 0; i < C; ++ i) {
        int pid = kernel_thread(consumer_using_condvar, (void *)i, 0);
        if (pid <= 0) {
            panic("create No.%d consumer_using_condvar failed.\n");
        }
        consumer_proc_condvar[i] = find_proc(pid);
        set_proc_name(consumer_proc_condvar[i], "consumer_proc_condvar");
    }
    for (i = 0; i < B; ++ i) {
        int pid = kernel_thread(both_using_condvar, (void *)i, 0);
        if (pid <= 0) {
            panic("create No.%d both_using_condvar failed.\n");
        }
        both_proc_condvar[i] = find_proc(pid);
        set_proc_name(both_proc_condvar[i], "both_proc_condvar");
    }
}
```
