---
title: 时间序列关联性
author: chiechie
mathjax: true
date: 2021-04-14 17:27:26
tags:
- 人工智能
- 时间序列
- 预测
- 论文笔记
categories:
- 技术
---

> performance counter： 性能监视器

## 一些疑问

- 文章说的metric/event/incident指的是什么？跟之前说的的log template有什么关系？
    - incident 就是指的一次事故和故障，我们说故障定位，就是需要对这一个事故找到根本原因。 （感觉也能跟BigPanda的告警关联之后的结果 对应起来），
    - metric： 实值-时间序列（通常有固定的时间间隔），例如CPU使用率等；指标数据( Metrics Data )：描述具体某个对象某个时间点， CPU 百分比等等，指标数据等等。
    - event：就是一个正常的操作比如程序A的启动，这个是需要运维指定的 他感兴趣的某个日志模版。 比如“Out of memory” /启动某个磁盘intensive程序/ 启动某个cpu intensive程序/query-time out alerts/某个告警。感觉也包括BigPanda的rca模块的输入--变更事件）
    - metric：比如cpu使用率
    
实值-时间序列（通常有固定的时间间隔），例如CPU使用率等；指标数据( Metrics Data )：描述具体某个对象某个时间点， CPU 百分比等等，指标数据等等。

- 看完引言，感觉这个event 应该就是人 基于日志数据 指定的 特定的事件类型，比如说程序A的启动，程序B的启动，跟log3C的关联分析的流程做对比，这里的event更像是到了最后一步(4)，只有2类（发生event的样本个数--更简单只有0或者1，不发生event的样本个数），对比不同时间段，发生event的样本个数 和 kpi的相关关系。类似big pandas 的变更事件，或者 其他事件。
- 这篇文章主要是解决的什么问题？ 总结下来就是 希望 找到 跟 incident 有关联的 多个event。 
- 这篇文章
- 先找 服务整体健康度的指标-整体KPI  （如 服务是否可用 或者 请求延迟， 也就是主指标），然后再找 这个 总体KPI 有关联关系的 一批系统指标（副指标）
    - 「关联关系」怎么定义？正常相关性，异常相关性，还是 业务上的关联性？
4. 难点是什么？
- 传统的关联分析方法 不适合 现在的 异构数据分析
- 这个关联性，反应的是 event 和 时序的异常相关性，而不是 正常+异常相关性。
- 5. 这篇文章用的什么分析方法？还可以应用到哪些领域？ 
- 想解决event 和 指标 之间的关联关系
    - 离散变量 的相关性分析 和 因果分析方法。使用的是2-sample问题
        - H. B. Mann and D. R. Whitney. On a test of whether one of two random variables is stochastically larger than the other. The annals of mathematical statistics, 18(1):50{60, 1947.
- 6. 这篇文章没有解决的问题是？这些问题有什么解决方法？
- 时间序列之间的关联关系-pearson相关系数
    - Y. Zhu and D. Shasha. Statstream: Statistical monitoring of thousands of data streams in real time. In VLDB, pages 358{369. VLDB Endowment, 2002
- events之间的关联关系--关联规则算法-aporior算法
    - P. Bahl, R. Chandra, A. Greenberg, S. Kandula, D. A. Maltz, and M. Zhang. Towards highly reliable enter-prise network services via inference of multi-level dependencies. In SIGCOMM, 2007
    - S. Kandula, R. Mahajan, P. Verkaik, S. Agarwal, J. Padhye, and P. Bahl. Detailed diagnosis in enterprise networks. In Proc. SIGCOMM, pages 243{254, 2009.
    - J.-G. Lou, Q. Fu, Y. Wang, and J. Li. Mining dependency in distributed systems through unstructured logs analysis. SIGOPS Operating Systems Review, 41(1):91{96, 2010
- 7. 如果要考虑end2end的解决方案，基于本文给出的方案，还有哪些工作要做？
- 本文还是比较技术流派，只是给了一个 分析指标  和 事件 的相关性 方法，至于怎么从众多原始日志 和 告警中  得到事件，  以及 分析出来的 相关性 怎么应用到 故障定位上面去，没有说。但是这两步 对于实际应用来说 是必不可少的。
- 8. 接着7的问题，其他人是怎么做的？
- tsinghua的另外一片文章是讲怎么 将众多告警收敛成摘要。之前是对指标数据进行告警。
    - 背后的分析链路应该是：
        - 原始日志--> 提取感兴趣的指标-->对指标进行异常检测生成告警-->告警收敛

- [[异常检测-系统日志分析]]
    - 原始日志-->提取事件模版-->得到日志序列计数向量---> 进行异常检测
- bigpandas是怎么做的呢？[[BigPanda-RCA]]
    - 将众多告警基于 业务关系（拓扑数据）收敛成incident，然后将incident跟变更事件进行关联，得到的变更事件就是root cause
- 9. 用来鉴定  日志序列 和 时间序列 关联关系  和 因果关系 的方法是什么？ 
- 10. 更进一步，用来鉴定 日志序列 和 时间序列的因果关系的方法是什么？
- 这里用到了2个trick-都是统计的假设检验的方法，一个是 使用 判断两个随机分布的相似性的指标 来分析因果关系，一个是使用 判断两个随机变量的 大小关系的指标 用来判断 关联到的 指标的方向。具体来说，
    - 判断因果关系是这么做：先将因果关系转换成以2个问题
        - 判断--事件发生之前的 一段时间序列 跟 整个时间序列的相似性
            - 如果差异大，说明指标 --> event
        - 判断--事件发生之后的 一段时间序列 跟 整个时间序列的相似性。
            - 如果差异大，说明event--->指标
    - 判断单调关系是这么做的：
        - 计算event前后，2个子序列的 均值的差
            - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FPKvEWDzw3o.png?alt=media&token=582bc811-36e1-469a-945a-9cd248679e7f)
        - 具体的判断规则如下：
            - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FefWyQfXARM.png?alt=media&token=8e71b7da-8269-437b-bb35-740793503b30)
  
## 文章detail

### 引言

对于大部分在线系统，工程师只能依靠 数据分析的方法来 诊断服务故障，具体需要的数据：

- 服务级别的日志
- 性能监视器：比如吸用的cpu使用率
- 机器/进程/服务级别events：event sequence 就是用来记录某个特定的软件message是否发生。（chiechie--我觉得个特定的massage就是运维指定的），比如“Out of memory”

实验：
- ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FGkHOys2alH.png?alt=media&token=95dde9c8-0249-4033-a027-83bca9a543ff)
- ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2Fy5QnzhCHAl.png?alt=media&token=13774051-eb1f-4ada-a978-9cd7d2bee748)

### 基本概念

- 时序数据（metric）：
- 事件数据（event/log）：指的是记录了特定事件发生的序列，例如内存溢出事件等。日志数据 ( Logging Data )：描述某个对象的是离散的事情，例如有个应用出错，抛出了 NullPointerExcepction，个人认为 Logging Data 大约等同于 Event Data，所以告警信息在我认为，也是一种 Logging Data。
- 事件序列（event sequence）：用来记录某个特定的软件信息，代表某个事情发生，比如An event sequence in an online service is used to record the occurrences of a specific software message indicating that something has happened in the system, e.g., an event sequence of “Out of memory” contains events of “Out of memory”,  which occur when there is not enough memory in the system
- 为了保证产品的服务质量、减少服务宕机时间，从而避免更大的经济损失，对关键的服务事件的诊断显得尤为重要。实际的运维工作中，对服务事件进行诊断时，运维人员可以通过分析与服务事件相关的时序数据，来对事件发生的原因进行分析。虽然这个相关关系不能完全准确的反映真实的因果关系，但是仍然可以为诊断提供一些很好的线索和启发。


### 总结

- 本文将事件数据（E，Event）和时序（S，metric）数据相关关系问题转化为两样本问题（two-sample problem），并使用邻近算法（nearest neighbor method）判断是否相关。主要回答了三个问题：
    - A．E和S之间是否存在相关关系？ 使用nn方法判断相关性是否存在
    - B．若存在相关关系，E和S的时间先后顺序是什么？E先发生，还是S先发生？
    - C．E和S的单调关系。假设S（或者E）先发生，S的增加还是降低导致的E发生？
        - 更进一步，相关性 是否 有特定的时序性，以及大小的先后性，通过分析event之前的子序列 和event之后的子序列的相关性。
- 如图，事件为程序A和B的运行，时序数据为CPU使用率。可以发现，事件（程序A的运行）与时序数据（CPU使用率）存在相关关系，并且是程序A启动会导致CPU使用率升高。![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FMjzFINogsh.png?alt=media&token=9002fb74-9363-4a57-a182-fac68f04bd60)


## 参考
1. [paper](http://www.microsoft.com/en-us/research/wp-content/uploads/2016/07/SIGKDD-2014-Correlating-Events-with-Time-Series-for-Incident-Diagnosis.pdf)
2. [code](https://github.com/jixinpu/aiopstools/tree/master/aiopstools/association_analysis):  360公司对该论文的开源实现
3. [blog](https://mp.weixin.qq.com/s/-NMwaCD4Kzkt4BTnr5JKDQ)
4.  《Capturing, indexing, clustering, and retrieving system history.  》I. Cohen, S. Zhang, M. Goldszmidt, J. Symons, T. Kelly, , and A. Fox.In Proc. SOSP, pages 105-118, 2005