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
## 1. introduction
lab1的内容或者目的：
* 熟悉x86的汇编语言，搭建环境：下载make qemu等，熟悉PC的开机流程
* examine the boot loader for 6.828 kernel
* initial template for 6.828 kernel ,named JOS
### a. 搭建环境：git clone qemu
* 完全参照 https://www.cnblogs.com/gatsby123/p/9746193.html
之前用的其他的博客但是没搞好，这个博客解决了  
* 还遇到一个头文件缺失的问题: https://github.com/Ebiroll/qemu_esp32/issues/12 通过添加头文件通过了
学到的东西：  
* 知道了qemu是个跑在linux上的模拟器，可以模拟出各种硬件状态。在这个实验中，qemu的作用是模拟出i386的环境，因为这个实验写出的操作系统是跑在i386上的。
### b. 提交作业
这个是针对mit自己的学生的，不知道我可不可以也跑测试评分，之后看到再说

## 2.Part1：Bootstrap
这个部分不写代码，主要是了解和学习汇编相关知识和开机引导流程
### a.汇编
* 汇编有两种语法，一个是Intel的一个是AT&T。x86一般用Intel语法，NASM（Netwide Assembly)是使用Intel语法的汇编器，GNU使用AT&T语法
* 汇编的各种细节（之后写）
### b.qemu的使用  
* 待写
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