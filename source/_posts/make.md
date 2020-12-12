---
layout: 
title: make manual read notes
date: 2020-12-11 18:51:26
tags:
---
## make manual read notes  
manual download: https://www.gnu.org/software/make/manual/make.pdf  
<!--more-->  
### 1 Overview of make
&emsp;&emsp;make可以自动地决定一个大型项目中的哪些部分需要被重新编译，并且列出重新编译他们的命令(所以make是为了更好地自动编译项目)。
&emsp;&emsp;make可以与任何可以使用命令行的编译器使用，因此几乎所有编程语言都可以用make编译。并且make不仅仅限于编程，还可以用它来描述任何需要自动更新的文件的更新方式。
&emsp;&emsp;为了可以使用make，需要事先写一个叫做`makefile`的文件，里面描述了你的程序里的文件的关系以及提供了如何更新这些文件的命令。比如对于一个程序来说，可执行文件是由目标文件构建或者更新的。
&emsp;&emsp;一旦写好了一个合适的`makefile`，每次更改一些源文件之后，只需要在命令行中调用`make`就可以自动编译你的项目了。
### 2 An Introduction to Makefiles
&emsp;&emsp;你需要一个叫做`makefile`的文件来告诉`make`应该做什么。大多数情况下`makefile`告诉`make`怎么编译和链接一个程序。
