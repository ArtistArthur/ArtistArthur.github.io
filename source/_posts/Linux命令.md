---
title: Linux命令
top: false
cover: false
toc: true
mathjax: true
date: 2020-09-21 19:33:52
password:
summary:
tags:
categories:
---

## 用到的Linux常用命令及解释
<!--more-->
uname : 英文全拼 unix name  用于显示系统信息 可显示电脑以及操作系统的相关信息
-a 或 --a 显示全部的信息
-m 或 --machine 显示电脑类型
-n 或 --nodename 显示在网络上的主机名称
-r 或 --release 显示操作系统的发行编号
-s 或 --sysname 显示操作系统名称
-v 显示操作系统的版本

apt : advanced package tool  
apt-get  xxx  install  

cp ：英文全拼 copy file 用于复制文件或目录
cp [options] source dest  

pwd:print working dictory 显示当前目录的路径  

~:用户根目录

#是管理员 $是用户
了解了
yum : yellow dogUpdater， Modified
sudo : superuser do  
rm -rf: rm remove -r delete files recrusively -f force  delete without warning
## 杂乱的知识
大端：数据低字节在内存高位 可以优先看到符号位快速判断正负和大小
小端：数据高字节在内存低位 可以优先进行计算最后考虑符号位
现在intel的80x86用小端
ARM芯片默认用小端，可切换到大端
MIPS芯片采用大端，可切换
网络序为大端  

## 简写来源
* 寄存器的ax,bx,cx,dx一般叫做通用寄存器，但也有说法是:Accumulator Base Counter Data    
## 待练习
vim的使用：以后用vim语法写题
























## TCP

TCP向应用层提供一种面向连接的、可靠的字节流服务。
面向连接意味着两个使用TCP的应用（通常是一个客户和一个服务器）在彼此交换数据之前必须先建立一个TCP连接。
在一个TCP连接中，仅有两方进行彼此通信，广播和多播不能用于TCP。
