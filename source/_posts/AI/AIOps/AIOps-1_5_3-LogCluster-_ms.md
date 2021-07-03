---
title: chapter1.5.3一种日志聚类的方法-LogCluster
author: chiechie
mathjax: true
date: 2021-07-01 18:56:03
categories: 
- AIOps
tags: 
- NLP
- AIOps
- 日志分析
- 论文笔记 
---


## 难点

1. 数据量大：每天产生的日志达到10TB甚至1PB
2. 故障转移：大部分在线系统都有故障转移（failover）机制，会根据节点的availability 和 performance，动态地给计算节点们分配jobs。系统可以主动杀死某个job，并且在另外一个地方重启，这样就产生很多包含“kill” 和“fail” keywords的logs。如果，只依赖关键词"fail"去搜索异常，势必产生很多误告。

> 故障转移，即当活动的服务或应用意外终止时，快速启用冗余或备用的服务器、系统、硬件或者网络接替它们工作。 故障转移(failover)与交换转移操作基本相同，只是故障转移通常是自动完成的，没有警告提醒手动完成，而交换转移需要手动进行。
> 
> 对于要求高可用和高稳定性的服务器、系统或者网络，系统设计者通常会设计故障转移功能。
> 
> 在服务器级别，自动故障转移通常使用一个“心跳”线连接两台服务器。只要主服务器与备用服务器间脉冲或“心跳”没有中断，备用服务器就不会启用。为了热切换和防止服务中断，也可能会有第三台服务器运行备用组件待命。当检测到主服务器“心跳”报警后，备用服务器会接管服务。有些系统有发送故障转移通知的功能。
> 
> 有些系统故意设计为不能进行完全自动故障转移，而是需要管理员介入。这种“人工确认的自动故障转移”配置，当管理员确认进行故障转移后，整个过程将自动完成。




## 参考

1. [Log Clustering based Problem Identification for Online Service Systems- 微软](https://netman.aiops.org/~peidan/ANM2018Fall/6.LogAnomalyDetection/LectureCoverage/2016ICSE_Log%20Clustering%20based%20Problem%20Identification%20for%20Online%20Service%20Systems%20.pdf)