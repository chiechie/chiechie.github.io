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

> CauseInfer是一个解决故障定位的通用方案，原理是通过在历史数据中学习因果图，进而在发生故障时，推断出故障根因。
>
> 这篇文章对小赵来说太难啃了，主要是因为涉及大量专业领域知识，比如计算机网络，微服务架构等，我对这块不熟悉，需要补很多功课。
>
> 输出的文章分成两篇: CauseInfer论文笔记1，即本文，讲背景知识和概要设计，以及直觉型的描述问题和解题思路，都是粗粒度的；[CauseInfer论文笔记2](https://chiechie.github.io/2021/03/03/technology/causeinfer-notes2/)，讲核心技术点，方案的实现逻辑，涉及到数学公式。
> 
> 原则上，第一篇不用技术背景也能看明白，问题是啥，解题思路是啥。第二篇就是给算法人员看的，要出方案了，所以要越详细越好。


# 名词解释

来，补充业务知识

- QPS：Queries Per Second，每秒查询次数
- SLA （service level agreement）:服务等级协议。指的是整个协议，协议的内容包含了SLI，SLO以及恢复的方式和时间等等一系列所构成的协议。
- SLI（service level indicator）:服务等级对象。指的是对象，例如：qps（，响应时间，准确性等
- SLO（service level objective）:服务等级目标。指的是目标，例如：qps 99.99% ，响应时间10ms等


# 几个问题

看完第一遍之后，还有几个问题

- 因果图是什么？
  
  因果图是一个两层分层的图。（two layered hierarchical causality graph）
  较高的层是粗粒度的信息，表示每台机器上每个服务间的依赖关系，也叫服务依赖图（Service Dependency Graph）。
  较低的层是细粒度的信息，表示系统指标组成的细粒度因果关系，也叫指标因果图（Metric Causality Graph）

- 指标因果图和服务依赖图是什么关系？
  
  指标因果图 和 服务依赖图可以看成是因果图的两个视图。
  
  因果图 = 指标因果图（刻画局部的依赖关系） + 服务依赖图（刻画整个信息设备的依赖关系）。
  
  指标因果图是细粒度的视图，表达的是一台机器上，一个服务的指标和多个机器之间的故障传播关系。
  
  服务依赖图是粗粒度的视图，表达的是不同机器的不同服务之间的故障传播关系。
  
- 因果图是怎么构建？
  
  两种方式结合：采集调用链构建拓扑 + 数据分析得到因果关系
  
- 因果图构建好了，如果做因果推断？
  
  深度优先遍历，找有故障的父母亲，直到找到根结点，或者正常的上游。
  
- 一个主机上的服务，构建一个因果图？
  
  对的，每个主机上的每个服务都有一个自己的指标因果图。 然后跨主机的服务调用，就靠服务依赖图表示。
  
- 因果图中不同的机器IP是否是不同的节点？
  
  是的，服务依赖图的粒度更细，是一个机器上的一个服务是一个节点。


#  CauseInfer工作流

看图说话--causeinfer的工作流图

- 底层是一个业务系统的物理拓扑
- 顶部是这个业务系统对应的因果图
![图1-根因分析框架](causeinfer_framework.jpeg)

- 因果图中
  - 大虚线圆圈表示服务
  - 红色节点表示根本原因，
  - 黑色节点表示性能指标，
  - 绿色节点表示 SLO 指标，
  - 箭头表示方向 故障传播。
  - 图中的有向边代表的意思是，根源指标（cause metric ）异常 **导致** 表象指标异常（effect metric），

举例：
假设服务 II 节点中的指标 E 是根因

- 当检测到服务 I 的 SLO 异常时，触发原因推断，
- 对服务 I 节点进行根因分析，定位到指标A发生了性能异常，
- 进一步使用服务调用图，可以推断出，服务I中的指标A异常可能是服务II的SLO异常导致。
- 因此会继续在服务II中触发因果推断，这里用到存储在服务II节点中的指标因果图，
- 最后我们找到了根因，指标E， 推断路径为：SLO（service II ） → A（service II ） → D（service II） → E（service II）


# 根因定位方案的框架

整个故障定位框架是这样：

## 离线分析
在离线阶段，构建因果图（causality graph），这个是本文的核心技术点。

- 因果图是一个两层分层的图。（two layered hierarchical causality graph）
- 较高的层是粗粒度的信息，表示每台机器上每个服务间的依赖关系，也叫服务依赖图（Service Dependency Graph）。
- 较低的层是细粒度的信息，表示系统指标组成的细粒度因果关系，也叫指标因果图（Metric Causality Graph）
- 构造服务依赖图分为两步：第一步通过采集器获取边是否存在；第二步通过分析两个服务间的通信延迟相关性（traffic lag correlation）来进一步确定边的方向。
- 服务依赖图中的实体是怎么定义的呢？二元组 (ip, service name)，有的文章用3元组（three-tuple）表示--(ip, port, proto)，proto是传输协议的类型，比如TCP或者UDP，因为考虑到一个服务可能占用多个端口。举一个例子，在一个三层的系统中，一个web server可以通过任意一个端口访问application server的。如果，使用端口作为唯一服务的属性，这个服务依赖图，就会变得非常之大，即使所有请求都是由同一个服务发起的。

## 在线推断
在实时阶段，使用因果图（causality graph）来做根因推断，得到可能的原因列表。

![图1-根因分析框架](img.png)


详情请看下一章


# 参考文献

1. [2014-INFOCOM_CauseInfer](https://netman.aiops.org/~peidan/ANM2016/RootCauseAnalysis/ReadingLists/2014INFOCOM_CauseInfer.pdf)
2. [2007-The Journal of MachineLearning Research-pc算法](Mhttps://www.jmlr.org/papers/volume8/kalisch07a/kalisch07a.pdf)
3. [别人对CauseInfer论文的解读](https://saruagithub.github.io/2020/04/13/20200413CauseInfer%E8%AE%BA%E6%96%871/)