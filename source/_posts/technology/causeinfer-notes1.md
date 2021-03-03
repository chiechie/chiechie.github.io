---
title: CauseInfer论文笔记1-白话版
author: chiechie
date: 2021-03-02 14:41:39
mathjax: true
categories: 技术类

tags:
- AIOps
- 根因分析
- 论文笔记
---

> CauseInfer是一个做故障根因定位的通用方法，通过在历史数据学习因果图，进而在发生故障时，能推断出故障根因。
>
> 这篇文章对小赵来说太难啃了，主要是因为涉及大量业务领域知识，我对这块不熟悉，需要补课啊。
>
> 分成两块吧: 第一篇-CauseInfer论文笔记1，即本文，讲背景知识和概要设计，以及一些直觉型的描述；第二篇-CauseInfer论文笔记2，讲核心技术点，这一篇会比较硬核，涉及到数学公式。 尽量能让第一篇说清楚，让不懂技术的人也能明白这个技术的原理。第二篇就是给算法人员看的，要出方案了，所以要越详细越好。


# 名词解释

来，补充业务知识

- QPS：Queries Per Second，每秒查询次数
- SLA （service level agreement）:服务等级协议。指的是整个协议，协议的内容包含了SLI，SLO以及恢复的方式和时间等等一系列所构成的协议。
- SLI（service level indicator）:服务等级对象。指的是对象，例如：qps（，响应时间，准确性等
- SLO（service level objective）:服务等级目标。指的是目标，例如：qps 99.99% ，响应时间10ms等


# 几个问题

看完第一遍之后，还有几个问题

- 一个主机上的服务，构建一个因果图？
    对的，每个主机上的每个服务都有一个自己的指标因果图。
    然后跨主机的服务调用，就靠服务依赖图表示。
- 因果图中不同的机器IP是否是不同的节点？
  是的，服务依赖图的粒度更细，是一个机器上的一个服务是一个节点。
  
- 因果图是怎么构建？
  两种方式都有：调用链路构建拓扑 + 流量延迟效应识别故障传播方向。
  
- 因果图构建好了，如果做因果推断？
  深度优先遍历，找有故障的父母亲，直到找到根结点，或者正常的上游。

- 什么是指标因果图？
  因果图的细粒度视图，

- 指标因果图和服务依赖图是什么关系？
  因果图 = 指标因果图（刻画局部的依赖关系） + 服务依赖图（刻画整个信息设备的依赖关系）
  指标因果图表达的是一台机器上，一个服务的指标和多个机器之间的故障传播关系。
  服务依赖图表达的是不同机器的不同服务之间的故障传播关系。


# 根因定位方案的框架

整个故障定位框架是这样：

- 在离线阶段，构建因果图（causality graph）。
- 在实时阶段，使用因果图（causality graph）来做根因推断，得到可能的原因列表。

![图1-根因分析框架](img.png)


# 核心技术点

构建因果图是本文的核心技术点。
因果图是一个两层分层的图。（two layered hierarchical causality graph）
较高的层是粗粒度的信息，表示每台机器上每个服务间的依赖关系，也叫服务依赖图（Service Dependency Graph）。
较低的层是细粒度的信息，表示系统指标组成的细粒度因果关系，也叫指标因果图（Metric Causality Graph）

图中的有向边代表的意思是，根源指标（cause metric ）异常 **导致** 表象指标异常（effect metric），

介绍了这两个因果图，再来看 CauseInfer 的基本结构和工作流程
![图1-根因分析框架](causeinfer_framework.jpeg)

- 最底层是三层系统的物理拓扑
- 顶部是抽象的服务和指标因果关系图 
- 因果关系图中
  - 大虚线圆圈表示服务
  - 红色节点表示根本原因，
  - 黑色节点表示性能指标，
  - 绿色节点表示 SLO 指标，
  - 弧线表示因果关系
  - 箭头表示方向 故障传播。

举例：
假设服务 II 节点中的指标 E 是根因
- 当检测到服务 I 的 SLO 异常时，触发原因推断，
- 对服务 I 节点进行根因分析，定位到指标A发生了性能异常，
- 进一步使用服务调用图，可以推断出，服务I中的指标A异常可能是服务II的SLO异常导致。
- 因此会继续在服务II中触发因果推断，这里用到存储在服务II节点中的指标因果图，
- 最后我们找到了根因，指标E， 推断路径为：SLO（service II ） → A（service II ） → D（service II） → E（service II）

构造服务依赖图分为两步：第一步通过采集器获取边是否存在；第二步通过分析两个服务间的通信延迟相关性（traffic lag correlation）来进一步确定边的方向。
构造指标因果图分

服务依赖图就是服务间的调用关系，属于工程型工作，一般通过采集器获取。
指标因果图：在服务依赖图的基础上，做一些分析性的工作。


详情请看下一章



# 参考文献

1. [2014-INFOCOM_CauseInfer](https://netman.aiops.org/~peidan/ANM2016/RootCauseAnalysis/ReadingLists/2014INFOCOM_CauseInfer.pdf)
2. [2007-The Journal of MachineLearning Research-pc算法](Mhttps://www.jmlr.org/papers/volume8/kalisch07a/kalisch07a.pdf)
3. [别人对CauseInfer论文的解读](https://saruagithub.github.io/2020/04/13/20200413CauseInfer%E8%AE%BA%E6%96%871/)