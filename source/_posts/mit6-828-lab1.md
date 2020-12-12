---
title: mit6.828-lab1
top: false
cover: false
toc: true
mathjax: true
date: 2020-10-21 23:00:33
password:
summary:
tags:
categories:
---
## Introduction
lab1的内容或者目的：
* 熟悉x86的汇编语言，搭建环境：下载make qemu等，熟悉PC的开机流程
* examine the boot loader for 6.828 kernel
* initial template for 6.828 kernel ,named JOS
<!--more-->

### a. 搭建环境：git clone qemu
* 完全参照 https://www.cnblogs.com/gatsby123/p/9746193.html
之前用的其他的博客但是没搞好，这个博客解决了  
* 还遇到一个头文件缺失的问题: https://github.com/Ebiroll/qemu_esp32/issues/12 通过添加头文件通过了
学到的东西：  
* 知道了qemu是个跑在linux上的模拟器，可以模拟出各种硬件状态。在这个实验中，qemu的作用是模拟出i386的环境，因为这个实验写出的操作系统是跑在i386上的。
### b. 提交作业
当完成代码后，可以在lab下 make grade,获得评分。  
## Part1：Bootstrap
这个部分不写代码，主要是了解和学习汇编相关知识和开机引导流程
### a.汇编
* 汇编有两种语法，一个是Intel的一个是AT&T。x86一般用Intel语法，NASM（Netwide Assembly)是使用Intel语法的汇编器，GNU使用AT&T语法
* 主要了解几种常见汇编指令   
### b.qemu的使用  
* 配置好qemu后，直接在lab下 make qemu,可以编译运行kernel  
### c. PC的物理地址空间

```

+------------------+  <- 0xFFFFFFFF (4GB)   
|      32-bit      |   
|  memory mapped   |   
|     devices      |   
|                  |   
/\/\/\/\/\/\/\/\/\/\  
  
/\/\/\/\/\/\/\/\/\/\   
|                  |   
|      Unused      |   
|                  |   
+------------------+  <- depends on amount of RAM   
|                  |   
|                  |   
| Extended Memory  |     
|                  |   
|                  |   
+------------------+  <- 0x00100000 (1MB)   
|     BIOS ROM     |   
+------------------+  <- 0x000F0000 (960KB)  
|  16-bit devices, |   
|  expansion ROMs  |   
+------------------+  <- 0x000C0000 (768KB)   
|   VGA Display    |    
+------------------+  <- 0x000A0000 (640KB)   
|                  |   
|    Low Memory    |   
|                  |   
+------------------+  <- 0x00000000 

```

* 16位的Intel 8088/86 有20位地址总线，可以寻址1MB的内存，但是由于数据总线只有16位，所以单个程序段最大只有64KB,为了可以寻址1MB，Intel引入了分段：
  * 逻辑段的开始地址必须是16的倍数，因为段寄存器长为16位；
  * 逻辑段的最大长度为64K，因为指针寄存器长为16位。 
  * 那么1M字节地址空间最多可划分成64K个逻辑段，最少也要划分成16个逻辑段。逻辑段与逻辑段可以相连，也可以不相连，还可以部分重叠。
  * 在实模式下，cpu寻址的方式是：CS:IP  物理地址是： 16*CS+IP
  * 这个博客说得很好  https://www.cnblogs.com/blacksword/archive/2012/12/27/2836216.html  
* 早期的cpu内存，只使用640KB以下的内存(被称为Low Memory)作为随机存取器（RAM），更早的使用的内存更少：16KB，32KB，64KB
* 640KB至1MB中间的384KB内存被保留作为硬件的特殊用途，比如显卡缓冲区和非易失固件内存（？）
* 被保留的最重要的区域是从0x000F0000（960KB）到0x000FFFFF（1MB）的64KB，这里会存放BIOS。BIOS会进行基本的初始化比如显卡检查和硬件检查以及内存检查
* BIOS初始化之后会从适当的位置（软盘硬盘等）加载操作系统
* 从80286和80386开始Intel支持16MB和4GB内存，但是低1MB的基本功能布局没有变化（为了兼容）
* 现代PC在640KB到1MB之间有个hole，把内存分为了低640KB的Low Memory和1MB以上的extended memory
* 并且在32位机器上，4GB的内存顶部也保留了一部分区域给32位的PCI接口设备，因此在64位机器普及后，内存又出现了第二个hole
* 这个课程的操作系统只使用低256MB内存
### d.The ROM BIOS
* 学习IA-32兼容的计算机是怎么开机的
* 打开两个终端，先后分别输入 make qemu-gdb make gdb,在输入make gdb的终端里可以使用相关gdb命令进行调试
  * si代表执行下一个语句并停下来，显示出来的是即将执行的语句，但是还未执行
* BIOS加载进内存后，将从CS=0xf00 IP=0xfff0出执行，这个语句是个jmp语句，跳到CS=0Xf000 IP=0xe05b，原因是0xffff0处于BIOS末端（因为之前的语句是把BIOS加载进来，然后就到了末尾），要执行BIOS，则要跳转到BIOS的入口出
* BIOS的功能是设置中断表、初始化设备，然后寻找一个可引导开机的设备（硬盘软盘等），然后读取开机引导设备的第一个扇区（里面的程序是bootloader）到内存0x7c00处（这个地址是随意的，但是是写死的，也就是说这个地址没什么特殊意义，只是刚好选了它，但它仍要满足一些条件，内存对齐、位置等），再跳转到bootloader的入口，控制权就转移到了bootloader  
## Part2: The Boot Loader
* 一般情况下，或者说老一代的PC其开机设备的第一个扇区存了bootloader，负责开机引导等，现代的PCbootloader会有更复杂的功能和大小
* JOS的bootloader由一个汇编文件:boot/boot.S和C源文件：boot/main.c组成，里面的内容分析：
* boot/boot.S

~~~

#include <inc/mmu.h>
# Start the CPU: switch to 32-bit protected mode, jump into C.
# The BIOS loads this code from the first sector of the hard disk into
# memory at physical address 0x7c00 and starts executing in real mode
# with %cs=0 %ip=7c00.

#.set symbol, expression 汇编意义：设置symbol为expression

.set PROT_MODE_CSEG, 0x8         # kernel code segment selector   
#段选择符的格式:13位索引+1位TI表指示标志（代表是不是全局描述符）+2位RPL
#因此 0x8  0000000000001 0 00
#此处预设代码段选择符,这个选择符代表，索引是1，TI为0表示全局描述符，RPL权限设为最高

.set PROT_MODE_DSEG, 0x10        # kernel data segment selector
#     0x10 0000000000010 0 00
#此处预设数据段选择符,代表索引是2，全局描述符，权限0最高

.set CR0_PE_ON,      0x1         # protected mode enable flag
#保护模式的设置由CR0管理，这里的标识符代表开启保护模式时CR0对应的位的值:最低位置位

.globl start  
start:                 #函数的开始，相当于main
  .code16                     # Assemble for 16-bit mode    
  #让汇编器按照16位代码汇编

  cli                         # Disable interrupts
  cld                         # String operations increment
#关闭中断，设置字符串操作是递增方向
#cld的作用是将direct flag标志位清零
#it means that instructions that autoincrement the source index and destination index (like MOVS) will increase both of them

  # Set up the important data segment registers (DS, ES, SS).

  xorw    %ax,%ax             # Segment number zero
  movw    %ax,%ds             # -> Data Segment
  movw    %ax,%es             # -> Extra Segment
  movw    %ax,%ss             # -> Stack Segment

  # Enable A20:
  #   For backwards compatibility with the earliest PCs, physical
  #   address line 20 is tied low, so that addresses higher than
  #   1MB wrap around to zero by default.  This code undoes this.

  #激活A20地址位,由于需要兼容早期pc，物理地址的第20位绑定为0，所以高于1MB的地址又回到了0x00000
  #激活A20后，就可以访问所有4G内存，就可以使用保护模式
  #激活方式：由于历史原因A20地址位由键盘控制器芯片8042管理。所以要给8042发命令激活A20
  #8042有两个IO端口：0x60和0x64,激活流程为： 先发送0xd1命令到0x64端口 --> 再发送0xdf到0x60

seta20.1:
  inb     $0x64,%al               # Wait for not busy
  #汇编语言有专门的读取端口信息的指令，in out后面的b代表一个字节

  testb   $0x2,%al
  #测试（两操作数作与运算,仅修改标志位，不回送结果）。

  jnz     seta20.1
#发送命令之前，要等待键盘输入缓冲区为空，这通过8042的状态寄存器的第2bit来观察，而状态寄存器的值可以读0x64端口得到。
#上面的指令的意思就是，如果状态寄存器的第2位为1，就跳到seta20.1符号处执行，知道第2位为0，代表缓冲区为空

  movb    $0xd1,%al               # 0xd1 -> port 0x64
  outb    %al,$0x64
#发送0xd1到0x64端口

seta20.2:
  inb     $0x64,%al               # Wait for not busy
  testb   $0x2,%al
  jnz     seta20.2

  movb    $0xdf,%al               # 0xdf -> port 0x60
  outb    %al,$0x60

  # Switch from real to protected mode, using a bootstrap GDT
  # and segment translation that makes virtual addresses 
  # identical to their physical addresses, so that the 
  # effective memory map does not change during the switch.
  #转入保护模式，这里需要指定一个临时的GDT，来翻译逻辑地址。
  #这里使用的GDT通过gdtdesc段定义，它翻译得到的物理地址和虚拟地址相同（段描述符里的段基址为0）
  #所以转换过程中内存映射不会改变

  lgdt    gdtdesc   
  #lgdt指令把gdtdesc的地址加载进gdtr寄存器，代表全局段描述符表

  movl    %cr0, %eax
  orl     $CR0_PE_ON, %eax
  movl    %eax, %cr0
  #开启保护模式
  # Jump to next instruction, but in 32-bit code segment.
  # Switches processor into 32-bit mode.
  #由于进入保护模式，所有地址都应该是：cs:eip

  ljmp    $PROT_MODE_CSEG, $protcseg         #ljmp cs esp
  #Long jump, use 0xfebc for the CS register and 0x12345678 for the EIP register:
  #ljmp $0xfebc, $0x12345678

  .code32                     # Assemble for 32-bit mode
protcseg:
  # Set up the protected-mode data segment registers
  movw    $PROT_MODE_DSEG, %ax    # Our data segment selector
  movw    %ax, %ds                # -> DS: Data Segment
  movw    %ax, %es                # -> ES: Extra Segment
  movw    %ax, %fs                # -> FS
  movw    %ax, %gs                # -> GS
  movw    %ax, %ss                # -> SS: Stack Segment
  
  # Set up the stack pointer and call into C.
  movl    $start, %esp
  call bootmain

  # If bootmain returns (it shouldn't), loop.
spin:
  jmp spin

# Bootstrap GDT
.p2align 2                                # force 4 byte alignment
gdt:
  SEG_NULL				# null seg
  SEG(STA_X|STA_R, 0x0, 0xffffffff)	# code seg
  SEG(STA_W, 0x0, 0xffffffff)	        # data seg

gdtdesc:
  .word   0x17                            # sizeof(gdt) - 1
  .long   gdt                             # address gdt


~~~

* boot/boot.S的功能有一个：
  * 从实模式进入32位保护模式，方法是把CR0最低位置位，即开启了保护模式，然后激活A20使得cpu可以寻址1MB以上的空间，之后设置一个段表，最后调用bootmain，进入main.c
* 



* stab的作用：
  * With the `-g` option, GCC puts in the .s' file additional debugging information, which is slightly transformed by the assembler and linker, and carried through into the nal executable. This debugging information describes features of the source file like line numbers, the types and scopes of variables, and function names, parameters, and scopes of variables, and function names, parameters, and scopes. 