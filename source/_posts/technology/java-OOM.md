---
title: JVM的自动内存管理
author: chiechie
mathjax: true
date: 2021-03-22 10:02:13
tags:
- 计算机原理
- 虚拟机
- 故障诊断
categories:
- 计算机原理
---

## 背景

![JVM运行时的数据区](img.png)

- 程序计数器（Program Counter Register）是一块较小的内存空间，分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成。是线程私有。
- Java虚拟机栈（Java Virtual Machine Stacks）描述的是Java方法执行的内存模型，每个方法在执行的同时都会创建一个栈帧（Stack Frame)[插图]用于存储局部变量表、操作数栈、动态链接、方法出口等信息。每一个方法从调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中入栈到出栈的过程。也是线程私有的，生命周期和线程一样。有人把Java内存区分为堆内存（Heap）和栈内存（Stack）
- 本地方法栈（Native Method Stack）跟 Java虚拟机栈类似。区别是，虚拟机栈为虚拟机执行Java方法（也就是字节码）服务，而本地方法栈则为虚拟机使用到的Native方法服务。
- Java堆（Java Heap）是被所有线程共享的一块内存区域，是Java虚拟机所管理的内存中最大的一块。Java堆是垃圾收集器管理的主要区域，因此很多时候也被称做“GC堆”（Garbage Collected Heap）。
- 方法区（Method Area）也是各个线程共享的内存区域，它用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据，



## OOM异常

Java虚拟机中，很多区域的问题，都会导致内存溢出（（OOM））的异常

## 参考
1. [深入理解JVM-微信读书](https://weread.qq.com/web/reader/9b832f305933f09b86bd2a9)
