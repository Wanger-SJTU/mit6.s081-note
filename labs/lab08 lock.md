
1. Memory allocator
    实验要求：实现一个per CPU freelist，以减小各个进程同时调用kalloc、kfree造成的对kmem.lock锁的竞争。可以调用cpuid()函数来获取当前进程运行的CPU ID，但是要在调用前加上push_off以关闭中断。
    在kernel/kalloc.c中，修改kmem结构体为数组形式
    kfree将释放出来的freelist节点返回给调用kfree的CPU
    在kalloc中，当发现freelist已经用完后，需要向其他CPU的freelist借用节点

2. Buffer cache
    实验要求：xv6文件系统的buffer cache采用了一个全局的锁bcache.lock来负责对buffer cache进行读写保护，当xv6执行读写文件强度较大的任务时会产生较大的锁竞争压力，因此需要一个哈希表，将buf entry以buf.blockno为键哈希映射到这个哈希表的不同的BUCKET中，给每个BUCKET一个锁，NBUCKET最好选择素数，这里选择13。

    注意：这个实验不能像上一个一样给每个CPU一个bcache，因为文件系统在多个CPU之间是真正实现共享的，否则将会造成一个CPU只能访问某些文件的问题。

    