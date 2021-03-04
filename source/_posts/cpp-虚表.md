---
title: C++虚表
top: false
cover: false
toc: true
mathjax: true
date: 2020-11-01 18:28:05
password:
summary:
tags: [C++]
categories: [C++]
---
## 虚表的具体细节
* 虚表是cpp实现多态的一个方式，是编译器的具体实现，cpp标准并未规定该如何实现
* 虚表的作用是，对于虚函数来说，可以使得通过父类调用子类的特定方法，而不是父类的方法，可以做到灵活，或者说是一种多态
<!--more-->