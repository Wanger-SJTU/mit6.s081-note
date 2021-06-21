
## part 1
实验要求：
在sbrk()时只增长进程的myproc()->sz而不实际分配内存。在kernel/trap.c中修改从而在产生page fault时分配物理内存给相应的虚拟地址。相应的虚拟地址可以通过r_stval()获得。r_scause()为13或15表明trap的原因是page fault

kernel/sysproc.c中修改sys_sbrk()
```cpp
uint64
sys_sbrk(void)
{
  int addr;
  int n;
  if(argint(0, &n) < 0)
    return -1;
  addr = myproc()->sz;
  myproc()->sz += n;
  // if(growproc(n) < 0)
  //   return -1;
  return addr;
}
```
kernel/trap.c中，在usertrap()函数中

```cpp
else if(r_scause() == 15 || r_scause() == 13){
    uint64 va = r_stval();
    char *mem;
    mem = kalloc();
    if (mem == 0) {
      p->killed = 1;
      exit(-1);
    }
    memset(mem, 0, PGSIZE);
    if (mappages(p->pagetable, PGROUNDDOWN(va), PGSIZE, (uint64)mem, PTE_W|PTE_X|PTE_R|PTE_U) != 0){
      kfree(mem);
      // panic("lazy alloc map failed");
      p->killed = 1;
    }

  } else if((which_dev = devintr()) != 0){
    // ok
  } 
```
s

修改kernel/vm.c中的uvmunmap，因为lazy allocation可能会造成myproc()-sz以下的内容没有被分配的情况，因此在unmap的过程中可能会出现panic，这是正常情况，需要continue

```cpp
if((pte = walk(pagetable, a, 0)) == 0)
    continue;
// panic("uvmunmap: walk");
if((*pte & PTE_V) == 0)
    continue;	    
// panic("uvmunmap: not mapped");
if(PTE_FLAGS(*pte) == PTE_V)
    panic("uvmunmap: not a leaf");
if(do_free){
```

# part2

实验要求： 在第一部分的基础上，要求处理

sbrk()的参数为负的情况，deallocate即可
```cpp
if (n < 0) {
    // deallocate the memory
    if ((myproc()->sz + n) < 0) {
        return -1;
    } else {
        if (uvmdealloc(myproc()->pagetable, addr, addr+n) != (addr+n)) {
            return -1;
        }
    }
}
```

fork()中将父进程的内存复制给子进程的过程中用到了uvmcopy，uvmcopy原本在发现缺失相应的PTE等情况下会panic，这里也要continue掉。在kernel/proc.c的uvmcopy中
```cpp
if((pte = walk(old, i, 0)) == 0)
    continue;
// panic("uvmcopy: pte should exist");
if((*pte & PTE_V) == 0)
    continue;
// panic("uvmcopy: page not present");

```

当造成的page fault在进程的user stack以下（栈底）或者在p->sz以上（堆顶）时，kill这个进程。在kernel/trap.c的usertrap中增加以下判断条件
```cpp
if (va > p->sz || va < PGROUNDDOWN(p->trapframe->sp)) {
    p->killed = 1;
    exit(-1);
} 
```
在exec中，loadseg调用了walkaddr，可能会找不到相应虚拟地址的PTE，此时需要分配物理地址。在kernel/vm.c中

```cpp
if((pte == 0) || ((*pte & PTE_V) == 0)) {
    if (va > myproc()->sz || va < PGROUNDDOWN(myproc()->trapframe->sp)) {
        myproc()->killed = 1;
        return 0;
    } 
    if ((pa = (uint64)kalloc()) == 0) 
      return 0;
    va = PGROUNDDOWN(va);
    if (mappages(myproc()->pagetable, va, PGSIZE, pa, PTE_W|PTE_X|PTE_R|PTE_U) != 0) {
        kfree((void*)pa);
        return 0;
    }
    return pa;
  }
```