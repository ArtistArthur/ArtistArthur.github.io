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









#### exercise2
完成几个函数:
* `env_init()`:初始化所有的在`envs`数组中的`Enc`结构体，并把他们添加到`env_free_list`指针后面，之后调用`env_init_per_cpu`以设置段管理的相关硬件，它们分别是特权级0(kernel)和特权级3(user)的段。
* `env_setup_vm()`:为新的`environment`分配一个页目录表,并且初始化新的地址空间中的内核部分(通过复制)。
* `region_alloc()`:为新的`environment`分配并映射物理地址。
* `load_icode()`:需要自己实现解析ELF二进制文件的功能(就和`bootloader`里面做的一样),并把二进制文件的内容加载到新的`environment`的用户地址空间中。
* `env_create`:通过`env_alloc()`分配一个`environment`,并且调用`load_icode()`把ELF二进制文件加载进去.
* `env_run`:在用户模式下启动一个给定的`environment`.
代码解析:  
* `env_init()`:  

<!--more-->

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


```c



```