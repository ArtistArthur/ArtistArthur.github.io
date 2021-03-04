---
title: mit6.828 lab4
top: false
cover: false
toc: true
mathjax: true
date: 2020-12-20 05:02:19
password:
summary:
tags:
categories:
---

# Introduce
这个实验将完成多处理器中的进程调度.  
在partA中将:
* 给JOS添加多处理器的支持
* 完成轮询调度
* 添加基本的进程管理系统调用(产生和销毁新进程,分配和映射内存)
在partB中:
* 实现一个和unix类似的`fork()`函数以允许用户进程产生一个自己的拷贝
在partC中:
* 实现进程间通讯以允许不同的用户进程进行交流和同步
* 实现时钟中断和优先权
<!-- more -->

# PartA: multiprocessor support and cooperative multitasking
* 让系统可以跑在多处理器环境中
* 实现一个新的系统调用以允许用户进程创建新进程
* 实现协作轮询调度以允许用户进程可以资源放弃cpu,让内核切换进程

## multiprocessor support
让JOS支持symmetric multiprocessor(SMP,对称多处理器,即地位相同).  


**excercise 1**
参照`boot_map_region()`就行,值得注意的是`pa`和`size`分别都要取整,`pa`想下取整,即从`pa`开始的那个页首开始映射,`size`向上取整,即每次映射`PGSIZE`整数倍的大小.   
而返回值根据调用它的函数
```
// lapicaddr is the physical address of the LAPIC's 4K MMIO
	// region.  Map it in to virtual memory so we can access it.
	lapic = mmio_map_region(lapicaddr, 4096);
```
可以看出,返回的是映射的首地址.  
```
//
// Reserve size bytes in the MMIO region and map [pa,pa+size) at this
// location.  Return the base of the reserved region.  size does *not*
// have to be multiple of PGSIZE.
//
void *
mmio_map_region(physaddr_t pa, size_t size)
{
	// Where to start the next region.  Initially, this is the
	// beginning of the MMIO region.  Because this is static, its
	// value will be preserved between calls to mmio_map_region
	// (just like nextfree in boot_alloc).
	static uintptr_t base = MMIOBASE;

	// Reserve size bytes of virtual memory starting at base and
	// map physical pages [pa,pa+size) to virtual addresses
	// [base,base+size).  Since this is device memory and not
	// regular DRAM, you'll have to tell the CPU that it isn't
	// safe to cache access to this memory.  Luckily, the page
	// tables provide bits for this purpose; simply create the
	// mapping with PTE_PCD|PTE_PWT (cache-disable and
	// write-through) in addition to PTE_W.  (If you're interested
	// in more details on this, see section 10.5 of IA32 volume
	// 3A.)
	//
	// Be sure to round size up to a multiple of PGSIZE and to
	// handle if this reservation would overflow MMIOLIM (it's
	// okay to simply panic if this happens).
	//
	// Hint: The staff solution uses boot_map_region.
	//
	// Your code here:
	size = ROUNDUP(size, PGSIZE);
	pa = ROUNDDOWN(pa, PGSIZE);
	if (base + size >= MMIOLIM)
	{
		panic("mmio_map_region overflow MMIOLIM!\n");
	}
	boot_map_region(kern_pgdir, base, size, pa, PTE_PCD | PTE_PWT | PTE_W);
	base += size;
	//panic("mmio_map_region not implemented");
	//返回此次映射的base
	return (void *)(base - size);
}
```
**exercise 2**   
在`page_init()`里面再加个条件就行.   
```
	if (i == 0 || i == (MPENTRY_PADDR / PGSIZE)) 
		{
			pages[i].pp_ref = 1;
			pages[i].pp_link = NULL;
		}
```

```
Question 

1.Compare kern/mpentry.S side by side with boot/boot.S. Bearing in mind that kern/mpentry.S is compiled and linked to run above KERNBASE just like everything else in the kernel, what is the purpose of macro MPBOOTPHYS? Why is it necessary in kern/mpentry.S but not in boot/boot.S? In other words, what could go wrong if it were omitted in kern/mpentry.S?
Hint: recall the differences between the link address and the load address that we have discussed in Lab 1.
```
答:因为对于`boot/boot.S`来说,它是运行在实模式下的,对它的寻址就是对实模式下物理地址中的寻址,因此它的寻址是对的.而对于`kern/mpentry.S`来说,这个文件的二进制代码也是和其他文件一起加载进物理内存的,并且`kern/mpentry.S`运行时,BSP开启了分页,且有了页表,此时对`kern/mpentry.S`文件里的对象寻址,得到的是`KERNBASE`上面的一个虚拟地址,而它实际上是被加载进0x7000的,如果按照原来的linker设置的地址寻址,会出问题.

## Locking
现在我们的代码会在`mp_main()`里初始化AP后自旋/无限循环(一个for循环).在进行下一步之前,我们需要强调一下当多处理器同时运行内核代码时的竞争条件(race conditions).实现这个的最简单方式是用一个大内核锁.大内核锁是一个单独的全局锁,当有进程进入内核态时就拥有这把锁,然后在返回用户态时释放锁.在这个模型中,在用户态的进程能在多cpu上并发,但是只有一个进程能运行在内核态;任何其他想要进入内核态的进程将被强制阻塞(等待). 
这个锁使得同时只有一个cpu处于内核态,而其他cpu想要进入内核态只能等待其他cpu退出内核态.
问题:为什么只能允许一个进程进入内核态?    
如果同时有多个cpu处于内核态,它们可能会修改相关的数据和数据结构,使得cpu之间出现混乱.  
`kern/spinlock.h`声明了一个大内核锁:`extern struct spinlock kernel_lock;`   
定义在`kern/spinlock.c`:
```
// The big kernel lock
struct spinlock kernel_lock = {
#ifdef DEBUG_SPINLOCK
	.name = "kernel_lock"
#endif
};
```
这个锁的结构:
```
// Mutual exclusion lock.
//互斥锁
struct spinlock {
    //如果锁被获取,locked=1,反之lock=0
	unsigned locked;       // Is the lock held?
    //为了debug时能发现是谁拥有锁
#ifdef DEBUG_SPINLOCK
	// For debugging:
	char *name;            // Name of lock.
	struct CpuInfo *cpu;   // The CPU holding the lock.
	uintptr_t pcs[10];     // The call stack (an array of program counters)
	                       // that locked the lock.
#endif
};
```
并且提供了`lock_kernel()`和`unlock_kernel()`函数来获取和释放锁.    
对于锁的实现需要用到一些原子指令,一个不用原子指令的获取锁的实现是:
Logically, xv6 should acquire a lock by executing code like
```
21 void
22 acquire(struct spinlock *lk)
23 {
24 for(;;) {
25 if(!lk->locked) {
26 lk->locked = 1;
27 break;
28 }
29 }
30 }
```
>Unfortunately, this implementation does not guarantee mutual exclusion on a multiprocessor. It could happen that two CPUs simultaneously reach line 25, see that lk->locked is zero, and then both grab the lock by executing line 26. At this point, two
different CPUs hold the lock, which violates the mutual exclusion property. Rather
than helping us avoid race conditions, this implementation of acquire has its own
race condition. The problem here is that lines 25 and 26 executed as separate actions.
In order for the routine above to be correct, lines 25 and 26 must execute in one
atomic (i.e., indivisible) step.    

为了把25,26行变为一行,可以使用`xchgl`汇编指令,它原子地交换两个寄存器或者内存寄存器的内容.  
`xchgl`的原理是:x86提供了一个指令前缀`lock`(0xf0),当检测到这个前缀时,就"锁定"内存总线,知道这条指令执行完成为止.因此在执行`xchgl`时,其他处理器不能访问这个内存单元.(参考深入理解linux内核-内核同步-原子操作)    
其他的解释:https://zhuanlan.zhihu.com/p/33445834     
JOS的实现中`xchg()`封装了`xchgl`指令:
```
void
spin_lock(struct spinlock *lk)
{
	// The xchg is atomic.
	// It also serializes, so that reads after acquire are not
	// reordered before it. 
	while (xchg(&lk->locked, 1) != 0)
		asm volatile ("pause");
}
```
`asm volatile ("pause");`的作用是提高性能,参见https://c9x.me/x86/html/file_module_x86_id_232.html   
https://kb.cnblogs.com/page/105657/    
http://web.cecs.pdx.edu/~alaa/courses/ece587/spring2012/notes/memory-ordering.pdf   
在这四个地方应该应用这个大内核锁:
>In i386_init(), acquire the lock before the BSP wakes up the other CPUs.
>
>In mp_main(), acquire the lock after initializing the AP, and then call sched_yield() to start running environments on this AP.
>
>In trap(), acquire the lock when trapped from user mode. To determine whether a trap happened in user mode or in kernel mode, check the low bits of the tf_cs.
>In env_run(), release the lock right before switching to user mode. Do not do that too early or too late, otherwise you will experience races or deadlocks.

> Exercise 5. Apply the big kernel lock as described above, by calling lock_kernel() and unlock_kernel() at the proper locations.   

在对应位置加入`lock_kernel();`或者`unlock_kernel();`就行.      


>Question 2
>It seems that using the big kernel lock guarantees that only one CPU can run the kernel code at a time. Why do we still need separate kernel stacks for each CPU? Describe a scenario in which using a shared kernel stack will go wrong, even with the protection of the big kernel lock.

因为在`_alltraps`到`lock_kernel()`的过程中(即引发中断进入内核态时和内核上锁之间),进程已经切换到了内核态,但并没有上内核锁,此时如果有其他CPU进入内核,如果用同一个内核栈,则`_alltraps`中保存的上下文信息会被破坏(因为切换中断时,指针会重置,被压栈的数据会被覆盖(或者两个cpu使用的段可能不同)),所以即使有大内核栈,CPU也不能用用同一个内核栈.同样的,解锁也是在内核态内解锁,在解锁到真正返回用户态这段过程中,也存在上述这种情况.   


## round-robin scheduling(轮询调度)
轮询调度算法的原理是每一次把来自用户的请求轮流分配给内部中的处理器,从1开始,直到N(内部处理个数),然后重新开始循环.
算法的优点是其简洁性,它无需记录当前所有连接的状态,所以它是一种无状态调度.   
JOS的round-robin:
* `kern/schec.c`里的`sched_yield()`函数负责挑选一个新的进程运行,因此它可以用来让出cpu.它循环线性搜索`envs[]`里的进程,对遇到的第一个`ENV_RUNNABLE`进程调用`env_run()`.
* `sched_yield()`绝对能在两个cpu上运行同一个进程.它能从进程的状态`ENV_RUNNABLE`分辨一个进程现在运行在某个cpu(可能是现在这个)
* 我们已经实现了一个系统调用`sys_yield()`,用户进程可以调用它来执行内核的`sched_yield()`,然后自动放弃cpu以运行不同的进程.
**exercise 6**  
>Implement round-robin scheduling in sched_yield() as described above. Don't forget to modify syscall() to dispatch sys_yield().  

在`kern/sched.c`下修改：  
```c
// Choose a user environment to run and run it.
void
sched_yield(void)
{
	struct Env *idle;

	// Implement simple round-robin scheduling.
	//
	// Search through 'envs' for an ENV_RUNNABLE environment in
	// circular fashion starting just after the env this CPU was
	// last running.  Switch to the first such environment found.
	//
	// If no envs are runnable, but the environment previously
	// running on this CPU is still ENV_RUNNING, it's okay to
	// choose that environment.
	//
	// Never choose an environment that's currently running on
	// another CPU (env_status == ENV_RUNNING). If there are
	// no runnable environments, simply drop through to the code
	// below to halt the cpu.

	// LAB 4: Your code here.
	int index = 0;
	if (curenv!=NULL)
	{
		index = ENVX(curenv->env_id);
	}
	//从0开始,因为curenv->env_status==ENV_RUNNING
	for (int i = 0; i < NENV; i++)
	{
		index += i;
		index %= NENV;
		if(envs[index].env_status==ENV_RUNNABLE)
		{
			env_run(&envs[index]);
		}
	}
	if(curenv&&curenv->env_status==ENV_RUNNING)
	{
		env_run(curenv);
	}
		// sched_halt never returns
		sched_halt();
}
```
再在`kern/syscall.c`下dispatch`sched_yield()`：
```c
case SYS_yield:
		sys_yield();
		ret= 0;
		break;
```
>Question  
>3. In your implementation of env_run() you should have called lcr3(). Before and after the call to lcr3(), your code makes references (at least it should) to the variable e, the argument to env_run. Upon loading the %cr3 register, the addressing context used by the MMU is instantly changed. But a virtual address (namely e) has meaning relative to a given address context--the address context specifies the physical address to which the virtual address maps. Why can the pointer e be dereferenced both before and after the addressing switch?  

在链接时，链接器把变量的虚拟地址链接到`KERNBASE`以上，在`bootstrap`时硬编码了一个页表，把初始4MB映射到`KERNBASE`开始的4MB，因此在`lcr3()`之前和之后，这个映射都是一样的。  
>4.Whenever the kernel switches from one environment to another, it must ensure the old environment's registers are saved so they can be restored properly later. Why? Where does this happen?  
这样才能恢复上下文，当任务切换回来时能正确运行。  
### System calls for environment creation  
尽管内核现在可以在多个用户级别的环境中运行和切换，但仍限于内核最初设置的运行环境。现在你要实现必要的JOS系统调用，以允许用户环境创建和启动其他新的用户环境。   
Unix提供fork()系统调用作为其过程创建原语。Unixfork()复制调用进程（父进程）的整个地址空间，以创建一个新进程（子进程）。从用户空间可观察到的两者之间的唯一区别是它们的进程ID和父进程ID（由getpid和返回getppid）。在父级中， fork()返回子级的进程ID，而在子级中，fork()返回0。默认情况下，每个进程都获得自己的专用地址空间，并且另一个进程对内存的修改对其他人都不可见。  
你将提供一组不同的，更原始的JOS系统调用，以创建新的用户模式环境。通过这些系统调用fork()，除了创建环境的其他样式之外，你还可以完全在用户空间中实现类似Unix的功能。你将为JOS编写的新系统调用如下：  
`sys_exofork`：  
该系统调用创建了一个几乎空白的新环境：在其地址空间的用户部分中未映射任何内容，并且该环境不可运行。`sys_exofork`调用时，新环境将具有与父环境相同的寄存器状态。在父进程中，`sys_exofork` 将返回`envid_t`新创建的环境（如果环境分配失败，则返回负错误代码）。但是，在子进程中，它将返回0。（由于子进程开始时标记为不可运行，sys_exofork因此，直到父代通过使用....标记子代可显式允许该子代之前，它 才真正返回子代。）
`sys_env_set_status`：  
将指定环境的状态设置为ENV_RUNNABLE或ENV_NOT_RUNNABLE。一旦其地址空间和寄存器状态已完全初始化，此系统调用通常用于标记准备运行的新环境。
`sys_page_alloc`：  
分配一页物理内存，并将其映射到给定环境的地址空间中的给定虚拟地址。
`sys_page_map`：  
将一个页面映射（而不是页面的内容！）从一个环境的地址空间复制到另一个环境，保留内存共享安排，以便新映射和旧映射都引用同一物理内存页面。
`sys_page_unmap`：  
取消映射在给定环境中映射到给定虚拟地址的页面。
对于上面所有接受环境ID的系统调用，JOS内核都支持以下约定：值0表示“当前环境”。本公约对实现envid2env() 在克恩/ env.c。

我们fork() 在测试程序user / dumbfork.c中提供了类Unix的非常原始的实现。该测试程序使用上述系统调用来创建和运行带有其自身地址空间副本的子环境。然后，使用sys_yield 与上一个练习相同的方法来回切换两个环境。父级在10次迭代后退出，而子级在20次迭代后退出。






问题:
# Call mp_main().  (Exercise for the reader: why the indirect call?)
	#无页表?
	movl    $mp_main, %eax
	call    *%eax
分页开启前是怎么寻址的?

```
//复习实模式寻址:
# Each non-boot CPU ("AP") is started up in response to a STARTUP
# IPI from the boot CPU.  Section B.4.2 of the Multi-Processor
# Specification says that the AP will start in real mode with CS:IP
# set to XY00:0000, where XY is an 8-bit value sent with the
# STARTUP. Thus this code must start at a 4096-byte boundary.
#
//为什么?
# Because this code sets DS to zero, it must run from an address in
# the low 2^16 bytes of physical memory.
#
# boot_aps() (in init.c) copies this code to MPENTRY_PADDR (which
# satisfies the above restrictions).  Then, for each AP, it stores the
# address of the pre-allocated per-core stack in mpentry_kstack, sends
# the STARTUP IPI, and waits for this code to acknowledge that it has
# started (which happens in mp_main in init.c).
#
# This code is similar to boot/boot.S except that
#    - it does not need to enable A20
//为什么?
#    - it uses MPBOOTPHYS to calculate absolute addresses of its
#      symbols, rather than relying on the linker to fill them

```