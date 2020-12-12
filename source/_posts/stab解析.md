---
title: stab解析
top: false
cover: false
toc: true
mathjax: true
date: 2020-11-29 01:24:08
password:
summary:
tags:
categories:
---
## stab格式解析
* 阅读材料：http://wwwcdf.pd.infn.it/localdoc/stabs.pdf  

### 1 overview
* stab是可执行文件中的一个部分的信息格式：This debugging information describes features of the source file like line numbers, the types and scopes of variables, and function names, parameters, and scopes.     

<!--more-->

#### 1.2  
* 有三种不同的stab格式，由stab的第一个字区分。指令的名字决定了接下来的四种可能的数据域，并赋予他们不同的意义。这三个名字分别是：.stabs(string) .stabn(number) .stabd(dot)。而IBM的XCOFF汇编器使用.stabx(以及一些其他的数据域比如.file .bi)，而不是 .stabs .stabn .stabd。  
  
大致的三种格式为：  
`.stabs "string" ,type ,other ,desc ,value   `
`.stabn type, other,desc ,value  `
`.stabd type, other, desc  `
`.stabx "string", value, type, sdb-type ` 
对于`.stabn`和`.stabd`来说，他们没有`string`域(`n_strx`域是0). 对于`.stabd`来说,`value`域是隐藏的，默认为当前文件位置。对于`.stabx`来说，`sdb-type`域一般不被使用并且经常被设置为0.其他域经常未被使用且被设置为0.
* `type`域的序号代表了这个符号表的一些基本信息。每一个有效的`type`序号定义了一个不同的符号表类型，并且，`type`定义了`value string desc`等域的确切意义。  

#### 1.3 string域
* 对于大多数的符号表，string域包含一些调试信息。这个域的灵活性使得符号表可扩展。对于某些type，string域只包含一个名字。另一些type里的string域包含更多的复杂信息。
* 大多数type的string域格式是：
  * `"name:symbol-descriptor type-information"`
* name是符号表所代表的符号的名字。name可以省略，意味着这个stab代表一个匿名对象。比如`:t10=*2`定义了type 10 作为type 2 的指针，但是不给这个type名字。省略name域被AIX的dbx和GDB支持。
* `:`号后面的`symbol-descriptor`是一个字母符号，描述这一栏`stab`代表的`symbol`的具体类型。如果`symbol-descriptor`被省略，但是紧接着的是type信息，那这一栏`stab`代表一个局部变量。
* `type-information`格式为：`type-number`或者`type-number=`。单独的`type-number`是一个`type`引用，直接指向一个已被定义的type
* `type-number=` 格式是一个type定义，其中的`number`代表一个将被定义的新的类型。这些type定义可以通过number指向其他type，并且这些`type number`可以紧接着一个=而嵌套定义。
* 对于一个type定义，如果紧接着=号的符号不是一个数字，那么它代表一个类型描述符，并且描述了将要被定义的类型的种类。类型描述符会决定紧接着的value。



   

https://blog.csdn.net/killmice/article/details/38226983