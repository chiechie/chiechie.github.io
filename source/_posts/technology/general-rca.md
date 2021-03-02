---
title: 对IT系统进行根因定位的一种方法
author: chiechie
date: 2021-03-01 17:25:23
categories: 技术类

tags:
- 计算机网络
---

## 问题分析

一个IT系统包括机器，网络，应用。

一个系统中任何一个组成部分都有可能出现故障，从而导致系统瘫痪。


为了减少故障带来的损失，我们需要提前制定预案，即哪些表象对应这哪些故障，从而在故障发生时候，能快速地根据表现识别出故症结所在，进而进行故障处理。
这个方案 是沿着减少故障影响的思路进行，还有一种思路是故障预测的思路，但是不够成熟在此不表。

下面重点说第一个思路

从表象推断到故障症结，就个可以表达为一个根因推断的需求。
即从众多的监控指标，日志信息，事件信息，归纳出导致这些现象出现的原因。
总结一下，由果推因。

先分别看故障出现在这三层会出现什么表现（演绎），以及用什么方法去根据表象推断根因（归纳）
应用之间在逻辑上存在调用关系，可以是一台机器上的调用，也可能是跨机器跨跨网络的调用
应用是部署在机器上的，应用之间的逻辑关系对应着机器之间的网络连接关系

- 应用层故障：如果某个服务b 调用服务a， 但是服务a故障了，比如某个操作系统死机了，或者mysql的请求堵塞了，
那么，会表现为b调用a的时长变长，即边异常。同时a的请求时长上升或者调用成功次数下降，即点异常。
  注意，b并不会异常，如果有服务调用c在调用服务b，然后这个调用并没有用到服务a，服务b的反应就是正常的。
  总结一下，每一个服务的健康度是一个量化指标，被调用的成功率，如果成功率很低，说明这个服务可能是异常，当然也有可能它被下一层牵连。即导致点异常，以及间接导致边异常。
  
- 主机故障：如果某台机器故障，那么部署在这台机器上面所有的应用都危险了，进而间接影响到，调用这些应用的调用方，即导致应用的点异常，以及间接导致边异常。

- 网络故障：网络故障比如网络设备（网线，机房，交换机，路由器）故障，会导致数据在传输过程中丢失，进而影响到两个跨机器的应用之间的调用，即导致 边异常。

画一个图表示一下

![图1-应用/主机/网络三种故障导致的结果](img.png)


## 更进一步的故障的分类
操作系统异常/应用异常/网络异常/自定义异常/容器异常/应用内故障。

操作系统异常

### 操作系统异常

CPU	制造一个或多个CPU cores高负载
Memory	分配一定数量的RAM以及Cache
IO

在IO设备上创作读写压力
Disk	填充一定比例到指定硬盘或路径

### 应用异常
Kill进程	Kill指定进程

### 网络异常
网络拒绝	丢弃匹配到的所有网络流量
网络延时	注入一定延时到匹配的网络流量上
网络超时

注入故障，调用方表现为超时
网络丢包	对匹配的网络流量进行丢包
数据错乱	

### 自定义异常
python自定义异常	自定义python异常


### 容器异常
Kill Pod	杀死指定的Pod
Kill container	杀死指定的容器






## 参考资料
