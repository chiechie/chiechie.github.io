---
title: 进程,线程和协程
author: chiechie
mathjax: true
date: 2021-06-27 12:16:07
tags:
- 计算机原理
- 线程
- 进程
categories:
- 阅读
---



## 基本定义


### 进程（process）

进程（process）有时候也称做任务，是指一个程序运行的实例。

我们想要计算机要做一项任务（task），我们会写一段代码（python/java等）。

编译器将它翻译成二进制代码--机器的语言。

但是此时不执行这段断码的话，就还是一段静态程序。

当执行起来的时候，就变成了一个进程。



#### 进程的3种状态

从程序员的角度，可以认为进程总是处于下面三种状态之一：

- 运行。进程要么在CPU上执行，要么在等待被执行且最终会被内核调度
- 停止。进程的执行被挂起（suspend），且不会被调度。当收到SIGSTOP/SIGTSTP/SIDTTIN/SIGTTOU信号时，进程就停止，直收到SIGCONT信号，这时，进程再次运行
- 终止。进程永远的停止了。进程会因为三种原因终止:
  
    1. 收到一个终止进程的信号
    2. 从主程序返回；
    3. 调用exit函数。

#### 创建子进程

1. 子进程得到与父进程用户级虚拟地址空间相同的（但是独立的）一份拷贝，包括文本、数据和bss段、堆以及用户栈。
2. 子进程还获得与父进程任何打开文件描述符相同的拷贝。
3. 父进程和新创建的子进程之间最大的区别在于它们有不同PID。
4. fork函数调用一次，返回两次，父进程中，fork返回子进程的PID，子进程中fork返回0。

#### 进程的地址空间

1. 每个程序都能看到一片完整连续的地址空间，这些空间并没有直接关联到物理内存，只是操作系统提供的对内存一种抽象，在程序的运行时，会将虚拟地址映射到物理地址。
2. 进程的地址空间是分段的，存在所谓的数据段，代码段，bbs段，堆，栈等等。每个段都有特定的作用，下面这张图介绍了进程地址空间中的划分。
  ![](img_2.png)
3. 对于32位的机器来说，虚拟的地址空间大小就是4G，可能实际的物理内存大小才1G到2G，意味着程序可以使用比物理内存更大的空间。
  
  1. 从0xc000000000到0xFFFFFFFF共1G的大小是内核地址空间，余下的低地址3G是用户地址空间。
  2. Code VMA: 即程序的代码段，CPU执行的机器指令部分。通常，这一段是可以共享的，即多线程共享进程的代码段。并且，此段是只读的，不能修改。
  3. Data VMA: 即程序的数据段，包含ELF文件在中的data段和bss段。
  4. 堆和栈: new或者malloc分配的空间在「堆」上，需要程序猿维护，如果没有主动释放堆上的空间，进程运行结束后会自动释放。「栈」上的是函数栈临时的变量，还有程序的局部变量，自动释放。
  5. 共享库和mmap内容映射区：位于栈和堆之间，例如程序使用的printf，函数共享库printf.o固定在某个物理内存位置上，让许多进程映射共享。mmap是一个系统函数，可以把磁盘文件的一部分直接映射到内存，这样文件中的位置直接就有对应的内存地址。
  6. 命令行参数: 程序的命令行参数
  7. 环境变量：类似于Linux下的PATH，HOME等环境变量，子进程会继承父进程的环境变量。

### 线程（threads）

一个进程中的执行的单位。

线程（thread）：能并行运行，并且与他们的父进程（创建他们的进程）共享同一地址空间（一段内存区域）和其他资源的轻量级的进程


### 协程（CoRoutines）


Co：即corperation，Routines即函数。

协程（CoRoutines）：即用来实现functions 即corperate with each other。

![img_1.png](img_3.png)

不同的编程语言有不同的实现协程的方式，在python和js里面，用的较多的就是yield

用python实现一个协程

```python
def my_coroutine_body(*args):
    while True:
        # Do some funky stuff
        *args = yield value_im_returning
        # Do some more funky stuff

my_coro = make_coroutine(my_coroutine_body)

x = 0
while True:
   # The coroutine does some funky stuff to x, and returns a new value.
   x = my_coro(x)
   print x
```
### 生成器(generator)

A generator is essentially a cut down (asymmetric) coroutine. 

The difference between a coroutine and generator is that a coroutine can accept arguments after it's been initially called, whereas a generator can't.


## 应用 vs 线程 vs 进程

一个应用，比如chrome，可能会启动多个进程（多个网页）, 一个进有多个线程。

进程和线程的区别：

• 进程（火车）间不会相互影响，一个线程（车厢）挂掉将导致整个进程（火车）挂掉
• 线程（车厢）在进程（火车）下行进
• 一个进程（火车）可以包含多个线程（车厢）
• 不同进程（火车）间数据很难共享，同一进程（火车）下不同线程（车厢）间数据很易共享

线程之间的通信更方便，同一进程下的线程共享全局变量、静态变量等数据，
进程之间的通信需要以通信的方式（IPC)进行
• 进程要比线程消耗更多的计算机资源
• 进程间不会相互影响，一个线程挂掉将导致整个进程挂掉
• 进程可以拓展到多机，线程最多适合多核
• 进程使用的内存地址可以上锁，即一个线程使用某些共享内存时，其他线程必须等它结束，才能使用这一块内存。－"互斥锁"
• 进程使用的内存地址可以限定使用量－“信号量”

进程提供给应用程序关键的抽象：
1. 一个独立的逻辑控制流，它提供一个假象，好像我们的程序独占地使用处理器
2. 一个私有的地址空间，它提供一个假象，好像我们的程序独占地使用存储器系统


## 硬件多线程vs软件多线程

CPU架构演进路线：
多cpu--->超线程-->多core

https://stackoverflow.com/questions/680684/multi-cpu-multi-core-and-hyper-thread

其中的超线程（hyper thread）指的硬件多线程，如下图，相当于给一个core，虚拟化为2个core，可以更方便压榨计算机性能


## 多进程 or 多线程？

多进程更稳定，但是多线程能达到更高的计算效率

![左边是单线程，右边是多线程](img_1.png)

多线程的优势：

- 响应性：比如启动一个网页（启动一个浏览器进程），可以同时并行做个事情，如浏览/下载/问答（并行启动多个线程）。
- 资源共享：一个进程上的所有线程共享同一份内存，这样能够让机器的使用效率更高，可以做更多的复杂的事情。---赋能/增效 
- 更经济： 多进程浪费资源，因为创建1个进程需要分配很多内存和资源，相比之下，创建和切换线程的成本小的多。另外，完成一个复杂的任务，多线程共用一份底层资源，多进程就需要把资源复制几份。又浪费了一遍。--降本
- 充分压榨多处理器的架构：

> 大中台类似多线程，烟囱式开发类似多进程

## 实践

### 练习1-模拟单线程CPP的进程管理

[leetcode](https://leetcode-cn.com/problems/single-threaded-cpu/)的题目，



需求：实现一个任务管理/编排的机制，即，输入一堆任务，每个任务的计划执行时间/执行时长都有，现在有一台单线程CPU，如何安排这些任务的执行顺序？

分析：
设想应用场景，医院的排队系统/有一堆任务要排期。

一遍在执行已有的任务，一边有源源不断的接到新的任务，

每执行完一个任务，check一下距离上次检查，有多少新任务进来了，加到任务池里面，从里面选出最容易的。

设计1个数据结构：

1个是优先队列，存放候选任务，并且按照执行时间长短从小到到排序。


```python
import heapq


tasks = [[1,2],[2,4],[3,2],[4,1]]
n = len(tasks)
timestamp = 1
candidate_list = []
new_task = []
j = 0
for i in range(n):
    while (j < n) and (tasks[j][0] <= timestamp):
        heapq.heappush(candidate_list, (tasks[j][1], j))
        j+=1
        print(j, n)
    print(candidate_list)
    process, index = heapq.heappop(candidate_list)
    print(candidate_list)
    new_task.append(index)
    timestamp += process
new_task
```



## 参考
1. [biaodianfu-zhihu](https://www.zhihu.com/question/25532384/answer/411179772)
2. [thred](https://www.youtube.com/watch?v=usyg5vbni34)
3. [计算机原理系列-blog](https://www.junmajinlong.com/os/multi_cpu/)
4. [关于CPU上的高速缓存](https://www.junmajinlong.com/os/cpu_cache/)
5. [What are CoRoutines in Programming?YOUTUBE](https://www.youtube.com/watch?v=tqay-vzqSN0)
6. [生成器和协程](https://stackoverflow.com/questions/715758/coroutine-vs-continuation-vs-generator
)
7. [进程、线程及其内存模型](https://buptjz.github.io/2014/04/23/processAndThreads)
8. 深入理解操作系统