---
title: chapter2.1.5 基于拓扑的根因定位-MicroCause
author: chiechie
mathjax: true
date: 2021-05-21 22:43:57
tags:
- AIOps
- 根因分析
- 微服务
- 图数据
categories: 
- AIOps
---


- [chapter0 概览](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-0-summary/)
- [chapter1 故障发现](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-1-event-generate.md/)
	- [chapter1.1 单指标异常检测](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-1_1-kpi-detector/)
	- [chapter1.2 故障预测](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-1_2-fault-prediction/)
	- [chapter1.4 指标异常关联](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-1_4-kpi-correlation.md)
	- [chapter1.5 日志聚类](https://chiechie.github.io/2021/05/06/AI/AIOps/AIOps-1_5-log-analysis/)
		- [chapter1.5.1 使用logmine加强版做日志聚类](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-1_5_1-log-analysis_logmine/)
		- [chapter1.5.2 美团日志聚类](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-1_5_2-log-analysis_meituan/)

- [chapter2 故障定位](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-2-event-analysis/)
	- [chapter2.1 基于拓扑数据的根因定位](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-2_1-topo-rca/)
		- [chapter2.1.1 CauseInfer1](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-2_1_1-topo-rca-causeinfer-notes1/)
		- [chapter2.1.2 CauseInfer2](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-2_1_2-topo-rca-causeinfer-notes2/)
		- [chapter2.1.3 AIOps挑战赛2020-获奖方案分享](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-2_1_3-topo-rca-aiops2020)
		- [chapter2.1.4 AIOps挑战赛2021-demo方案](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-2_1_4-topo-rca-aiops2021/)
		- [chapter2.1.5 N-Softbei2020比赛](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-2_1_5-topo-rca-cnsoftbei2020)
		- [chapter2.1.6 MicroCause](https://chiechie.github.io/2021/05/21/AI/AIOps/.)
	- [chapter2.2 多维下钻根因定位](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-2_2-multi-dimensional-rca/): 暂无
	- [chapter2.3 调用链数据的预处理](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-2_3-trace_rca/)
	- [chapter2.4 时间序列关联性分析](https://chiechie.github.io/2021/04/14/AI/AIOps/AIOps-2_4-metric_event_correlation/)


> 清华阿里合作paper《基于因果分析的微服务内根因定位》, Localizing Failure Root Causes in a Microservice through Causality Inferenc
> 
> **MicroCause**，可以定位到导致微服务故障的根因指标，原理是，先使用路径条件时间序列算法（PCTS）挖掘时间序列之间的因果关系，然后使用随机游走方法（TCORW）来推断因果。TCORW方法综合分析了因果关系，指标异常程度和优先级信息。 
> 
> 在86个实际故障实例中，MicroCause给出的top 5根因的准确性（AC @ 5）为98.7％。


## 背景介绍

在微服务架构中，一个「应用程序」往往被分解为多个「微服务」。例如，图1展示了，在淘宝购物时，完成用户下单操作需要调用多个微服务。当前针对微服务系统的故障根因问题，学术界已有一些成熟的工作。这些工作主要是通过学习故障如何在微服务之间传播，并找到根因的微服务，例如，图1中Address微服务是导致支付异常的根因。但是，更进一步的根因信息尚不得而知，也就是说是什么导致这个微服务故障？运维人员还是没法明确知道下一步要怎么响应故障。


![](./image-20200622132926186.png)

图1：在线购物平台中，当接收到用户下单指令后，Order Creation微服务，将调用Inventory，Discount Coupon，Freight，Address这几个微服务来完成用户下单的需求。

怎么进一步定位微服务故障根因？ 上游组件，下游组件，部署环境。

对于一个微服务来说，通常有上游组件（例如，微服务，中间件）调用它，并且它可以调用某些下游组件（例如，微服务，中间件）。如图2所示，用户在淘宝下了一个订单后，”Order Creation“微服务将调用“Discount Coupon”微服务查看可用的优惠券，“Discount Coupon”微服务将调用“Inventory”微服务，针对不同的库存计算不同的优惠金额。此外，微服务通常部署在一个或多个容器或虚拟机。


![](./image-20200622132951991.png)

图2：““Discount Coupon”微服务实现细节

对每个微服务需要配置监控指标（如图2所示），包括用户可感知的KPI，上游组件相关的指标，下游组件相关的指标，与部署环境相关的指标（例如CPU）。当KPI异常时，通常表示微服务故障，

运维人员通常会配置一系列监控指标以持续监视每个微服务的性能（如图2所示）。这些指标包括用户可感知的指标KPI，例如用户响应时间(RT)和指标Metrics. Metrics包括与上游组件相关的指标（例如，Web用户的每秒查询（QPS）），下游组件相关指标（例如中间件的RT），与部署环境相关的指标（例如CPU）。

当KPI异常时表示微服务故障，到底是组件异常还是部署环境异常，可通过观察指标是否异常得到。

因此，微服务故障的根本原因可以用异常metrics表示。例如，如图2中，由Web RT异常标明的微服务故障是由Web QPS异常引起的。

因此在该问题中，我们将针对一个微服务故障实例，给出N个metrics，作为故障的Top N根因。通常一个故障实例由三部分信息组成：异常的KPI，异常的微服务ID和异常时间点。



## 已有方法

如表1中所示，目前已有的解决方法主要是利用微服务之间的故障传播关系（cross），通常分为两个阶段：

1. 构造依赖关系（relationship learning）：适用pc算法学习微服务之间依赖关系，或者采用系统工具获取依赖关系
2. 推断根因（rcf）：通过随机游走等基于相关性的方法推断根本原因。

![image-20200622133032479](./image-20200622133032479.png)

表1：微服务系统中已有解决方法


## 研究挑战
针对本问题的研究，主要有以下两个挑战：

### 挑战一：已有的pc算法是基于iid的假设，不能捕捉延迟的因果关系

已有的方法，要解决的问题比较简单，即只用定位微服务间的因果关系，相应的解决方案要么通过系统工具获取依赖关系，要么通过pc算法学习因果关系。

但是现在，要解决的为问题更有挑战性，除了微服务之间的因果，还想知道一个微服务的指标之间的因果。
所以，提出了一种PC加强版因果挖掘的方法--PCTS。

例如图3所示的4个时间序列的因果关系中，以往的使用PC的根因定位算法会在开始的时候假设17:17分的数据和17:18分的数据相互独立。而通过图3我们可以看到，时间序列之间的因果关系往往会有时间差（如17:17分Web QPS会影响17:18的Web RT。如果在开始的时候就假设不同时刻相互独立，那样时间序列间具有延迟的因果关系就无法被学习。

![img.png](./img.png)

图3：四个时间序列的因果关系图

### **挑战二：基于相关性的随机游走无法准确定位故障根因。**

随机游走已被广泛用于微服务系统的根因定位。它的基本假设是越与异常KPI相关的metric越有可能是根因。然而，在我们的场景中，对于监控指标（KPI和metric）来说，同一类别指标比不同类别天然的相关更强。例如，图4中的异常KPI,Web RT与Middleware1 Consumer RT因为同为RT指标。Web RT和Middleware1 Consumer RT的表现为变成大峰值，因此更加相关。但是Web QPS的异常往往表现为电平转换。但是，这个实例中导致Web RT的根因是Web QPS。因此这种基于相关性的随机游走往往无法准确定位故障根因。


![img_1.png](./img_1.png)

图4：微服务中的4个监控指标：Web RT是KPI，Web QPS, Middleware1 Consumer RT和JVM FGC Time是metrics。

导致第一个指标异常的根因是第二个指标，但是但从指标相关性分析，第三个更像根因，所以带来误判



## 架构设计

1. 使用pcts算法，从故障kpi和相关指标中，学习故障因果图。
2. 对相关指标进行异常检测，生成异常事件序列。
3. 1和2并行处理完之后，使用tco随机游走，推断可能的根因指标

为了解决上述问题和挑战，我们提出了MicroCause，这是一个针对微服务内故障利用因果关系进行根因定位的研究框架。在图5中，我们演示了MicroCause针对某一故障进行根因定位。当在KPI（例如Web RT）中检测到在线异常时，MicroCause将被激活。根据经验故障传播时间，故障微服务的故障前数小时的监控指标将用作MicroCause的输入。利用输入数据集，我们提出了PCTS算法，将用于生成该故障的故障因果图。与此同时，metrics指标将在异常检测模块中，检查输入数据集是否存在异常。故障因果图学习模块和异常检测模块可以进行并行处理。接下来我们设计了面向时间因果的随机游走（TCORW），其将基于故障因果图和metric故障诊断信息，给出N个潜在的根本原因。

![image-20200622134206479](./image-20200622134206479.png)

图5:MicroCause架构图


### 1.故障因果图学习

针对挑战一，我们设计了故障因果图学习算法PCTS，基于监控指标学习故障因果图。PCTS算法分为两步，第一步我们使用了改进的PC算法，该算法用于学习时间序列的因果图。该算法的结果如图3所示。图中的每一个节点代表了时间序列的某一个时间点，两个节点之间的箭头，标明了两个时间点的数据之间的因果关系。但是由于我们需要利用指标之间的因果关系进行根因定位，我们对于改进的PC算法的结果进行了调整。在PCTS中，我们假设，在改进的PC算法的结果中，如果时序A和时序B的时间点中存在一条边，那么在最后的故障因果图中时间序列A和时间序列B间就会存在一条边。因此图3的因果图最终会被转化为图6中的故障因果图。

![image-20200622135001778](./image-20200622135001778.png)

图6：四个指标间的故障因果图

### 2.异常检测

在MicroCause中，我们假设根因指标需要再KPI异常前存在异常，因此我们在异常检测模块中对于metric指标进行异常检测。这里我们采用了SPOT异常检测算法。SPOT算法利用极值理论来检测时间序列的异常波动。而在我们的场景中，metric的异常往往表现为突增或者是突降。因此SPOT算法可以有效的检测metric中的异常。另外，我们还对异常指标的异常程度进行了评估。异常程度  的计算方式如下所示:

$$\eta_{\max }^{i}=\max _{k \in O} \frac{| M_{k}^{i}-\phi_{M_{k}^{i}}|}{\phi_{M^{i}}}$$



其中是时间序列 在时刻的值，是时间序列 在时刻SPOT拟合的阈值。

### 3. 面向时间因果的随机游走(TCORW)

总结一下
先构建一个指标 = 随机游走的根因概率 + 指标的异常程度
相关指标分为3层，每一层找出top2的两个指标，一起得到6个
按照时间排序，得到前3个或者5个输出


在上述的两个模块中我们得到了故障因果图和metric指标的异常信息，基于这两部分信息，我们设计了面向时间因果的随机游走TCORW算法得到潜在的N个根因。TCORW算法主要分为三步：

- 面向因果的随机游走，

- 潜在根因得分

- 根因排序。

下面我们对这三步进行详细的介绍：

#### **1. 面向因果的随机游走**

这一步我们主要利用监控指标之间的因果关系进行分析。首先我们利用模块一中生成的故障因果图进行随机游走。不同于传统的随机游走算法，在此算法中，我们利用偏相关系数（Partial Correlation)来计算转移概率矩阵。与Pearson相关性不同的是，和异常KPI因果性更强的metric指标将具有更高的偏相关系数，而Pearson相关性更注重两个指标之间的相关性。因此和异常KPI因果性更强的metric将在随机游走中获得更高的访问次数。

#### **2. 潜在根因得分**

这一步中我们利用，随机游走的结果和指标的异常程度对于异常的metric计算潜在根因得分，计算方式如下所示：

$$\gamma_{i}=\lambda \bar{c}_{i}+(1-\lambda) \bar{\eta}_{\max }^{i}$$

其中是归一化后的随机游走访问次数，是归一化后的异常程度，作为参数，将用于控制着两部分的贡献比例。

#### **3. 根因排序**

在这一步中，我们首先利用指标间可能的故障传播关系，将指标分为三个级别Level1，Level2，Level3。

如表2所示，当Level1和Level2中的指标同时发生异常时，我们认为Level1中的指标更有可能是根因。

![表2：指标分级](./image-20200622135235013.png)


基于指标分级信息，指标的异常时间和指标的潜在异常得分我们设计了如下算法给出最后的根因排序。
![](./image-20200622135254320.png)


## 数据集和评估指标

在此工作中，从2019年9月份到2020年1月份，我们一直某大型在线购物平台中监控超过400种微服务状态。我们收集了86**个真实的在线故障实例作为评估数据集。根据过去的根因分析工作，我们使用AC@k和avg@k评估算法给出的Top K个根因的准确性。

$$ AC@ k=\frac{1}{|A|} \sum_{a \in A}\frac{\sum_{i< k} R^{a}[i] \in V_{r c}^{a}}{\min (k,|\mathrm{V^a}|)} $$

$$A v g @ k=\frac{1}{k} \sum_{1 \leq j \leq k} A C @ j$$


## 案例分析

在CauseInfer[INFOCOM14]中，作者提出，当时使用PC算法来学习基于时间序列的因果图时容易生成的孤立子图。在那篇论文中，作者使用领域知识来弥补这一缺陷。 在我们的工作中，同样观察到了这种现象。从图7中，由PC生成的因果图可以看出，异常KPI与根因之间没有路径。

![](./image-20200622140008064.png)
图7：针对故障实例A,基于PC算法生成的因果图

但是PCTS可解决这个问题，因为它可以表达时序之间的延迟因果关系（图8）。

![](./image-20200622140031849.png)
图8：针对故障实例A,基于PCTS算法生成的因果图

## 参考
1. [清华阿里AIOps新作：基于因果分析的微服务内根因定位](https://wemp.app/posts/6013f2da-c11a-4f6f-b393-2b631c45172a)