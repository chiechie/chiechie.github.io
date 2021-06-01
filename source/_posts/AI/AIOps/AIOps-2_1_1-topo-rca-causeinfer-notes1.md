---
title: chapter2.1.1 基于拓扑的根因定位CauseInfer1
author: chiechie
date: 2021-05-21 14:41:39
mathjax: true
tags:
- AIOps
- 根因分析
- 微服务
- 图数据
categories: 
- AIOps
---

## 目录
- [chapter0 概览](../AIOps-0-summary/)
- [chapter1 故障发现](../AIOps-1-event-generate/)
	- [chapter1.1 单指标异常检测](../AIOps-1_1-kpi-detector/)
	- [chapter1.3 故障预测](../AIOps-1_2-fault-prediction/)
	- [chapter1.4 指标异常关联](../AIOps-1_4-kpi-correlation/)
	- [chapter1.5 日志聚类](../AIOps-1_5-log-analysis/)
		- [chapter1.5.1 使用logmine加强版做日志聚类](../AIOps-1_5_1-log-analysis_logmine/)
		- [chapter1.5.2 美团日志聚类](../AIOps-1_5_2-log-analysis_meituan/)
- [chapter2 故障定位](../AIOps-2-event-analysis/)
	- [chapter2.1 微服务系统的故障定位](../AIOps-2_1-topo-rca/)
		- [chapter2.1.1 CauseInfer1](../AIOps-2_1_1-topo-rca-causeinfer-notes1/)
		- [chapter2.1.2 CauseInfer2](../AIOps-2_1_2-topo-rca-causeinfer-notes2/)
		- [chapter2.1.3 AIOps挑战赛2020-获奖方案分享](../AIOps-2_1_3-topo-rca-aiops2020/)
		- [chapter2.1.4 AIOps挑战赛2021-demo方案](../AIOps-2_1_4-topo-rca-aiops2021/)
		- [chapter2.1.5 N-Softbei2020比赛](../AIOps-2_1_5-topo-rca-cnsoftbei2020/)
		- [chapter2.1.6 MicroCause](../AIOps-2_1_6-topo-rca-MicroCause)
	- [chapter2.2 多维下钻根因定位](../AIOps-2_2-multi-dimensional-rca/): 暂无
	- [chapter2.3 调用链根因分析](../AIOps-2_3-trace_rca/)
	- [chapter2.4 时间序列关联性分析](../AIOps-2_4-metric_event_correlation/)
- chapter3 故障恢复


> CauseInfer是一个解决故障定位的通用方案，原理是通过在历史数据中学习因果图，进而在发生故障时，推断出故障根因。
>
> 这篇文章对我说太难啃了，主要是因为涉及大量专业领域知识，比如计算机网络，微服务架构等，对这块不熟悉，需要补很多功课。
>
> 输出的文章分成两篇: CauseInfer论文笔记1，即本文，讲背景知识和概要设计，以及直觉型的描述问题和解题思路，都是粗粒度的；[CauseInfer论文笔记2](https://chiechie.github.io/2021/03/03/technology/causeinfer-notes2/)，讲核心技术点，方案的实现逻辑，涉及到数学公式。
> 
> 原则上，第一篇不用技术背景也能看明白，问题是啥，解题思路是啥。第二篇就是给算法人员看的，要出方案了，所以要越详细越好。


## 名词解释

来，补充业务知识

- QPS：Queries Per Second，每秒查询次数
- SLA （service level agreement）:服务等级协议。指的是整个协议，协议的内容包含了SLI，SLO以及恢复的方式和时间等等一系列所构成的协议。
- SLI（service level indicator）:服务等级对象。指的是对象，例如：qps，响应时间，准确性等
- SLO（service level objective）:服务等级目标。指的是目标，例如：qps 99.99% ，响应时间10ms等。


## 几个问题

看完第一遍之后，还有几个问题

- 因果图是什么？
  - 因果图是一个两层分层的图（two layered hierarchical causality graph），因果图 = 指标因果图（刻画局部的依赖关系） + 服务依赖图（刻画整个信息设备的依赖关系）。
  - 服务依赖图（Service Dependency Graph）：粗粒度的信息，表示每台机器上每个服务间的依赖关系，用于定位到服务级别的cause。
  - 指标因果图（Metric Causality Graph）：细粒度的信息，表示表达的是一台机器上，一个服务的指标和多个机器之间的故障传播关系），用于定位到性能异常的真正的元凶。
    
- 指标因果图和服务依赖图是什么关系？
  
    指标因果图中的节点 是 服务依赖图中的节点 的一个属性

- 因果图是怎么构建？
  
  两种方式结合：采集调用链构建拓扑 + 数据分析得到因果关系
  
- 因果图构建好了，如果做因果推断？
  
  深度优先遍历，找有故障的父母亲，直到找到根结点，或者正常的上游。
  
- 一个主机上的服务，构建一个因果图？
  
  对的，每个主机上的每个服务都有一个自己的指标因果图。 然后跨主机的服务调用，就靠服务依赖图表示。
  
- 因果图中不同的机器IP是否是不同的节点？
  
  是的，服务依赖图的粒度更细，是一个机器上的一个服务是一个节点。

-  一个服务的SLO指标是怎么定义的？有什么规则？其他指标是怎么选取的？
   
  一般来说是需要对每一个应用去定制化定义的，但是这里可以用一个统一的指标，即服务相应时长。   

- 根因推断的时候，多个因导致的结果怎么推断出来？
  
  这种复杂的问题解决不了.

- 根因推断的时候，多个根因怎么打分？
  
  找异常程度最高的那个指标。

## 问题描述

关于性能诊断领域，之前大部分工作都很粗糙，只涉及定位异常，但是我们觉得还应该包括根因推断，使得软件维护更高效，并且提供更有用的信息给软件bug。

解决所有的性能诊断问题，没有银弹（silver bullet）。本文也只是诊断性能问题的一个子集。

当review完一些开源软件的bugs之后，我们观察到，很多bugs都可能导致性能问题。

这里，我们仅仅考虑那些造成资源异常的bugs，例如导致物理资源消耗异常（eg CPU） 或者 逻辑资源异常（eg 锁）。

为什么要选择这些bugs？ 因为 不需要关注源代码，就可以搜集资源消耗指标；软件中存在太多这种bugs了。
我们的目的是，找到导致这些指标异常（cpu消耗过多，db被锁）的原因，我们不直接识别软件bug，我们提供一些提示。
举个例子，如果我们定位到根因是，锁的个数异常，则可能是系统中有一个并发bug。

CauseInfer的思路是这样的，构建一个因果图，来捕捉因果（cause-effect ）关系，然后沿着因果路径推断出根因（root cause）。
为了完成这个任务，CauseInfer自动化地构建了一个两层的分层的因果图。

一旦服务级别目标（SLO）异常了，就触发因果推断。
首先将性能异常定位到某个服务（eg，tomcat），通过检测该SLO指标是否异常。
然后通过检测同一台机器上的其他性能指标是否异常，找到根因，
更进一步为了保证效果的稳定，提出了突变点检测方法，基于贝叶斯理论--这个方法比传统的检测方法如cusum要好。
通过在两组benchmark（Olio 和 TPC-W,）做实验评估，发现本方法可以定位出根因，准召分别是80%和85%。
本文的贡献

- 提出了一个新的BCP突变点检测方法，比cusum更稳定，在长期的数据序列上
- 提出了一个轻量的服务依赖方式的 发现方法，通过分析两个服务的流量延迟，很快定位到服务级别的性能异常，
- 提供了一个基于原始PC-算法的因果图构建方法，使用这个因果图，我们肯呢个很准确定位到性能指标级别的性能问题的根因。
- 设计和实现了CauseInfer，可以推断性能问题的根因，可以以较小的代价，较高的准确率找到性能问题的真正根因。


## 系统概览

先描述CauseInfer的框架，并且通过一个简单的例子来描述这个系统的工作流

###  CauseInfer的框架

看图说话--CauseInfer的框架

对于每个节点-机器，都存在一个本地的因果图，
因果推断的流程 在前端的SLO异常时被触发，然后迭代往复地 沿着服务依赖图的路径，跑到后端服务，
如果在某个服务节点检测到SLO异常，那么就可以进行更精细的推断，--在指标依赖图上。
【图1-根因分析框架】展示了基本结构和workflow，


- 底层是一个业务系统的物理拓扑
- 顶部是这个业务系统对应的因果图
![图1-根因分析框架](causeinfer_framework.jpeg)
- 大虚线圆圈表示服务
- 红色节点表示根本原因，
- 黑色节点表示性能指标，
- 绿色节点表示 SLO 指标，
- 箭头表示方向 故障传播。
- 图中的有向边代表的意思是，因果关系

### 举个🌰演示工作流

假设服务II节点中的指标 E 是根因

- 当检测到服务 I 的 SLO 异常时，触发原因推断
- 对服务 I 节点进行根因分析，定位根因是指标A，A就是服务II的SLO指标
- 因此，服务II中的根因分析被触发，在服务II在的机器中，加载出相应的服务指标依赖图，服务I中的指标A异常可能是服务II的SLO异常导致。
- 因此会继续在服务II中触发因果推断，这里用到该机器的指标因果图。
- 总结：最后我们找到了根因--指标E， 推断路径为：SLO（service II ） → A（service II ） → D（service II） → E（service II）

![图2example](./eample1.png)

另外，需要注意的是，推断结果包含多个根因，因此需要一个排序的步骤选择最可能的根因， 来降低误告。

## 系统细节

CauseInfer包含两个步骤：离线和在线。

在线阶段包括两个模块，数据收集和因果推断。

- 数据收集模块，收集了运行的性能指标--多个数据源。
- 因果推断模块，负责因果图遍历和对根因进行排序。

离线阶段包括两个模块--变点检测和因果图构建

- 变点检测：将指标转化为0/1的二值序列，使用的是贝叶斯变点检测。
- 因果图构建：使用二值指标来构建一个2层的层次化的因果图。


### A：数据收集

数据收集模块，收集高维度运行信息，从多个数据源，横跨多个不同的软件栈，包括应用，进程 和 操作系统。

在因果图构建阶段，我们需要一个应用的SLO指标。然而，并不是所有的应用都显示地提供SLO指标（例如mysql，hadoop），
并且，不同应用中SLO对应不同指标。因此我们提出了一个新的统一的SLO指标--tcp请求延迟，简称为TCP LATENCY。
TCP LATENCY 衡量的是某个端口， 最后一个入站包（请求）和第一次出站包（响应）的时间间隔。
大部分应用都是用TCP作为传输协议，例如 Mysql, Httpd等，因此TCP延迟，可以作为大部分应用的SLO指标。

### B: 变点检测
根据Pearl的因果（cause-effect）[7]概念，如果有两个变量有因果关系，一个变量的变化将导致另一个变化。因此，在构建因果图之前，我们首先确定时间序列中的变化。一般采用CUSUM [4]来检测突变点，
但是由于对噪声的高敏感性，CUSUM很难检测出长期的变化，导致离线分析的高误警。因此我们引入了更有效的方法，贝叶斯变点检测（缩写为BCP）[8]

BCP的基本思想是找到一个underlying 参数序列，将时间序列划分为连续的blocks，每个blocks的参数一样
突变点的位置，就是每个block的边缘。
。给定一个观察序列：X =（x1，x2，···，xn），目的是
找到一个分区：ρ=（P1，P2，P3，···，Pn-1），
其中Pi = 1 表示在位置i + 1处发生了变化，否则Pi = 0。
关于BCP的详细理论分析参考论文[8]。与CUSUM方法相比，BCP不需要设置突变点个数，以及原始序列中group个数。

BCP效果更好，更适合分析长序列，但是不适合在线模式，
所以我们仍采用CUSUM作为在线的变点检测方法。

### C: 因果图构建

在本节中，我们将描述两层的分层因果图（即服务依赖图和 性能指标因果图）构建过程。
在collective变量中，如果Y的所有父母已经固定的，Y的分布将是固定的，并且不受其他变量影响。在这种因果关系中， 不允许两个变量互为因果。所以因果关系可以由DAG进行编码。

详细的服务依赖图 和 性能依赖图 的构建，参考[CauseInfer论文笔记2](https://chiechie.github.io/2021/03/03/technology/causeinfer-notes2/)


## 把离线和在线串起来

### 离线分析

在离线阶段，构建因果图（causality graph）:

- 构造服务依赖图分为两步：
  - 第一步通过采集器获取边是否存在；
  - 第二步通过分析两个服务间的通信延迟相关性（traffic lag correlation）来进一步确定边的方向。
- 服务依赖图中的节点怎么定义？二元组 (ip, service name)
  
  > 有的文章用3元组(ip, port, proto)表示，proto是传输协议的类型，比如TCP或者UDP。
  > 
  > 考虑到一个服务可能占用多个端口, 采用这种方式就会导致图的节点非常多，但是对根因定位其实没有很大的用处。
  > 
  > 举个例子，在一个三层的系统中，一个web server可以通过任意一个端口访问application server的。如果使用端口作为服务的唯一属性，这个服务依赖图就会变得非常大，即使所有请求都是由同一个服务发起的。

### 在线推断

在实时阶段，使用因果图（causality graph）来做根因推断，得到可能的原因列表。

![图1-根因分析框架](./img.png)

## 评估

我们解决的是一个因果推断的问题，因此常用的统计学的设计实验的思路在这里行不通。


## 参考文献

1. [2014-INFOCOM_CauseInfer](https://netman.aiops.org/~peidan/ANM2016/RootCauseAnalysis/ReadingLists/2014INFOCOM_CauseInfer.pdf)
2. [2007-The Journal of MachineLearning Research-pc算法](https://www.jmlr.org/papers/volume8/kalisch07a/kalisch07a.pdf)
3. [别人对CauseInfer论文的解读](https://saruagithub.github.io/2020/04/13/20200413CauseInfer%E8%AE%BA%E6%96%871/)
4. [CauseInfer论文笔记2](https://chiechie.github.io/2021/03/03/technology/causeinfer-notes2/)
5. J. Pearl, Causality: models, reasoning and inference. Cambridge Univ Press, 2000, vol. 29.
6. D. Barry, “A bayesian analysis for change point problems,” Journal of the American Statistical Association, vol. 88, no. 421, pp. 309–319, 1993.