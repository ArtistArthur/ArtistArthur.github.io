---
title: IOmodels
top: false
cover: false
toc: true
mathjax: true
date: 2021-02-06 10:55:12
password:
summary:
tags:
categories:
---

## I/O models in unix network programming 
通过[<UNIX Network Programming(Volume1,3rd)>](https://mathcs.clarku.edu/~jbreecher/cs280/UNIX%20Network%20Programming(Volume1,3rd).pdf) 学习unix下的io模型  
there are  five I/O models that are available to us under Unix:
* blocking I/O
* nonblocking I/O
* I/O multiplexing (select and poll)
* signal driven I/O (SIGIO)
* asynchronous I/O (the POSIX aio_functions)
<!-- more -->
As we show in all the examples in this section, there are normally two distinct phases for
an input operation:
1. Waiting for the data to be ready
2. Copying the data from the kernel to the process
For an input operation on a socket, the first step normally involves waiting for data to
arrive on the network. When the packet arrives, it is copied into a buffer within the kernel.
The second step is copying this data from the kernel's buffer into our application buffer.  
在unix下有五种I/O模型:
* 阻塞I/O
* 非阻塞I/O
* 多路复用I/O
* 信号驱动I/O
* 异步I/O
所有的例子中,输入有两个阶段的操作:
1. 等待数据就位
2. 从内核中copy数据到进程中
3. 