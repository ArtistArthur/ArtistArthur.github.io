---
title: 面试总结
top: false
cover: false
toc: true
mathjax: true
date: 2021-02-18 17:28:42
password:
summary:
tags:
categories:
---
## 什么是进程什么是线程？
进程是资源分配的基本单位。 
线程是资源调度的基本单位。  
一个进程包含一个或者多个线程。cpu上运行的是一个个线程。  
进程拥有自己的地址空间、栈、堆、寄存器等。  
线程拥有线程栈、寄存器、程序计数器等，一个进程中的线程共享进程的地址空间、全局变量、静态变量、堆等。  
<!-- more -->
## 进程间通信的方式？
管道、命名管道：在  
信号：  
信号量：一个信号量代表了一种有限的资源，  
共享内存：可以通过页表映射完成  
消息队列：  
socket：  
客户端：`socket()` `connect()`  
服务器端：  
  
## 线程间通信的方式？
1. 锁：
* 互斥锁
* 自旋锁
* 读写锁
* 条件变量

2. 信号  
3. 信号量  
4. 屏障？？？？？？？？ 
 
线程间通信的目的主要是为了线程同步，所以线程中没有像进程中用于数据交换的通信机制。  

## 死锁

### 什么是死锁？
死锁是两个或者多个进程在竞争资源时产生的一种阻塞现象。  
两个或多个进程中，一个进程请求另一个进程所拥有的资源，这种占有和请求的关系形成了一条环，导致这些进程都被阻塞无法推进。  
### 死锁产生的原因
系统提供的资源不能满足进程的要求。（资源的有限性）   
进程推进顺序的不合理。  
### 死锁产生的必要条件
1. 请求和保持：进程因未分配到新的资源而受阻，但对已占有的资源又不释放。  
2. 互斥：进程占有资源后排斥其他进程使用该资源。
3. 不可剥夺：正在是使用的资源不可剥夺，只能由占有资源的进程自动释放，不能被其他进程强行占用。
4. 环路等待：存在进程的循环等待链，前一进程占有的资源正是后一进程所请求的资源，结果就形成了循环等待的僵持局面。 

### 死锁的解决
预防死锁、发现死锁、解除死锁，具体来说：  
* 打破互斥条件：将独占性资源改造成虚拟资源，但是大部分资源不可改造
* 打破不可抢占条件：当一进程占有一独有资源后又申请一独占资源而无法被满足则释放原独占的资源
* 打破请求和保持条件：采用资源预先分配策略，即进程运行前就申请全部资源，不能被满足就不运行。优点是易于实现，缺点是资源利用率不高
* 打破环路等待条件：实现资源有序分配策略，将每种资源按序排列，进程只能按序递增申请资源，按序递减释放资源，只有低编号的资源得到满足才能申请高编号的资源，高编号资源被释放才能释放低编号资源，这样就打破了循环等待链。

相关算法：
* 有序资源分配算法：编号。
* 银行家算法：计数、替换。  

## socket编程
Socket通信的数据传输方式，常用的有两种：
1. SOCK_STREAM：表示面向连接的数据传输方式。数据可以准确无误地到达另一台计算机，如果损坏或丢失，可以重新发送，但效率相对较慢。常见的 http 协议就使用 SOCK_STREAM 传输数据，因为要确保数据的正确性，否则网页不能正常解析。
2. SOCK_DGRAM：表示无连接的数据传输方式。计算机只管传输数据，不作数据校验，如果数据在传输中损坏，或者没有到达另一台计算机，是没有办法补救的。也就是说，数据错了就错了，无法重传。因为 SOCK_DGRAM 所做的校验工作少，所以效率比 SOCK_STREAM 高。

### 创建socket 
`#include<sys/socket.h>`  
`int socket(int af, int type, int protocol);`  
`af`是一个地址描述符，仅支持`AF_INET`格式，也就是`ARPA Internet`地址格式  
`type`指定socket类型，支持的类型有：`SOCKET_STREAM`(TCP协议)、`SOCKET_DGRAM`(UDP协议)  
`protocol`指定协议，如果调用者不想指定，则传入0。  
若无错误发生，`socket()`返回引用新套接口的描述字。否则的话，返回`INVALID_SOCKET`错误，应用程序可通过`WSAGetLastError()`获取相应错误代码。  
```c

  /**********************************************************
   * create an IPv4/TCP socket, not yet bound to any address
   *********************************************************/
  int fd = socket(AF_INET, SOCK_STREAM, 0);
  if (fd == -1) {
    printf("socket: %s\n", strerror(errno));
    exit(-1);
  }
```
### socket的绑定
```c
  /**********************************************************
   * internet socket address structure, for the remote side
   *********************************************************/
  struct sockaddr_in sin;
  sin.sin_family = AF_INET;
  sin.sin_addr.s_addr = inet_addr(server);
  sin.sin_port = htons(port);

  if (sin.sin_addr.s_addr == INADDR_NONE) {
    printf("invalid remote IP %s\n", server);
    exit(-1);
  }
```
创建了一个socket后，把这个socket绑定到相应的ip和端口。  


## cpp的四种cast
有时我们希望显式地将对象强制转换成另一种类型。  
`cast-name<type>(expression);`  
`cast-name`有`static_cast` `dynamic_cast` `const_cast`和`reinterpret_cast`  
### `stacit_cast`
任何具有明确定义的类型转换，只要不包含底层const（指针本身是const叫做顶层const，指针指向的对象是const叫底层const）都可以使用`static_cast`。  
当需要把一个较大的算术类型赋值给较小的类型时，它非常有用。  




## malloc和new的区别
1. 申请的内存所在位置  
new操作符从自由存储区（free store）上为对象动态分配内存空间，而malloc函数从堆上动态分配内存。自由存储区是C++基于new操作符的一个抽象概念，凡是通过new操作符进行内存申请，该内存即为自由存储区。而堆是操作系统中的术语，是操作系统所维护的一块特殊内存，用于程序的内存动态分配，C语言使用malloc从堆上分配内存，使用free释放已分配的对应内存。  
那么自由存储区是否能够是堆（问题等价于new是否能在堆上动态分配内存），这取决于operator new 的实现细节。自由存储区不仅可以是堆，还可以是静态存储区，这都看operator new在哪里为对象分配内存。  
特别的，new甚至可以不为对象分配内存！定位new的功能可以办到这一点：  
`new(place_address)type`
place_address为一个指针，代表一块内存的地址。当使用上面这种仅以一个地址调用new操作符时，new操作符调用特殊的operator new，也就是下面这个版本：  
`void* operator new (size_t,void *)` //不允许重定义这个版本的operator new
这个operator new不分配任何的内存，它只是简单地返回指针实参，然后右new表达式负责在place_address指定的地址进行对象的初始化工作。  
2. 返回类型安全性
new操作符内存分配成功时，返回的是对象类型的指针，类型严格与对象匹配，无须进行类型转换，故new是符合类型安全性的操作符。而malloc内存分配成功则是返回void * ，需要通过强制类型转换将void*指针转换成我们需要的类型。  
类型安全很大程度上可以等价于内存安全，类型安全的代码不会试图访问自己没被授权的内存区域。  
3. 内存分配失败时的返回值
new内存分配失败时，会抛出bac_alloc异常，它不会返回NULL；malloc分配内存失败时返回NULL。  
在使用C语言时，我们习惯在malloc分配内存后判断分配是否成功：  

```c
int *a  = (int *)malloc ( sizeof (int ));
if(NULL == a)
{
    ...
}
else 
{
    ...
}
```
从C语言走入C++阵营的新手可能会把这个习惯带入C++： 
```c
int * a = new int();
if(NULL == a)
{
    ...
}
else
{   
    ...
}
```
实际上这样做一点意义也没有，因为new根本不会返回NULL，而且程序能够执行到if语句已经说明内存分配成功了，如果失败早就抛异常了。正确的做法应该是使用异常机制：  
```c++
try
{
    int *a = new int();
}
catch (bad_alloc)
{
    ...
}
```
如果你想顺便了解下异常基础，可以看http://www.cnblogs.com/QG-whz/p/5136883.htmlC++ 异常机制分析。  
4. 是否需要指定内存大小
使用new操作符申请内存分配时无须指定内存块的大小，编译器会根据类型信息自行计算，而malloc则需要显式地指出所需内存的尺寸。  
```cpp
class A{...}
A * ptr = new A;
A * ptr = (A *)malloc(sizeof(A)); //需要显式指定所需内存大小sizeof(A);
```
当然了，我这里使用malloc来为我们自定义类型分配内存是不怎么合适的，请看下一条。  
5. 是否调用构造函数/析构函数
使用new操作符来分配对象内存时会经历三个步骤：  
第一步：调用operator new 函数（对于数组是operator new[]）分配一块足够大的，原始的，未命名的内存空间以便存储特定类型的对象。  
第二步：编译器运行相应的构造函数以构造对象，并为其传入初值。  
第三部：对象构造完成后，返回一个指向该对象的指针。  
使用delete操作符来释放对象内存时会经历两个步骤：  
第一步：调用对象的析构函数。
第二步：编译器调用operator delete(或operator delete[])函数释放内存空间。
总之来说，new/delete会调用对象的构造函数/析构函数以完成对象的构造/析构。而malloc则不会。如果你不嫌啰嗦可以看下我的例子：

class A
{
public:
    A() :a(1), b(1.11){}
private:
    int a;
    double b;
};
int main()
{
    A * ptr = (A*)malloc(sizeof(A));
    return 0;
}
在return处设置断点，观看ptr所指内存的内容：



可以看出A的默认构造函数并没有被调用，因为数据成员a,b的值并没有得到初始化，这也是上面我为什么说使用malloc/free来处理C++的自定义类型不合适，其实不止自定义类型，标准库中凡是需要构造/析构的类型通通不合适。

而使用new来分配对象时：

int main()
{
    A * ptr = new A;
}
查看程序生成的汇编代码可以发现，A的默认构造函数被调用了：



6.对数组的处理
C++提供了new[]与delete[]来专门处理数组类型:

A * ptr = new A[10];//分配10个A对象
使用new[]分配的内存必须使用delete[]进行释放：

delete [] ptr;
new对数组的支持体现在它会分别调用构造函数函数初始化每一个数组元素，释放对象时为每个对象调用析构函数。注意delete[]要与new[]配套使用，不然会找出数组对象部分释放的现象，造成内存泄漏。

至于malloc，它并知道你在这块内存上要放的数组还是啥别的东西，反正它就给你一块原始的内存，在给你个内存的地址就完事。所以如果要动态分配一个数组的内存，还需要我们手动自定数组的大小：

int * ptr = (int *) malloc( sizeof(int) );//分配一个10个int元素的数组
7.new与malloc是否可以相互调用
operator new /operator delete的实现可以基于malloc，而malloc的实现不可以去调用new。下面是编写operator new /operator delete 的一种简单方式，其他版本也与之类似：

void * operator new (sieze_t size)
{
    if(void * mem = malloc(size)
        return mem;
    else
        throw bad_alloc();
}
void operator delete(void *mem) noexcept
{
    free(mem);
}
8.是否可以被重载
opeartor new /operator delete可以被重载。标准库是定义了operator new函数和operator delete函数的8个重载版本：

//这些版本可能抛出异常
void * operator new(size_t);
void * operator new[](size_t);
void * operator delete (void * )noexcept;
void * operator delete[](void *0）noexcept;
//这些版本承诺不抛出异常
void * operator new(size_t ,nothrow_t&) noexcept;
void * operator new[](size_t, nothrow_t& );
void * operator delete (void *,nothrow_t& )noexcept;
void * operator delete[](void *0,nothrow_t& ）noexcept;
我们可以自定义上面函数版本中的任意一个，前提是自定义版本必须位于全局作用域或者类作用域中。太细节的东西不在这里讲述，总之，我们知道我们有足够的自由去重载operator new /operator delete ,以决定我们的new与delete如何为对象分配内存，如何回收对象。

而malloc/free并不允许重载。

1. 能够直观地重新分配内存
使用malloc分配的内存后，如果在使用过程中发现内存不足，可以使用realloc函数进行内存重新分配实现内存的扩充。realloc先判断当前的指针所指内存是否有足够的连续空间，如果有，原地扩大可分配的内存地址，并且返回原来的地址指针；如果空间不够，先按照新指定的大小分配空间，将原有数据从头到尾拷贝到新分配的内存区域，而后释放原来的内存区域。

new没有这样直观的配套设施来扩充内存。

10. 客户处理内存分配不足
在operator new抛出异常以反映一个未获得满足的需求之前，它会先调用一个用户指定的错误处理函数，这就是new-handler。new_handler是一个指针类型：

namespace std
{
    typedef void (*new_handler)();
}
指向了一个没有参数没有返回值的函数,即为错误处理函数。为了指定错误处理函数，客户需要调用set_new_handler，这是一个声明于的一个标准库函数:

namespace std
{
    new_handler set_new_handler(new_handler p ) throw();
}
set_new_handler的参数为new_handler指针，指向了operator new 无法分配足够内存时该调用的函数。其返回值也是个指针，指向set_new_handler被调用前正在执行（但马上就要发生替换）的那个new_handler函数。

对于malloc，客户并不能够去编程决定内存不足以分配时要干什么事，只能看着malloc返回NULL。

总结
将上面所述的10点差别整理成表格：

特征	new/delete	malloc/free
分配内存的位置	自由存储区	堆
内存分配失败返回值	完整类型指针	void*
内存分配失败返回值	默认抛出异常	返回NULL
分配内存的大小	由编译器根据类型计算得出	必须显式指定字节数
处理数组	有处理数组的new版本new[]	需要用户计算数组的大小后进行内存分配
已分配内存的扩充	无法直观地处理	使用realloc简单完成
是否相互调用	可以，看具体的operator new/delete实现	不可调用new
分配内存时内存不足	客户能够指定处理函数或重新制定分配器	无法通过用户代码进行处理
函数重载	允许	不允许
构造函数与析构函数	调用	不调用
malloc给你的就好像一块原始的土地，你要种什么需要自己在土地上来播种



而new帮你划好了田地的分块（数组），帮你播了种（构造函数），还提供其他的设施给你使用:



当然，malloc并不是说比不上new，它们各自有适用的地方。在C++这种偏重OOP的语言，使用new/delete自然是更合适的。

感谢您的耐心阅读。

原文：






id to struct 




ps
generation = (e->env_id + (1 << ENVGENSHIFT)) & ~(NENV - 1);//0x000fff 0xfff000
	if (generation <= 0)	// Don't create a negative env_id.
		generation = 1 << ENVGENSHIFT;
	e->env_id = generation | (e - envs);
hash 
  数目小   
  空间浪费 



STL的实现string等  
找项目 
程序员的自我修养—链接、装载与库  
锁https://www.jianshu.com/p/5725db8f07dc  
linux用户抢占和内核抢占详解(概念, 实现和触发时机)--Linux进程的管理与调度(二十）https://cloud.tencent.com/developer/article/1368153  
https://github.com/gatsbyd/melon  
Linux内核同步机制之（四）：spin lockhttp://www.wowotech.net/kernel_synchronization/spinlock.html
系统调用的三种方式https://blog.csdn.net/QFFQFF/article/details/76762232
简单的聊天室实现（上）：通信-SOCKEThttps://zhuanlan.zhihu.com/p/24475299  
Example: Nonblocking I/O and select() https://www.ibm.com/support/knowledgecenter/ssw_ibm_i_72/rzab6/xnonblock.htm  
MIT 6.828 JOS/XV6 LAB4-partAhttps://www.cnblogs.com/bdhmwz/p/5105325.html
TLB
unordered_map
deque
迭代器失效

csapp


管道
共享内存
rpc
linux文件系统
内核空间是怎么同步的
僵尸进程
孤儿进程
http tcp实现，客户端发送请求，服务器发送应答，三次握手，
请求格式：请求行：请求url、请求协议版本、请求方法（get获取页面、put提交数据、head）；
请求头：键值对（通信具体细节：close（短连接）、keepalive（长连接）、）；
请求数据（body） 空行请求结束
服务器收到后:根据url决定；
响应：响应行（协议版本、状态码（1xx等待，过程中、2xx成功、3xx几重定向、4xx客户端失败、5xx服务器端失败）、状态描述）；
响应头部：键值对、内容长度；
响应数据：html、css文件 空行结束  
短链接：每次tcp三次握手只发生一次请求和响应，然后断开
长连接：保持tcp连接
一次点击请求多次
响应头里可以设置超时时间   KeyAlive
多路复用：http2支持，多个请求用一个tcp连接；不管请求的顺序是怎么样，可以随意次序回应
url