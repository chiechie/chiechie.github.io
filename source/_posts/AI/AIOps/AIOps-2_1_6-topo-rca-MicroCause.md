---
title: AIOps-2_1_6-topo-rca-MicroCause.md
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
		- [chapter2.1.6 MicroCause](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-2_1_6-topo-rca-MicroCause)
	- [chapter2.2 多维下钻根因定位](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-2_2-multi-dimensional-rca/): 暂无
	- [chapter2.3 调用链数据的预处理](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-2_3-trace_rca/)
	- [chapter2.4 时间序列关联性分析](https://chiechie.github.io/2021/04/14/AI/AIOps/AIOps-2_4-metric_event_correlation/)


## 清华阿里AIOps新作：基于因果分析的微服务内根因定位



##**简 介**

本文介绍清华大学NetMan实验室与阿里巴巴合作的AIOps论文：基于因果分析的方法，对于微服务内故障进行根因定位。该论文发表于IWQOS2020，IEEE/ACM International Symposium on Quality of Service会议的论文***Localizing Failure Root Causes in a Microservice through Causality Inference***。

近些年来，由于微服务架构灵活性和逻辑清晰性，越来越多的互联网应用采用微服务架构。因此，微服务的稳定性对于这些应用程序的服务质量十分重要。准确定位故障根本原因可以帮助运营商快速恢复微服务故障并减轻损失。虽然在微服务架构中定位故障的根因微服务问题已得到很好的研究，如何在微服务内部的定位故障根本原因却尚未被研究。在这项工作中，我们提出了一个框架，**MicroCause**，可以准确地定位导致微服务故障根本原因的监视指标。MicroCause结合了简单而有效的路径条件时间序列（PCTS）算法以准确地捕获时间序列之间的因果关系，以及面向时间因果的新型随机游走方法（TCORW）。TCORW方法整合了因果关系，监控数据的异常和优先级信息。基于收集于顶级的全球在线购物平台的86个实际故障实例，我们对于MicroCause进行了评估。我们的实验表明MicroCause用于微服务内部故障给出的top 5的可能根因的准确性（AC @ 5）为98.7％，比最佳基准方法提高了33.4％。



##背景

近些年来，在需要支持多平台的互联网应用中，微服务架构越来越受欢迎。与此同时，对于互联网公司来说，微服务的性能质量至关重要，因为微服务故障会降低用户体验并带来经济损失。而有效的定位故障的根本原因有助于快速恢复服务并减轻损失。在微服务架构中，一个应用程序应用程序往往被分解为多个微服务。例如，图1展示了，在购物平台中，完成用户下单操作需要调用多个微服务。当前针对微服务系统的故障根因问题，学术界已有一些成熟的工作。这些工作主要是通过学习故障如何在微服务之间传播，并尝试根因的微服务（例如，图1中Address微服务是导致支付异常的根因）。但是，导致某一微服务故障的根本原因却仍旧无法得知。而运维人员是需要依赖于这个信息来采取措施减轻故障。



![image-20200622132926186](/Users/stellazhao/research_space/EasyMLBOOK/_image/image-20200622132926186.png)

图1：在线购物平台中，当接收到用户下单指令后，Order Creation微服务，将调用Inventory，Discount Coupon，Freight，Address这几个微服务来完成用户下单的需求。



为了找到微服务故障的根本原因，我们需要了解微服务的工作原理。这里我们以“Discount Coupon”这一在线购物平台中的微服务举例。如图2所示，对于一个微服务来说，通常有上游组件（例如，微服务，中间件）调用它，并且它可以调用某些下游组件（例如，微服务，中间件）。例如，用户下了一个订单后，”Order Creation“微服务将调用“Discount Coupon”微服务查看可用的优惠券。“Discount Coupon”微服务将调用“Inventory”微服务，针对不同的库存计算不同的优惠金额。此外，微服务通常部署在一个或多个容器或虚拟机。因此在本问题中主要有三种类型的组件，它们可能会导致微服务故障：上游组件，下游组件，部署环境。

![image-20200622132951991](/Users/stellazhao/research_space/EasyMLBOOK/_image/image-20200622132951991.png)

图2：““Discount Coupon”微服务实现细节



运维人员通常会配置一系列监控指标以持续监视每个微服务的性能（如图2所示）。这些指标包括用户可感知的指标KPI，例如用户响应时间(RT)和指标Metrics. Metrics包括与上游组件相关的指标（例如，Web用户的每秒查询（QPS）），下游组件相关指标（例如中间件的RT），与部署环境相关的指标（例如CPU）。当KPI异常时，通常表示微服务故障，而这往往是由基础组件的异常导致的。通常，异常组件或部署环境往往可以通过一个或多个metrics的异常反映出来。因此，微服务故障的根本原因可以用异常metrics表示。例如，如图2中，由Web RT异常标明的微服务故障是由Web QPS异常引起的。

因此在该问题中，我们将针对一个微服务故障实例，给出N个metrics，作为故障的Top N根因。通常一个故障实例由三部分信息组成：异常的KPI，异常的微服务ID和异常时间点。



## 已有方法

如表1中所示，<u>目前已有的解决方法主要是利用微服务之间的故障传播关系</u>（cross），通常分为两个阶段：

1. 构造依赖关系（relationship learning）：适用pc算法学习微服务之间依赖关系；或者 采用系统工具获取依赖关系

2. 推断根因（rcf）：通过随机游走等基于相关性的方法推断根本原因。

![image-20200622133032479](/Users/stellazhao/research_space/EasyMLBOOK/_image/image-20200622133032479.png)

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

表1：微服务系统中已有解决方法



![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

## 研究挑战



针对本问题的研究，主要有以下两个挑战：

###**挑战一：已有的pc算法是基于iid的假设，不能捕捉延迟的因果关系**

首先由于在本问题中我们的研究对象是监控指标，因此我们无法使用系统工具获取依赖关系图。以往的工作中，这种情况往往会采用PC算法学习因果关系图。而本身PC算法是针对*iid*数据进行设计的，因此以往的算法会将时间序列当做*iid*数据进行学习。例如图3所示的4个时间序列的因果关系中，以往的使用PC的根因定位算法会在开始的时候假设17:17分的数据和17:18分的数据相互独立。而通过图3我们可以看到，时间序列之间的因果关系往往会有时间差（如17:17分Web QPS会影响17:18的Web RT。如果在开始的时候就假设不同时刻相互独立，那样时间序列间具有延迟的因果关系就无法被学习。

![image-20200622133454753](/Users/stellazhao/research_space/EasyMLBOOK/_image/image-20200622133454753.png)

图3：四个时间序列的因果关系图

###**挑战二：基于相关性的随机游走无法准确定位故障根因。**

随机游走已被广泛用于微服务系统的根因定位。它的基本假设是越与异常KPI相关的metric越有可能是根因。然而，在我们的场景中，对于监控指标（KPI和metric）来说，同一类别指标比不同类别天然的相关更强。例如，图4中的异常KPI,Web RT与Middleware1 Consumer RT因为同为RT指标。Web RT和Middleware1 Consumer RT的表现为变成大峰值，因此更加相关。但是Web QPS的异常往往表现为电平转换。但是，这个实例中导致Web RT的根因是Web QPS。因此这种基于相关性的随机游走往往无法准确定位故障根因。

![image-20200622133729009](/Users/stellazhao/research_space/EasyMLBOOK/_image/image-20200622133729009.png)

图4：微服务中的4个监控指标：Web RT是KPI，Web QPS, Middleware1 Consumer RT和JVM FGC Time是metrics。

导致第一个指标异常的根因是第二个指标，但是但从指标相关性分析，第三个更像根因，所以带来误判



##架构设计-MicroCause

1. 使用pcts算法，从故障kpi和相关指标中，学习故障因果图。

2. 对相关指标进行异常检测，生成异常事件序列。
3. 1和2并行处理完之后，使用tco随机游走，推断可能的根因指标

为了解决上述问题和挑战，我们提出了MicroCause，这是一个针对微服务内故障利用因果关系进行根因定位的研究框架。在图5中，我们演示了MicroCause针对某一故障进行根因定位。当在KPI（例如Web RT）中检测到在线异常时，MicroCause将被激活。根据经验故障传播时间，故障微服务的故障前数小时的监控指标将用作MicroCause的输入。利用输入数据集，我们提出了PCTS算法，将用于生成该故障的故障因果图。与此同时，metrics指标将在异常检测模块中，检查输入数据集是否存在异常。故障因果图学习模块和异常检测模块可以进行并行处理。接下来我们设计了面向时间因果的随机游走（TCORW），其将基于故障因果图和metric故障诊断信息，给出N个潜在的根本原因。

![image-20200622134206479](/Users/stellazhao/research_space/EasyMLBOOK/_image/image-20200622134206479.png)

图5:MicroCause架构图



### 1.故障因果图学习

针对挑战一，我们设计了故障因果图学习算法PCTS，基于监控指标学习故障因果图。PCTS算法分为两步，第一步我们使用了改进的PC算法，该算法用于学习时间序列的因果图。该算法的结果如图3所示。图中的每一个节点代表了时间序列的某一个时间点，两个节点之间的箭头，标明了两个时间点的数据之间的因果关系。但是由于我们需要利用指标之间的因果关系进行根因定位，我们对于改进的PC算法的结果进行了调整。在PCTS中，我们假设，在改进的PC算法的结果中，如果时序A和时序B的时间点中存在一条边，那么在最后的故障因果图中时间序列A和时间序列B间就会存在一条边。因此图3的因果图最终会被转化为图6中的故障因果图。

![image-20200622135001778](/Users/stellazhao/research_space/EasyMLBOOK/_image/image-20200622135001778.png)

图6：四个指标间的故障因果图



### 2.异常检测

在MicroCause中，我们假设根因指标需要再KPI异常前存在异常，因此我们在异常检测模块中对于metric指标进行异常检测。这里我们采用了SPOT异常检测算法。SPOT算法利用极值理论来检测时间序列的异常波动。而在我们的场景中，metric的异常往往表现为突增或者是突降。因此SPOT算法可以有效的检测metric中的异常。另外，我们还对异常指标的异常程度进行了评估。异常程度  的计算方式如下所示:

$$\eta_{\max }^{i}=\max _{k \in O} \frac{| M_{k}^{i}-\phi_{M_{k}^{i}}|}{\phi_{M^{i}}}$$



其中是时间序列 在时刻的值，是时间序列 在时刻SPOT拟合的阈值。

###3. 面向时间因果的随机游走(TCORW)

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

在这一步中，我们首先利用指标间可能的故障传播关系，将指标分为三个级别，如表2所示。当Level1和Level2中的指标同时发生异常的时候，我们认为Level1中的指标更有可能是根因。

![image-20200622135235013](/Users/stellazhao/research_space/EasyMLBOOK/_image/image-20200622135235013.png)

表2：指标分级



基于指标分级信息，指标的异常时间和指标的潜在异常得分我们设计了如下算法给出最后的根因排序。

![image-20200622135254320](/Users/stellazhao/research_space/EasyMLBOOK/_image/image-20200622135254320.png)

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

## 实验评估



#### 数据集和评估指标

在此工作中，从2019年9月份到2020年1月份，我们一直某大型在线购物平台中监控超过400种微服务状态。我们收集了86**个真实的在线故障实例作为评估数据集。根据过去的根因分析工作，我们使用AC@k和avg@k评估算法给出的Top K个根因的准确性。

$$ AC@ k=\frac{1}{|A|} \sum_{a \in A}\frac{\sum_{i< k} R^{a}[i] \in V_{r c}^{a}}{\min (k,|\mathrm{V^a}|)} $$

$$A v g @ k=\frac{1}{k} \sum_{1 \leq j \leq k} A C @ j$$

#### 端对端性能评估

在我们的工作中，我们将MicroCause与四种微观系统根因定位方法和“异常时间顺序”方法进行了比较。我们可以看到MicroCause，在所有方法中具有最佳性能。尤其是，MicroCause的Top 5位准确性达到了98.7％，高于最佳性能基准方法33.4％。

![image-20200622135756159](/Users/stellazhao/research_space/EasyMLBOOK/_image/image-20200622135756159.png)

表3：MicroCause与baseline算法性能对比



####MicroCause性能分析

此外我们评估了MicroCause的故障因果图学习部分和面向时间因果的随机游走部分的性能。

从表4结果可以看出，PCTS的AC @ 5比PC高5.1％。

![image-20200622135932892](/Users/stellazhao/Library/Application Support/typora-user-images/image-20200622135932892.png)

表4：PCTS性能评估



从表5结果可以看出，与传统的一阶随机游动和二阶随机游动相比，MicroCause可以实现此任务的最佳性能。

![image-20200622135948373](/Users/stellazhao/Library/Application Support/typora-user-images/image-20200622135948373.png)

表5：TCORW性能评估




##案例分析

在CauseInfer[INFOCOM14]中，作者提出，当时使用PC算法来学习基于时间序列的因果图时容易生成的孤立子图。在那篇论文中，作者使用领域知识来弥补这一缺陷。在我们的工作中，我们同样观察到了这种现象。从图7中，由PC生成的因果图可以看出，异常KPI与根因之间没有路径。但是这个问题可以通过PCTS成功解决，因为它可以捕获时间序列之间的延迟因果关系（图8）。因此，我们相信PCTS可以用于其他与时间序列相关的因果学习问题。

![image-20200622140008064](/Users/stellazhao/research_space/EasyMLBOOK/_image/image-20200622140008064.png)



图7：针对故障实例A,基于PC算法生成的因果图

![image-20200622140031849](/Users/stellazhao/research_space/EasyMLBOOK/_image/image-20200622140031849.png)

图8：针对故障实例A,基于PCTS算法生成的因果图




##总结



当前，在当前的大型Internet系统中，基于微服务架构的应用变得越来越普遍。快速故障根因定位可以提高服务质量并减少收入损失。在本文中，我们首次提出在微服务内部研究故障根本原因。这也是整个系统故障定位的重要一步。我们设计了一个框架MicroCause，其在基于86个在线故障实例的实验中，以最高性能实现了微服务内部故障根因定位。在MicroCause，我们设计了PCTS，可以学习监控指标间的因果关系图。我们相信这种方法可以用于其他与时间序列相关的根因定位问题。