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
# Linux命令练习
shell不仅仅是用户命令解释器，同时一种强大的编程语言，Linux缺省的shell是bash  
`$$`Shell本身的PID（ProcessID）  
`$!`Shell最后运行的后台Process的PID  
`$?`最后运行的命令的结束代码（返回值）  
`$#`添加到Shell的参数个数  
`$0`Shell本身的文件名  
`$1～$n`添加到Shell的各参数值。$1是第1参数、$2是第2参数…。  
`du`disk usage 显示当前目录下文件和目录的大小  
## 用到的Linux常用命令及解释
uname : 英文全拼 unix name  用于显示系统信息 可显示电脑以及操作系统的相关信息
-a 或 --a 显示全部的信息
-m 或 --machine 显示电脑类型
-n 或 --nodename 显示在网络上的主机名称
-r 或 --release 显示操作系统的发行编号
-s 或 --sysname 显示操作系统名称
-v 显示操作系统的版本

<!--more-->
apt : advanced package tool  
apt-get  xxx  install  

cp :英文全拼 copy file 用于复制文件或目录
cp [options] source dest  

pwd:print working dictory 显示当前目录的路径  

~:用户根目录

#是管理员 $是用户
了解了
yum : yellow dogUpdater， Modified
sudo : superuser do  
rm -rf: rm remove -r delete files recrusively -f force  delete without warning  
`ps -ef | grep tomcat`ps命令用于报告当前系统进程状态,-e参数表示显示所有用户所有进程,-f参数表示全格式显示  
grep全称是Globally search a Regular Expression and Print,能使用特定模式匹配(包括正则表达式)搜索文本,并默认输出匹配行,所以用管道连接后,这个命令就表示显示所有进程,并且格式化输出,然后用“tomcat”字符串来过滤每一行,得到最终的输出结果  


## vim语法
1.在命令模式下输入v进入自由选取模式,选择需要剪切的文字，按下d可以剪切.  
2.其他命令模式下剪切命令:   
``` 
dd:剪切当前行    
ndd:n表示大于1的数字，剪切n行  
dw:从光标处剪切至一个单词的末尾，包括空格  
de:从光标处剪切至一个单词的末尾，不包括空格  
d$:从当前光标剪切到行末  
d0:从当前光标位置（不包括光标位置）剪切之行首  
d3l:从光标位置（包括光标位置）向右剪切3个字符  
d5G:将当前行（包括当前行）至第5行（不包括它）剪切  
d3B:从当前光标位置（不包括光标位置）反向剪切3个单词  
dH:剪切从当前行至所显示屏幕顶行的全部行  
dM:剪切从当前行至命令M所指定行的全部行  
dL:剪切从当前行至所显示屏幕底的全部行   
```
3.复制
```
yy：复制当前行
nyy：n表示大于1的数字，复制n行
yw：从光标处复制至一个单词的末尾，包括空格
ye：从光标处复制至一个单词的末尾，不包括空格
y$：从当前光标复制到行末
y0：从当前光标位置（不包括光标位置）复制之行首
y3l：从光标位置（包括光标位置）向右复制3个字符
y5G：将当前行（包括当前行）至第5行（不包括它）复制
y3B：从当前光标位置（不包括光标位置）反向复制3个单词
```

## 杂乱的知识
大端:数据低字节在内存高位 可以优先看到符号位快速判断正负和大小
小端:数据高字节在内存低位 可以优先进行计算最后考虑符号位
现在intel的80x86用小端
ARM芯片默认用小端，可切换到大端
MIPS芯片采用大端，可切换
网络序为大端    
XCHG的原理

## 简写来源
* 寄存器的ax,bx,cx,dx一般叫做通用寄存器，但也有说法是:Accumulator Base Counter Data    
* BX —— based register——基地址寄存器
* BP —— base point——基础指针
* SI —— source index——源变址寄存器
* DI —— destination index——目的变址寄存器
## 待练习
vim的使用:以后用vim语法写题
























## TCP

TCP向应用层提供一种面向连接的、可靠的字节流服务。
面向连接意味着两个使用TCP的应用（通常是一个客户和一个服务器）在彼此交换数据之前必须先建立一个TCP连接。
在一个TCP连接中，仅有两方进行彼此通信，广播和多播不能用于TCP。




二进制整数表示方法:  
正数的补码是他自己,复数的补码是先用正数表示然后取反+1.  
两个数相减就是加被减数的补码(取反+1).  
地址相减:先获得差值,把这个差值转换为int,再用int除法,除以地址指向类型的大小.  
因此如果地址差值很大,最高位不为0,则会出现虽然a-b应该是正数,但是出现了负数,且相差一个0xf0000000.