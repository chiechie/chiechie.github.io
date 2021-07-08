---
title: chapter2.4 时间序列关联性分析
author: chiechie
mathjax: true
date: 2021-05-27 17:27:26
tags:
- 人工智能
- 时间序列
- 预测
- 论文笔记
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
	- [chapter2.2 多维下钻根因定位](../AIOps-2_2-multi-dimensional-rca/)
	- [chapter2.3 调用链根因分析](../AIOps-2_3-trace_rca/)
	- [chapter2.4 时间序列关联性分析](../AIOps-2_4-metric_event_correlation/)
- chapter3 故障恢复


> performance counter： 性能监视器

## 背景

为了保证产品的服务质量、减少服务宕机时间，从而避免更大的经济损失，对关键的服务事件的诊断显得尤为重要。实际的运维工作中，对服务事件进行诊断时，运维人员可以通过分析与服务事件相关的时序数据，来对事件发生的原因进行分析。虽然这个相关关系不能完全准确的反映真实的因果关系，但是仍然可以为诊断提供一些很好的线索和启发。

- 如下图， 两个事件：程序A启动和程序B启动，一个时序：cpu使用率。
  可以看到，事件（程序A的运行）与时序（CPU使用率）存在相关关系，并且程序A启动会导致CPU使用率升高，记为A->cpu上升。
  
  ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FMjzFINogsh.png?alt=media&token=9002fb74-9363-4a57-a182-fac68f04bd60)

希望能使用数据驱动的方法，找到类似的相关组合，也就是<事件, 指标>对，例如A->cpu上升，

## 基本概念

- 时间序列（metric）： 通常有固定的时间间隔，例如CPU使用率等；指标数据( Metrics Data )：描述具体某个对象某个时间点， CPU 百分比等等，指标数据等等。
- 事件（event）：就是一个正常的操作比如程序A的启动，这个是需要运维指定的 他感兴趣的某个日志模版。 比如“Out of memory” /启动某个磁盘intensive程序/ 启动某个cpu intensive程序/query-time out alerts/某个告警，类似log3C的日志pattern，big pandas 的变更事件。event报告告警和日志。
- 日志(logging)：例如有个应用出错，抛出了NullPointerExcepction。
- 事件序列（event sequence）：记录了特定事件发生的序列。
- 故障（incident）：故障定位，就是要找到这个故障的根因。 （感觉也能跟BigPanda的告警关联之后的结果 对应起来），


## 总结

- 这篇文章主要是解决的什么问题？找到事件（E，event）和指标（S，metric）的相关性。
- 具体的思路？将这个问题转换为统计中的两样本问题，并且使用邻近算法判断相关性。
- 难点是什么？异构数据 + 异常关联性，异构指的是文本+时序，异常关联性，指的是，只关心事件和指标异常时刻的相关性。 
- 用的什么分析方法？还可以应用到哪些领域？ 用2-sample解决event 和 指标 之间的关联关系，可推广到所有离散变量的相关性分析中。
- 这篇文章没有解决的问题是？这些问题有什么解决方法？ 本文还是比较学术派，只是给了一个 分析指标  和 事件 的相关性 方法，至于怎么从众多原始日志和告警中得到事件，以及分析出来的相关性怎么应用到故障定位上面去，没有说。但是这两步 对于实际应用来说 是必不可少的。
- BigPandas将众多告警基于业务关系（拓扑数据）收敛成incident，然后将incident跟变更事件进行关联，得到的变更事件就是root cause

### 技术细节

要解决两个问题：

1. 确定指标和事件的因果关系，即事件A-->指标1：两个随机分布的相似性指标
2. 确定因果关系中指标的单调性，即事件A-->指标1上升：判断两个随机变量的大小关系的指标

都是统计中的假设检验的方法。

判断方向可以转化为以下问题：

1. 判断事件发生前的 一段序列 跟 整个时序的相似性。如果差异大，说明指标 --> event
2. 判断事件发生后的 一段序列 跟 整个时序的相似性。如果差异大，说明event --> 指标


判断单调性，是这么做的：

1. 计算event前后，2个子序列的 均值的差
$$ t_{s c o r e}=\frac{\mu_{\Gamma^{f r o n t}}-\mu_{\Gamma^{r e a r}}}{\sqrt{\frac{\sigma_{\Gamma^{f} r o n t}^{2}+\sigma_{\Gamma^{r e a r}}^{2}}{n}}} $$
   
2. 判断单调性：
  - 如果$t_{score} > \alpha$, 得到负的单调性，即
    ![](img.png)
  - 如果$t_{score} < -\alpha$, 得到正的单调性。 
       
      这里P=0.025对应$\alpha=1.96$，P=0.001对应$\alpha=2.58$


### 实验

- ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FGkHOys2alH.png?alt=media&token=95dde9c8-0249-4033-a027-83bca9a543ff)
- ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2Fy5QnzhCHAl.png?alt=media&token=13774051-eb1f-4ada-a978-9cd7d2bee748)


## 参考

1. [微软-指标与事件的关联分析paper](http://www.microsoft.com/en-us/research/wp-content/uploads/2016/07/SIGKDD-2014-Correlating-Events-with-Time-Series-for-Incident-Diagnosis.pdf)
2. [关联分析code](https://github.com/jixinpu/aiopstools/tree/master/aiopstools/association_analysis):
3. [智能运维前沿-微软AIOps工作：时序数据与事件的关联分析](https://mp.weixin.qq.com/s/-NMwaCD4Kzkt4BTnr5JKDQ)
4. H. B. Mann， On a test of whether one of two random variables is stochastically larger than the other. The annals of mathematical statistics,  1947.
