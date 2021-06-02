---
title: chapter2.2 多维下钻根因定位 
author: chiechie
mathjax: true
date: 2021-05-21 16:05:42
tags:
- AIOps
- 根因分析
categories: 
- AIOps
---

# 目录


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


# 问题描述

在运维场景中对监控指标的多个维度进行根因定位。具体来说，当某个总指标（如总流量）发生异常时，需要快速准确地定位到是哪个交叉维度的细粒度指标（如“省份=北京 & 运营商=联通”的流量）的异常导致的，以便尽快做进一步的修复止损操作。图示如下：


## 输入输出


输出：

| timestamp  |  set              |
|------------|-------------------|
| 1501475700 | a1&b2;a3&b4       |
| 1501475760 | a1&b2&x3;a4&b5&x6 |




# 方案介绍

## 几个问题

1. 要训练集干嘛？
   
    训练集用来当做历史数据，学习正常波动区间，对测试数据进行异常检测。

2. 为什么要注入异常？
   
    真实中故障很少发生，并且故障根因大部分都比较简单，不会出现多层，多个elemnts是根因的情况, 需要人工构造。


## Adtributor

Adtributor: Revenue Debugging in Advertising Systems

简介：Adtributor假设所有根因都是一维的，提出了解释力（Explanatory power）和惊奇性（Surprise）来量化根因的定义。使用ARMA模型对KPI进行预测，通过计算维度的惊奇性（维度内所有元素惊奇性之和）对维度进行排序，确定根因所在的维度（例如省份）。在维度内部计算每个元素的解释力，当元素的解释力之和（例如北京+上海）超过阈值时，这些元素就被认为是根因。

![img_1.png](img_1.png)

上述是针对基本类型的KPI的计算公式（例如PV、交易量），对于派生类型的KPI（多个基本类型KPI计算得到，例如成功率）就不太适用了。当然，作者也提出了另一套公式来计算派生型KPI，主要是让“解释力”和“惊奇性”的分数符合我们对于异常的“直觉”。

评价：Adtributor提出的“解释力”和“惊奇性”对根因给出了量化的评价，有很强的借鉴意义。将根因限定在一维的假设也极大的简化了问题的复杂度。但是这样的假设并不太符合我们的实际场景，同时用解释性和惊奇性的大小来衡量根因也不完全合理。因为其没有考虑到维度之间的相互影响以及「外部根因」的可能，即根因所在的维度没有在数据中体现。另外，Adtributor的根因分析严重依赖于整体KPI的变化情况，对于整体变化不大，但是内部波动较为剧烈的数据表现不好。

总的来说，Adtributor是一种可以落地的业务指标明细多维分析算法，能够提供一些对排障有帮助的信息，配合人工筛查，可以有一定的效果。但是如果应对更多的维度、更复杂的情况，甚至存在外部根因的时候，能够提供的帮助就有限了。

## iDice

iDice: Problem Identification for Emerging Issues

简介：iDice针对的场景和Adtributor稍有不同，它分析的数据是一段时间序列下的多维明细数据而不是对某一个具体的时间点。首先它认为根因的表现是一定在时间序列上体现突变，然后根据信息熵的思想提出Isolation Power的概念来解释根因，基本的思想就是对每一个维度组合计算他们的Isolation Power值，一个有效的根因维度组合（图1 Effective Combination）是其IP值大于其所有的直接子节点和直接父节点。在判断了当前节点为「可能的根因」后，其所有子节点都被剪枝掉。为了降低搜索和聚合的复杂度，本文还提出了多种剪枝方法。如减掉数量较小的节点、减掉在时间序列上没有突变的的节点等。最终对每个「可能的根因」突变点前后计算类似Adtributor的“惊奇性”进行排序。

评价：iDice提出了一种更符合实际情况的根因评估指标：Isolation Power，利用多种剪枝手段减少搜索的复杂度。利用了数据的时间序列，从一定程度上缓解了单一时间点做预测时不准的情况。但是对大量的节点做异常检测也带来了额外的复杂度，在维度和取值较多时，上层节点的数目要远远大于叶子结点的数目。从上至下的搜索以及至下而上的聚合的复杂度极大，虽然本文提出了几种剪枝方法，但是并不能带来量级上的减少。

同时，iDice的几种剪枝略显暴力，例如基于Impact的剪枝，虽然减掉的是量较少的维度组合，但是当维度和取值增加，极限情况下每一个叶子结点的维度组合的量只有1，这样剪枝的结果是不可以接受的。另外iDice对于派生类型的指标没有很好的处理方法，例如成功率，首先剪枝不能很好地进行，其次做变点检测时，容易忽略掉真正的“根因”。

## HotSpot

3. HotSpot: Anomaly Localization for Additive KPIs with Multi-Dimensional Attributes

简介：HotSpot和Adtributor一样使用“预测+搜索”的策略，针对基本类型的指标（可加型，例如交易量，PV等）提出了基于Ripple Effect的根因判断方法，即维度组合与其子节点之间越满足Ripple Effect条件就越有可能是根因，提出了Potential Score来量化一个节点与其所有叶子结点之间满足Ripple Effect程度。举个简单的例子，例如表一中的北京。PV从30->21,而北京联通和北京电信分别从20->14、10->7，变化的比例和北京相同（Ripple Effect），所以判断出北京是根因。和iDice对于根因集的定义不同，HotSpot会考虑同一维度下的根因组合，例如不仅有（北京，联通）这样的元素，还会有（北京，上海）这样的元素。这样当然会带来额外的搜索复杂度（2n-1，n是维度取值数目之和），所以作者提出基于MCTS（蒙特卡洛树搜索）和分层的剪枝策略，按照获得最大Potential Score收益的路径进行搜索，如果某一元素的PS分数较低，则减掉其子节点（Hierarchical Pruning）。

评价：HotSpot提出具有启发意义的根因判断方式：Ripple Effect。使用叶子结点而不使用直接子节点参与Potential Score的计算，达到了边搜索边聚合的效果，极大的降低了空间的使用。创新的将MCTS应用到搜索的剪枝中，降低了搜索的复杂度。当然，HotSpot也存在一些局限，例如假设所有的根因在一个Cuboid内（完全相同的维度组成：eg.C省份、C省份、运营商）、没有考虑到非可加型指标（eg.成功率）、时间序列预测方法也只是使用了简单的MA，如果指标有一定的波动，可能会影响PS的准确性。另外使用叶子结点计算PS虽然减少了空间的使用，但是随着维度和取值的增加，叶子结点的量下降，极端情况下下降到1，此时的PS计算会受到较大的影响。
 


## Squeeze

 Squeeze：Generic and Robust Localization of Multi-Dimensional Root Causes

简介：Squeeze在HotSpot的基础上提出广义的Ripple Effect,证明RE不仅仅适用于基本类型的指标，也适用于派生类型的指标（eg.成功率）同时解决了0值预测的问题。和之前的算法搜索策略不同，Squeeze先至下而上剪掉正常的叶子结点的减小搜索空间，然后计算叶子结点的deviation score来进行聚类，因为根据RE，相同的根因的叶子结点将具有相似的deviation score。然后自顶向下在每一个簇中搜索根因。Squeeze假设每一个簇中的根因都在一个Cuboid（见HotSpot简介）内，同样提出类似于Potential Score的GPS来作为是否是根因的量化标准。

评价：Squeeze在HotSpot的基础上进行了理论的拓展，增加了算法的通用性和鲁棒性。“Squeeze”的做法也极具创新性。不过实际的业务场景可能会对算法的准确性产生一定的影响，例如预测不准可能会导致聚类不准确、同时有多个异常或者外部异常也会影响聚类效果。另外按照GPS对结果根因进行排序有时也不符合真实业务场景，因为真实的业务场景数量的大小同样是一个重要的衡量指标。失败量从50到100的组合可能比0到5的组合更值得注意。


## 百度的方案

参考人工定位过程中的分析思路，提取了两个特征，用来描述某维度是否为根因：

1. 贡献度：即该维度PVLost与总PVLost的比例。
2. 一致度：即构成该维度的子维度的异常程度的相似度。子维度的异常程度的一致度可通过各子维度异常程度间的变异系数衡量，变异系数越小，则异常程度越一致。(如何衡量小？绝对值还是相对值?)

#  评估指标

评估准确性的指标是F-score,该指标是准确率（Precision）和召回率(Recall)综合体现。具体计算如下所示：
F-score =(2 ∗ Precision ∗ Recall)/(Precision+ Recall)，其中：
Precision ＝ TP / (TP + FP)，
Recall = TP / (TP + FN)。

每个异常时刻都有一个真正的根因集合，记为S*，该集合中包含一个或多个属性值组合，参赛队伍的算法输出结果 记为S。对于S*中的每一个元素，S中包含其中一个，则算一次true positive （TP），遗漏一个算一次false negative （FN），多出一个S*中不存在的，记一次false positive （FP）。计算出所有异常时刻总的TP、FP、FN，最后得出F-score。



# 代码地址

1. https://github.com/chiechie/ADminer



# 参考

1. [HotSpot-英文paper](https://netman.aiops.org/wp-content/uploads/2018/12/sunyq_IEEEAccess2018_HotSpot.pdf)
2. [多维指标异常定位-HotSpot](https://mp.weixin.qq.com/s/Kj309bzifIv4j80nZbGVZw)
3. [百度做根因钻](https://zhuanlan.zhihu.com/p/48926992)
4. [Adtributor](https://www.usenix.org/system/files/conference/nsdi14/nsdi14-paper-bhagwan.pdf)
5. [iDice](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/07/ICSE-2016-1-iDice-Problem-Identification-for-Emerging-Issues.pdf)
6. [Squeeze](https://netman.aiops.org/wp-content/uploads/2019/08/camera_ready.pdf)
7. [bizseer总结](https://www.bizseer.com/index.php?m=content&c=index&a=show&catid=26&id=3)
8. [比赛](http://iops.ai/competition_detail/?competition_id=8&flag=1)

