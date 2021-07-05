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

> 这篇文章讲了很多方案细节，实际中的需求，以及可以扩展到哪些应用。这篇文章属于detail & application oriented 。
> 
> 现在只看了实现方案，关于实现过程中的一些细微的需求，后面再看。


## 总结

1. 使用日志定位故障的基本假设：The delta sets of execution sequences between these two sets（tes和生产） could reflect the potential deployment failures。
2. 一些提升效率的方法：
    - 对log event去重：removing repetitions and permutations of the sequences. For example, both of the following two sequences: “E1, E2, E3, E3, E5, E6” and “E1, E3, E5, E2, E6” are reduced to the sequence “E1, E2, E3, E5, E6”.
    
3. 向量化：将log sequence表达成一个N-dim的向量，N表示词典的大小，每个词代表一个patter或者一个token list。
   
    $$w(t)=0.5 \times \operatorname{Norm}\left(w_{i d}(t)\right)+0.5 \times w_{c o n}(t)$$
    
    ![Log sequences represented as vectors](img_2.png)

    - Norm is Sigmoid, which normalizes the IDF to [0, 1].
4. 层次聚类：聚类函数--N-维向量的cosine similarity， the cluster distance metric使用的是the maximum distance of all element pairs between two clusters，并且把 distance threshold θ作为stopping criterion for the clustering process
   ![Agglomerative Hierarchical Clustering](img_3.png)
   
5. 提取每个cluster的代表性Log Sequence：计算每个log sequence离其他成员的平均距离作为分数，找出分数最小的那个log sequence，最为代表，时间复杂度为O(n^2),n表示该cluster的成员个数。
6. check生产环境是否产生了新的cluster？对比新cluster和知识库（离线产生的）中的cluster，计算两个cluster的representative log sequence的cosion相似度，使用跟层次聚类一样的距离参数$\theta$作为决策标准。

7. 评估：使用控制实验，先正常运行两个Hadoop应用---WordCount和pagerank，搜集相应的日志。然后注入以下故障以模拟生产环境：
    
    - Machine Down: we turn off one server when the applications  are running to simulate the machine failure.
   -  Network Disconnection: we disconnect one server from the network to simulate the network connection failure.
   - Disk Full: we manually fill up one server’s hard disk when the
applications are running to simulate the disk full failure.
8. 评估看两个方面, 一个是使用日志聚类之后，还需要人check的工作量，一个是定位到故障的准确率。

Measured in terms of #log sequences to be examined. Numbers in brackets indicate the precision values (i.e., the percentage of examined log sequences that are associated with the
actual failures). 
8. 怎么评估聚类？使用NMI（normalized mutual information），a number between 0 and 1.越大越好
9. LogCLuster的不适用范围：非Hadoop应用；测试环境有bug（算法默认测试是标准）；不适合performance相关的异常定位（有时序性，算法在向量化的时候，就把时序性抹去了）.
10. 应用场景之一--rca，某个产品G对原始执行日志做聚类，得到clusters，每一个identified cluster 都分配一宜昌分数，根据cluster的大小/age/用户feedback。当某个服务经历一个live site issue时，在故障发生时，服务团队的工程师使用G去check异常程度最高的几个cluster，。 This allows for more efficient root cause analysis, which in turn leads to improvement in key metrics like “Mean Time to Mitigate” and “Mean Time to Fix”。

![img_4.png](img_4.png)



## 附录

### 难点

1. 数据量大：每天产生的日志达到10TB甚至1PB
2. 故障转移：大部分在线系统都有故障转移（failover）机制，会根据节点的availability 和 performance，动态地给计算节点们分配jobs。系统可以主动杀死某个job，并且在另外一个地方重启，这样就产生很多包含“kill” 和“fail” keywords的logs。如果，只依赖关键词"fail"去搜索异常，势必产生很多误告。

> 故障转移，即当活动的服务或应用意外终止时，快速启用冗余或备用的服务器、系统、硬件或者网络接替它们工作。 故障转移(failover)与交换转移操作基本相同，只是故障转移通常是自动完成的，没有警告提醒手动完成，而交换转移需要手动进行。
> 
> 对于要求高可用和高稳定性的服务器、系统或者网络，系统设计者通常会设计故障转移功能。
> 
> 在服务器级别，自动故障转移通常使用一个“心跳”线连接两台服务器。只要主服务器与备用服务器间脉冲或“心跳”没有中断，备用服务器就不会启用。为了热切换和防止服务中断，也可能会有第三台服务器运行备用组件待命。当检测到主服务器“心跳”报警后，备用服务器会接管服务。有些系统有发送故障转移通知的功能。
> 
> 有些系统故意设计为不能进行完全自动故障转移，而是需要管理员介入。这种“人工确认的自动故障转移”配置，当管理员确认进行故障转移后，整个过程将自动完成。


### 算法流程

![img_1.png](img_1.png)

离线和在线的前半段流程：

collect log messages ---> parse into  log sequence --> log vectorization ---> group similar log sequences into clusters -- > extract a representative log sequence from each cluster

abstract log message(or log event): 对原始日志泛化

```shell
E1: $DATE INFO [ContainerLauncher #$NUMBER]
org.apache.hadoop.mapreduce.v2.app.launcher.ContainerLauncherImp
l: Processing the event EventType:
CONTAINER_REMOTE_CLEANUP for container $CONTAINERID
taskAttempt $ATTEMPTID
...
E6: $DATE INFO [AsyncDispatcher event handler]
org.apache.hadoop.mapreduce.v2.app.job.impl.TaskAttemptImpl:
Diagnostics report from $ATTEMPTID: Container killed by the
ApplicationMaster.
```

log sequence： 属于同一个task ID的log message搜集起来。
(E1, E1, E2, E3, E4, E5, E6) 



## 参考

1. [Log Clustering based Problem Identification for Online Service Systems- 微软](https://netman.aiops.org/~peidan/ANM2018Fall/6.LogAnomalyDetection/LectureCoverage/2016ICSE_Log%20Clustering%20based%20Problem%20Identification%20for%20Online%20Service%20Systems%20.pdf)