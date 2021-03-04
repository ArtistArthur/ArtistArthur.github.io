---
title: mit6.828-lab3
top: false
cover: false
toc: true
mathjax: true
date: 2020-12-02 23:40:02
password:
summary:
tags: [OS]
categories:
---
## mit6.828-lab3:user environments
在这个lab里你将:
* 完成基本的用户进程相关设施和数据结构(envs struct等). 
* 加载一个程序镜像到内存并运行它.
* 完成中断/异常,系统调用的相关设施,让kernel有能力处理中断/异常和系统调用.  
<!-- more -->
### partA:user environments and exception handling
首先是用户相关的数据结构`Env`:
```c
struct Env {
	struct Trapframe env_tf;	// Saved registers
	struct Env *env_link;		// Next free Env
	envid_t env_id;			// Unique environment identifier
	envid_t env_parent_id;		// env_id of this env's parent
	enum EnvType env_type;		// Indicates special system environments
	unsigned env_status;		// Status of the environment
	uint32_t env_runs;		// Number of times environment has run
	// Address space
	pde_t *env_pgdir;		// Kernel virtual address of page dir
};   
```
<!--more-->      
进程上下文切换的相关结构:   
```c
struct PushRegs {
	/* registers as pushed by pusha */
	uint32_t reg_edi;
	uint32_t reg_esi;
	uint32_t reg_ebp;
	uint32_t reg_oesp;		/* Useless */
	uint32_t reg_ebx;
	uint32_t reg_edx;
	uint32_t reg_ecx;
	uint32_t reg_eax;
} __attribute__((packed));
struct Trapframe {
	struct PushRegs tf_regs;
	uint16_t tf_es;
	uint16_t tf_padding1;
	uint16_t tf_ds;
	uint16_t tf_padding2;
	uint32_t tf_trapno;
	/* below here defined by x86 hardware */
	uint32_t tf_err;
	uintptr_t tf_eip;
	uint16_t tf_cs;
	uint16_t tf_padding3;
	uint32_t tf_eflags;
	/* below here only when crossing rings, such as from user to kernel */
	uintptr_t tf_esp;
	uint16_t tf_ss;
	uint16_t tf_padding4;
} __attribute__((packed));

```
```c
void
env_pop_tf(struct Trapframe *tf)
{
	asm volatile(
		"\tmovl %0,%%esp\n"
		"\tpopal\n"
		"\tpopl %%es\n"
		"\tpopl %%ds\n"
		"\taddl $0x8,%%esp\n" /* skip tf_trapno and tf_errcode */
		"\tiret\n"
		: : "g" (tf) : "memory");
	panic("iret failed");  /* mostly to placate the compiler */
}

```
`%0`表示`tf`代表的寄存器,`movl %0,%%esp`把`tf`的地址存入`esp`寄存器中,`popal`是`popa`的长指令(pop all).  
`pusha`作用是:把八个通用寄存器全部`push`入`esp`中,顺序为`eax ecx edx ebx oldesp ebp esi edi`而`popa`的作用相反:从`esp`中把这八个值弹出到相应的寄存器,但是并不弹出`old_esp`,会跳过它,方式是`esp`+4.   
IRET是一个汇编指令,这个指令会做很多事情:   
the IRET instruction pops the return instruction pointer, return code segment selector, and EFLAGS image from the stack to the EIP, CS, and EFLAGS registers, respectively, and then resumes execution of the interrupted program or procedure. If the return is to another privilege level, the IRET instruction also pops the stack pointer and SS from the stack, before resuming program execution.  

```

Operation
IF OperandSize = 32 (* instruction = POPAD *)
THEN
EDI ← Pop();
ESI ← Pop();
EBP ← Pop();
increment ESP by 4 (* skip next 4 bytes of stack *)
EBX ← Pop();
EDX ← Pop();
ECX ← Pop();
EAX ← Pop();
ELSE (* OperandSize = 16, instruction = POPA *)
DI ← Pop();
SI ← Pop();
BP ← Pop();
increment ESP by 2 (* skip next 2 bytes of stack *)
BX ← Pop();
DX ← Pop();
CX ← Pop();
AX ← Pop();
参考资料:https://www.cs.cmu.edu/~410/doc/intel-isr.pdf
```
参考:https://www.cnblogs.com/whutzhou/articles/2638498.html  
`env_pop_tf()`的作用是:转到`tf`代表的用户程序,先把`esp`设为这个`Trapframe`的地址,然后`popa`,即从`tf`中的`struct PushRegs tf_regs`弹出值到相应的寄存器,实现上下文的切换.然后再从`tf`中弹出`es`和`ds`的值,再跳过`tf_trapno and tf_errcode`,最后`iret`.    
值得注意的是:栈是从上往下增长的,`tf`陷阱门是从下网上长的,因此把`esp`设为`tf`,`pop`弹出值的时候,`esp`回退(即往上),对应着`tf`往上依次读取成员.

#### exercise2
完成几个函数:
* `env_init()`:初始化所有的在`envs`数组中的`Enc`结构体,并把他们添加到`env_free_list`指针后面,之后调用`env_init_per_cpu`以设置段管理的相关硬件,它们分别是特权级0(kernel)和特权级3(user)的段.
* `env_setup_vm()`:为新的`environment`分配一个页目录表,并且初始化新的地址空间中的内核部分(通过复制).
* `region_alloc()`:为新的`environment`分配并映射物理地址.
* `load_icode()`:需要自己实现解析ELF二进制文件的功能(就和`bootloader`里面做的一样),并把二进制文件的内容加载到新的`environment`的用户地址空间中.
* `env_create`:通过`env_alloc()`分配一个`environment`,并且调用`load_icode()`把ELF二进制文件加载进去.
* `env_run`:在用户模式下启动一个给定的`environment`.
代码解析:  
* `env_init()`:  

```c

// Mark all environments in 'envs' as free, set their env_ids to 0,
// and insert them into the env_free_list.
// Make sure the environments are in the free list in the same order
// they are in the envs array (i.e., so that the first call to
// env_alloc() returns envs[0]).
//
void
env_init(void)
{
	// Set up envs array
	// LAB 3: Your code here.
	size_t i = 0;
	env_free_list = envs;
    //让env_free_list等于数组第一个元素,再让整个数组通过链表连起来
    //env_link指向下一个节点,同时把id设置为0
    //让最后一个节点指向null
	for (i = 0; i < NENV-1; i++)
	{
		envs[i].env_link = envs + i + 1;
		envs[i].id = 0;

	}
	(envs + NENV - 1)->env_link = NULL;
	// Per-CPU part of the initialization
	env_init_percpu();
}

```

* env_setup_vm():

```c

// Initialize the kernel virtual memory layout for environment e.
// Allocate a page directory, set e->env_pgdir accordingly,
// and initialize the kernel portion of the new environment's address space.
// Do NOT (yet) map anything into the user portion
// of the environment's virtual address space.
//
// Returns 0 on success, < 0 on error.  Errors include:
//	-E_NO_MEM if page directory or table could not be allocated.
//
static int
env_setup_vm(struct Env *e)
{
	int i;
	struct PageInfo *p = NULL;

	// Allocate a page for the page directory
	if (!(p = page_alloc(ALLOC_ZERO)))
		return -E_NO_MEM;

	// Now, set e->env_pgdir and initialize the page directory.
	//
	// Hint:
	//    - The VA space of all envs is identical above UTOP
	//	(except at UVPT, which we've set below).
	//	See inc/memlayout.h for permissions and layout.
	//	Can you use kern_pgdir as a template?  Hint: Yes.
	//	(Make sure you got the permissions right in Lab 2.)
	//    - The initial VA below UTOP is empty.
	//    - You do not need to make any more calls to page_alloc.
	//    - Note: In general, pp_ref is not maintained for
	//	physical pages mapped only above UTOP, but env_pgdir
	//	is an exception -- you need to increment env_pgdir's
	//	pp_ref for env_free to work correctly.
	//    - The functions in kern/pmap.h are handy.

	// LAB 3: Your code here.

	e->env_pgdir =(pde_t*) page2kva(p);
	memcpy(e->env_pgdir, kern_pgdir, PGSIZE);
	p->pp_ref++;
	// UVPT maps the env's own page table read-only.
	// Permissions: kernel R, user R
	e->env_pgdir[PDX(UVPT)] = PADDR(e->env_pgdir) | PTE_P | PTE_U;

	return 0;
}


```

* `region_alloc`:
为进程分配`len`个字节的物理内存并且映射到`va`所代表的虚拟地址上,不要设置0或者初始化被映射的页(page_alloc()传0进去就可以).   
页面的权限应该是用户和内核都可写的.  
如果分配失败则`panic`  

```c

//
// Allocate len bytes of physical memory for environment env,
// and map it at virtual address va in the environment's address space.
// Does not zero or otherwise initialize the mapped pages in any way.
// Pages should be writable by user and kernel.
// Panic if any allocation attempt fails.
//
static void
region_alloc(struct Env *e, void *va, size_t len)
{
	// LAB 3: Your code here.
	// (But only if you need it for load_icode.)
	//
	// Hint: It is easier to use region_alloc if the caller can pass
	//   'va' and 'len' values that are not page-aligned.
	//   You should round va down, and round (va + len) up.
	//   (Watch out for corner-cases!)
	void *low = ROUNDDOWN(va, PGSIZE);//向下取整,即把va所在的那一页全部映射,从va前面页面对齐开始
	void *up = ROUNDUO(va + len, PGSIZE);//向上取整
	for (; low < up;low+=PGSIZE)
	{
		struct PageInfo *p = page_alloc(0);
		if(!p)
		{
			panic("region_alloc error: region_alloc failed!\n");
		}
		p->pp_ref++;
		page_insert(e->env_pgdir, p, low, PTE_W|PTE_U);
	}
}

```

* `load_icode`:
为用户进程设置初始的二进制程序,栈,处理器标志等.  
这个函数只在内核初始化的时候被调用,在运行第一个用户态进程之前.  
这个函数从二进制镜像加载所有可加载的段到进程的内存中,从适当的、二进制文件头里描述的虚拟地址开始.   
同时它把在二进制头里要求的一些段的内容设置为0,比如`bss segment`.  
这些和我们的`boot loader`所做的很像,但是它是从硬盘读取代码.去参考下`boot/main.c`里的代码.  
最后,这个函数从这个进程的初始栈映射一个页.  
如果它遇到任何问题,则会`panic`,请思考它在什么情况下会出错.

#### the task state segment
处理器需要一个地方储存中断或者异常发生之前的处理器状态,比如`EIP`和`CS`的值,以便当异常或者中断执行完毕后可以恢复cpu的状态,并从原来的程序位置重新执行.  
但是这个储存的地方,也必须免受权限不够的用户态程序影响,否则一些恶意程序会影响kernel.  
因此x86处理器当特权级切换的时候也会切换栈,`TSS`任务状态段的目的便是服务这一过程,它指定了段选择符并指出这个栈在段中的位置,处理器会`push` `SS,ESP,EFLAGS,CS,EIP`和error code到这个栈里(error code只有某些中断或者异常会有,有的没有).随后处理器从中断描述符中加载`CS`和`EIP`以切换到新栈.  
在JOS的实现中,当处理器从中断描述符中加载`CS`和`EIP`后,接下来的指令开始压入error code(如果cpu之前没有压的话),trap编号,然后`call _alltraps`,压入`ds es`然后`pushal`等等,目的是在kernel的栈上构造一个`Trapframe`,它保存了中断前进程的状态,在`trap()`中会复制这个结构体到`curenv->env_tf`中,以便之后恢复现场.
以上的意思是:当转移控制权时,如果特权级发生变化,那么先要把当前处理器状态存入一个安全的`TSS`指定的栈中,再切换到目的栈,这个过程涉及到三个栈:当前栈,`TSS`指定的栈和将要转移到的代码段的栈.    

```
http://blog.chinaunix.net/uid-685034-id-2076045.html  
堆栈切换和任务切换
堆栈切换
中断发生时,从用户堆栈切换到内核堆栈是硬件完成的是吗？需要软件上哪些支持呢？
x86处理器是由硬件完成的.
但很多RISC(reduced instruction set computer,精简指令集计算机,例如：MIPS R3000、HP—PA8000系列,Motorola M88000等均属于RISC微处理器)处理器必须由软件来实现用户态与核态之间的堆栈切换
X86是CISC处理器(复杂指令集计算机(Complex Instruction Set Computer,CISC))
X86处理器的SP切换过程是这样的：

当中断或异常发生时,处理器会检查是否有CPU运行级别的改变,如果有的话,则进行堆栈切换.切换的过程如下：

1. 读取TR寄存器以便访问当前进程的TSS段,因为TSS段中保存着当前进程在核心态下的堆栈指针.
2. 从TSS段中加载相应的堆栈地址到SS和ESP寄存器中.
3. 在核态堆栈中,保存用户态下的SS寄存器和ESP寄存器值.

由于linux内核仅使用了一个TSS段,因此当发生进程切换时,内核必须将新进程的核态堆栈更新到TSS段中.

而对于RISC处理器而言, 本着“简洁”的设计原则,当发生中断或异常时,CPU仅仅只是跳转到某个TRAP向量地址去执行(当然,硬件还是会自动设置处理器状态寄存器PSR中的核心态标志位,同时保存trap发生前的处理器运行级别),而其余的工作就都统统留给软件去完成了.

因此,RISC处理器的trap handler通常都做这样的一些工作：

1.根据trap发生前的处理器运行级别判断是否需要进行堆栈切换. 如果trap发生之前就是处在核心态下,那显然就不要切换堆栈.而是直接去做SAVE_ALL好了.

2. 如果之前是用户态,那么从内核的某个固定的地址加载当前进程的核态SP指针.然后进行SAVE_ALL保存中断现场.
#####
我们知道每个进程都有一个用户堆栈与系统堆栈,那么此外是否还有一个操作系统内核专用的堆栈呢？
BTW：内核代码都是运行在当前进程的核心态堆栈中的,并不需要专门的堆栈
########
那么当系统初始化时,系统中第一个进程还没有生成的时候,用的是哪个堆栈呢？
从head.s程序起,系统开始正式在保护模式下运行.此时堆栈段被设置为内核数据段(0x10),堆栈指针esp设置成指向user_stack数组的顶端,保留了1页内存(4K)最为堆栈使用.此时该堆栈是内核程序自己使用的堆栈.
(有疑问的答案：系统初始化用的是0进程的堆栈 －－ 也就是是init_task的堆栈.
整个start_kernel()都是在init_task的堆栈中执行的.start_kernel()最后clone出1进程－－也就是init进程.然后0进程就去执行cpu_idle()函数了－－也就是变成idle进程了.然后发生一次进程调度(进程切换时会切换内核堆栈),init进程得到运行,此时内核就在init进程的堆栈中运行.)
#####
任务0的堆栈
    任务0的堆栈比较特殊,在执行了move_to_user_mode()之后,它的内核堆栈位于其任务数据结构所在页面的末端,而它的用户态堆栈就是前面进入保护模式后所使用的堆栈,即user_stack数组的位置.任务0的内核态堆栈是在其人工设置的初始化任务数据结构中指定的,而它的用户态堆栈是在执行move_to_user_mode()时,在模拟iret返回之前的堆栈中设置的.在该堆栈中,esp仍然是user_stack中原来的位置,而ss被设置成0x17,也即用户局部表中的数据段,也即从内存地址0开始并且限长640KB的段.
 
任务切换
I386硬件任务切换机制
 
1.I386硬件任务切换机制
　　 Intel 在i386体系的设计中考虑到了进程的管理和调度,并从硬件上支持任务间的切换.为此目的,Intel在i386系统结构中增设了一种新的段“任务状态段”TSS.一个TSS虽然说像代码段,数据段等一样,也是一个段,实际上却是一个104字节的数据结构,用以记录一个任务的关键性的状态信息.
　　 像其他段一样,TSS也要在段描述表中有个表项.不过TSS只能在GDT中,而不能放在任何一个LDT中或IDT中.若通过一个段选择项访问一个TSS,而选择项中的TI位为1,就会产生一次GP异常.
　　 另外,CPU中还增设一个任务寄存器TR,指向当前任务的TSS.相应地,还增加了一条指令LTR,对TR寄存器进行装入操作.像CS和DS一样,TR也有一个程序不可见部分,每当将一个段选择码装入到TR中时,CPU就会自动找到所选择的TSS描述项并将其装入到TR的程序不可见部分,以加速以后对该TSS段的访问.
　　 还有,在IDT表中,除了中断门、陷阱门和调用门以为,还定义了一种任务门.任务门中包含一个TSS段选择码.当CPU因中断而穿过一个任务门时,就会将任务门中的选择码自动装入TR,使TR指向新的TSS,并完成任务的切换.CPU还可以通过JMP和CALL指令实现任务切换,当跳转或调用的目标段实际上指向GDT表中的一个TSS描述项时,就会引起一次任务切换.


```

```
内核栈的实现
以linux内核为例,内核在创建进程并时,首先需要给进程分配task_struct结构体,在做这一步的时候内核实际上分配了两块连续的物理空间(一般是1个物理页),上边供堆栈使用,下边保存进程描述符task_struct.这个整体叫做进程的内核栈,因此task_struct是在进程内核栈内部的.
当为内核栈分配地址空间的时候,分配一个页面(这里以8k为例)返回的地址是该该页面的低地址,而栈是由高地址向低地址增长的,栈顶指针只需将该内核栈的首地址+8k即可

```

#### nested exceptions and interrupt
在用户态和内核态处理器都可以接受异常和中断,但是x86只有从用户态切换到内核态的时候才会在储存cpu当前状态时自动切换栈(其他时候不会),然后再从中断向量表中invoke适当的中断或异常.  
如果处理器以及处于内核态了(即`CS`的低2位为0),那么处理器只是把当前处理器状态再次储存到当前栈中而不切换栈.下面在`system call`中我们可以看到这个特性的好处.
如果处理器已经在内核态了,并出发了嵌套异常,由于它不用切换栈,因此`SS`和`ESP`寄存器的状态就没有必要储存了,因此只需储存 `EFLAGS,CS,EIP`和error code,如果内核栈已经满了,这个时候再次出现中断或者异常, `EFLAGS,CS,EIP`压不进去了,就会出现不能恢复原来现场的功能,这是一个bug,处理器面对这样的情况时,粗暴地重置自己,设计kernel的时候应该避免这种情况.  

There are two sources for external interrupts and two sources for
exceptions:
 1. Interrupts
 * Maskable interrupts, which are signalled via the INTR pin.
 * Nonmaskable interrupts, which are signalled via the NMI
 (Non-Maskable Interrupt) pin.
 2. Exceptions
 * Processor detected. These are further classified as faults, traps,
 and aborts.
 * Programmed. The instructions INTO, INT 3, INT n, and BOUND can
 trigger exceptions. These instructions are often called "software
 interrupts", but the processor handles them as exceptions.   
INTR是一个外部中断请求触发器,这个触发器可以传递中断,INTR为“1”时(即有设备请求中断),表示该设备向CPU提出中断请求.但是设备如果要提出中断请求,其设备本身必须准备就绪,即接口内的完成触发器D的状态必须为“1”.MASK为中断屏蔽触发器,如果是“1”,中断会被屏蔽掉,封锁中断源的请求.仅当设备准备就绪(D=1),且该设备未被屏蔽(MASK=0)时,CPU的中断查询信号可将中断请求触发器置“1”.
NMI是2号中断,它由cpu直接使用,操作系统不能使用,它用来处理一些严重的突发情况比如电源掉电、存储器读写出错、总线奇偶位出错等.NMI线上中断请求是不可屏蔽的(即无法禁止的)、而且立即被CPU锁存.因此NMI是边沿触发,不需要电平触发.NMI的优先级也比INTR高.不可屏蔽中断的类型指定为2,在CPU响应NMI时,不必由中断源提供中断类型码,因此NMI响应也不需要执行总线周期INTA.   


#### setting up the idt
下面我们将设置idt去处理0-31的异常和中断,system call和32-47的中断和异常会在以后的lab实现.  
`inc/trap.h`和`kern/trap.h`中有中断和异常的相关定义,`kern/trap.h`中定义的只用在kernel里,`inc/trap.h`中定义的在用户程序中也会起作用.  
note:0-31中有intel保留的项,这些项随便自己怎么处理.  
每个异常或者中断都应该在`trapentry.S`中有它自己的handler,并且`trap_init()`应该用这些handler的地址初始化IDT.每个handler应该在栈上建立一个`struct Trapframe`并且通过一个指向Trapframe的地址call`trap()`.然后`trap()`会处理异常/中断,或者交给另一个处理函数处理.  
#### exercise 4
任务:编辑`trapentr.S`和`trap.c`并且实现上述描述的功能,`TRAPHANDLER`和`TRAPHANDLER_NOEC`宏可以帮到你,`T_*`也可以.你需要在`trapentry.S`为每一个`trap`添加entry point,你需要提供`TRAPHANDLER`指向的`_alltrap`函数,你也要修改`trap_init()`去初始化idt使得它指向每个entry point,`SETGAE`宏会帮到你.  
* `_alltrap`应该:
  * push values 以让栈看起来像一个`struct Trapframe`
  * 把`GD_KD`(kernel data段)加载进`%ds`和`%es`中 (怎么加载?我一开始用的`movl $GD_KD %ds`,但是报错了,原因是段寄存器不能通过立即数赋值,只能通过通用寄存器或存储器赋值,因此使用`pushl $GD_KD popl %ds)
  * `pushl %esp`把地址传给`Trapframe`将作为一个`trap()`的一个参数
  * call `trap()` trap可以返回吗?//不可以


```c
/* See COPYRIGHT for copyright information. */

#include <inc/mmu.h>
#include <inc/memlayout.h>
#include <inc/trap.h>



###################################################################
# exceptions/interrupts
###################################################################

/* TRAPHANDLER defines a globally-visible function for handling a trap.  TRAPHANDLER为处理中断/异常定义了一个全局可见的函数
 * It pushes a trap number onto the stack, then jumps to _alltraps.		这个函数把trap number压栈然后跳转到 _alltraps函数
 * Use TRAPHANDLER for traps where the CPU automatically pushes an error code.	TRAPHANDLER用在处理器自动压栈错误码的中断/异常中(即有错误码的中断/异常,没有错误码的用下面的TRAPHANDLER_NOEC函数处理)
 *
 * You shouldn't call a TRAPHANDLER function from C, but you may	不能够在c语言程序中调用TRAPHANDLER
 * need to _declare_ one in C (for instance, to get a function pointer	但是需要在c语言程序中声明TRAPHANDLER定义的函数
 * during IDT setup).  You can declare the function with	以便于在建立idt的时候获得相应函数指针
 *   void NAME();											声明方式是: void name();
 * where NAME is the argument passed to TRAPHANDLER.
 */

#define TRAPHANDLER(name, num)						\
	.globl name;		/* define global symbol for 'name' */	\
	.type name, @function;	/* symbol type is function */		\
	.align 2;		/* align function definition */		\
	name:			/* function starts here */		\
	pushl $(num);							\
	jmp _alltraps

/* Use TRAPHANDLER_NOEC for traps where the CPU doesn't push an error code.
 * It pushes a 0 in place of the error code, so the trap frame has the same
 * format in either case.
 */
#define TRAPHANDLER_NOEC(name, num)					\
	.globl name;							\
	.type name, @function;						\
	.align 2;							\
	name:								\
	pushl $0;							\
	pushl $(num);							\
	jmp _alltraps

.text

/*
 * Lab 3: Your code here for generating entry points for the different traps.
 */
	TRAPHANDLER_NOEC(traphd0,0)
	TRAPHANDLER_NOEC(traphd1,1)
	TRAPHANDLER_NOEC(traphd2,2)
	TRAPHANDLER_NOEC(traphd3,3)
	TRAPHANDLER_NOEC(traphd4,4)
	TRAPHANDLER_NOEC(traphd5,5)
	TRAPHANDLER_NOEC(traphd6,6)
	TRAPHANDLER_NOEC(traphd7,7)
	TRAPHANDLER(traphd8,8)
	TRAPHANDLER_NOEC(traphd9,9)
	TRAPHANDLER(traphd10,10)
	TRAPHANDLER(traphd11,11)
	TRAPHANDLER(traphd12,12)
	TRAPHANDLER(traphd13,13)
	TRAPHANDLER(traphd14,14)
	TRAPHANDLER_NOEC(traphd16,16)

/*
 * Lab 3: Your code here for _alltraps
 */
//根据Trapframe结构可以看出,处理器已经把SS,ESP,EFLAGS,CS,EIP压栈了
//上面的函数又把error code和trap number压栈了
//剩下只有ds es还有pusha对应的没有压了
//再根据实验提示,把GD_KD加载进ds和es最后call trap
_alltraps:
	pushl %ds
	pushl %es
	pushal 
	movl $GD_KD %ds
	movl $GD_KD %es
	pushl %esp
	call trap

```
##### the breakpoint exception
断点异常被用来允许debugger在一个程序中插入断点,方式是临时把程序中的某个位置的代码用`int3`代替,`int3`是一个一字节的汇编代码,因此可以放入几乎所有地方.因此当程序执行到这个地方的时候,就会发生中断,进而运行相应的中断程序,这个时候往往可以看到程序的上下文内容,以此来调试代码.  
JOS把3号中断向量的处理程序设为`monitor()`,因此发生断点异常的时候,将会进入`monitor()`程序.
Questions  
4. The break point test case will either generate a break point exception or a general protection fault depending on how you initialized the break point entry in the IDT (i.e., your call to SETGATE from trap_init). Why? How do you need to set it up in order to get the breakpoint exception to work as specified above and what incorrect setup would cause it to trigger a general protection fault?   
break point 的dpl应该设为3,因为用户态程序也会用到.否则会因为权限不够而产生一般保护性异常.
5. What do you think is the point of these mechanisms, particularly in light of what the user/softint test program does?   




##### system call
c
对于用户程序来说,执行`cprintf()`等函数需要调用系统调用.  
比如一个用户程序执行`cprintf()`,这是`lib/printf.c`下的函数,是一个普通函数,但是这个函数需要IO输出,就涉及到kernel的资源调度,因此需要通过系统调用完成,它最终会调用`sys_cputs()`,这个函数又会调用`lib/syscall.c`中的`syscall()`,这个函数先通过汇编将函数参数压入通用寄存器,通过这个方式传递参数给即将产生的`int 0x30`中断.  

```
static inline int32_t
syscall(int num, int check, uint32_t a1, uint32_t a2, uint32_t a3, uint32_t a4, uint32_t a5)
{
	int32_t ret;

	// Generic system call: pass system call number in AX,
	// up to five parameters in DX, CX, BX, DI, SI.
	// Interrupt kernel with T_SYSCALL.
	//
	// The "volatile" tells the assembler not to optimize
	// this instruction away just because we don't use the
	// return value.
	//c
	// The last clause tells the assembler that this can
	// potentially change the condition codes and arbitrary
	// memory locations.

	asm volatile("int %1\n"
		     : "=a" (ret)
		     : "i" (T_SYSCALL),
		       "a" (num),
		       "d" (a1),
		       "c" (a2),
		       "b" (a3),
		       "D" (a4),
		       "S" (a5)
		     : "cc", "memory");

	if(check && ret > 0)
		panic("syscall %d returned %d (> 0)", num, ret);

	return ret;
}


```

之后交给中断处理程序来处理相应的函数.这里会通过`trap_dispatch()`传递给`kern/syscall.c`中的`syscall()`,该函数根据系统调用调用号调用`kern/print.c`中的`cprintf()`函数,该函数最终调用`kern/console.c`中的`cputchar()`将字符串打印到控制台.当`trap_dispatch()`返回后,`trap()`会调用`env_run(curenv)`;,该函数会将`curenv->env_tf`结构中保存的寄存器快照重新恢复到寄存器中,这样又会回到用户程序系统调用之后的那条指令运行,只是这时候已经执行了系统调用并且寄存器`eax`中保存着系统调用的返回值.任务完成重新回到用户模式CPL=3.  
对比一下普通的函数调用和中断以及系统调用参数传递的区别:
* 普通函数掉用,如果没有特权级的变化,堆栈不会改变,函数调用过程是:
  * 先从右往左先把参数入栈,然后跳转到被调函数的地址
  * 被调函数开始:把ebp压栈(先esp-1,然后把ebp的值存入esp),然后把esp的值赋给ebp.被调用函数通过esp+x(即往回找)来获得参数.  
  * 最后函数会把返回值存入eax寄存器,主调函数通过eax寄存器获得返回值.  
* 中断是先保存所有寄存器的值然后调用其他函数,返回的时候恢复现场,对程序运行没有影响.
* 系统调用先调用普通函数,然后把参数压入eax等通用寄存器来传递参数.中断完成之后返回现场,但会把返回值通过`tf->tf_regs.reg_eax=syscall()`写入eax中,被调用的普通函数把ret写入eax返回给主调函数.    
* 可变参数:



##### 
中断和异常一般是在程序运行过程中发生的,这个时候cpu执行 INT n指令,





#### 杂
bootstack在哪里?   
kernel的初始化代码`entry.S`里,由编译器分配  
并且本质上esp的位置决定了栈的位置,因此只需要把esp置为想要的栈的位置就行了,栈的增长方式和具体细节由编译器提前决定(即设好规则,当什么时候,增长多少)

```
mov	$relocated, %eax
	jmp	*%eax
relocated:

	# Clear the frame pointer register (EBP)
	# so that once we get into debugging C code,
	# stack backtraces will be terminated properly.
	movl	$0x0,%ebp			# nuke frame pointer

	# Set the stack pointer
	#这个位置设置的内核用栈,因为esp决定了栈的位置
	movl	$(bootstacktop),%esp

	# now to C code
	call	i386_init


```

```

.data
###################################################################
# boot stack
###################################################################
	.p2align	PGSHIFT		# force page alignment
	.globl		bootstack
bootstack:
	.space		KSTKSIZE //32K
	.globl		bootstacktop   
bootstacktop:

```
用户进程怎么使用用户栈?  
kernel通过把用户进程的`Trapframe`的`tf_esp`设为`USTKTOP`就行了,而这个栈的物理页会有内核特别申请,映射.  
用户的内核栈由tf_esp0指定,在JOS中,大家共享一个内核栈,或者说这是不一定的,但是具体实现的时候是指向相同的地方,每个CPU有单独的内核栈.  


```

//通过这一步把用户栈设为USTACKTOP,之后esp就在这个位置往下增长
	//由此可以看出,esp决定了栈的位置
	e->env_tf.tf_esp = USTACKTOP;
	e->env_tf.tf_cs = GD_UT | 3;
	// You will set e->env_tf.tf_eip later.

```

在用户进程中,用户是怎么得到`envs`的地址的?(因为envs是定义在kernel里的,用户不能访问,而kernel把这个地方映射给了`UENVS`,用户只能访问这个`UENVS`里的envs,是怎么访问的呢?)  
在用户的`entry.S`文件里,有这样一段:  

```

.data
// Define the global symbols 'envs', 'pages', 'uvpt', and 'uvpd'
// so that they can be used in C as if they were ordinary global arrays.
	.globl envs
	.set envs, UENVS  //这个地方把envs的值设为UENVS,从此,用户进程访问envs就指向UENVS,而不是kernel里原来的envs,下面同理
	.globl pages
	.set pages, UPAGES
	.globl uvpt
	.set uvpt, UVPT
	.globl uvpd
	.set uvpd, (UVPT+(UVPT>>12)*4)

```
日志:`make qemu |tee -a linux.log`   
* 类似于将水流发送到两个方向的三通管,`tee`命令将输出发送到终端以及文件(或作为另一个命令的输入).你可以像这样使用它:`command | tee file.txt`如果该文件不存在,它将自动创建.还可以使用 tee 命令 -a 选项进入附加模式.    
* 管道是一种通信机制,通常用于进程间的通信(也可通过socket进行网络通信),它表现出来的形式将前面每一个进程的输出(stdout)直接作为下一个进程的输入(stdin)
`trap()`中incoming Trapframe的地址:  
此时,这个是内核栈中的Trapframe,因此是内核栈中的地址:0xefffffbc 内核栈顶是:0xf0000000 相差4+4*16=68= sizeof(struct Trapframe)    
如果用户的内核栈和内核用的栈相同,而用户的内核栈每次使用时都默认里面没有内容,从栈底开始,那么就破坏了内核的内容,该怎么办?  
这个时候已经返回内核了,在JOS中,会继续在内核栈中执行,知道最后销毁进程,销毁之后进入`monitor()`,实际上这对内核没有什么影响,因为进入了monitor,但是之后不知道会不会有影响,我猜应该把内核用的栈和内核栈分开,当程序销毁后,再切换到内核用的栈.  
进程从用户态进入内核态的三种方式:
* 异常
* 外围设备中断
* 系统调用
系统调用的三种实现方法:   

