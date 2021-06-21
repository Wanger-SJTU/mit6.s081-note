1. 实现用户态下的线程切换。
    实验要求: 实现一个用户层面的线程系统,需要实现thread_create() thread_schedule()以及uthread_switch.S

    在uthread_switch.S中,仿照swtch.S进行上下文寄存器的保存和切换

    在uthread.c的thread_create()中,第一次创建进程时需要初始化ra和sp寄存器. ra寄存器需要存放传入的函数地址, sp寄存器传入当前线程的栈底(最开始的位置)

    thread_schedule()中,直接调用thread_switch,仿照swtch的格式进行上下文切换

2. linux/MacOS下实现多线程安全的map
    实验要求: 这个实验是基于实际的UNIXpthread库进行的,而非基于xv6. 需要对notxv6/ph.c进行补充,以实现多线程情况下能够正确地向一个哈希表插入键值对,其核心思想就是给每个线程对哈希表的操作都加锁.

3. linux/MacOs下线程同步
    实验要求: 实现线程的同步,即每一个线程必须等待其他所有线程都到达barrier之后才能继续进行下面的操作,需要用到pthread_cond_wait(&cond, &mutex)和pthread_cond_broadcast(&cond)来进行线程的sleep和唤醒其他所有&cond中睡眠的线程. 需要对barrier.c中的barrier()进行实现.

当每个线程调用了barrier()之后,需要增加bstate.nthread以表明到达当前round的线程数量增加了1, 但是由于bstate这个数据结构是线程之间共享的, 因此需要用pthread_mutex_lock对这个数据结构进行保护. 当bstate.nthread的数量达到线程总数nthread之后, 将bstate.round加1. 注意, 一定要等到所有的线程都达到了这个round, 将bstate.nthread清零之后才能将所有正在睡眠的线程唤醒, 否则如果先唤醒线程的话其他线程如果跑得很快, 在之前的线程将bstate.nthread清零之前就调用了bstate.nthread++,会出现问题(round之间是共用bstate.nthread的)