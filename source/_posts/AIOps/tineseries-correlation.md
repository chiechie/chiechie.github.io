---
title: 时间序列关联性分析
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

## 背景

为了保证产品的服务质量、减少服务宕机时间，从而避免更大的经济损失，对关键的服务事件的诊断显得尤为重要。实际的运维工作中，对服务事件进行诊断时，运维人员可以通过分析与服务事件相关的时序数据，来对事件发生的原因进行分析。虽然这个相关关系不能完全准确的反映真实的因果关系，但是仍然可以为诊断提供一些很好的线索和启发。

## 基本概念

- 时间序列（metric）： 通常有固定的时间间隔，例如CPU使用率等；指标数据( Metrics Data )：描述具体某个对象某个时间点， CPU 百分比等等，指标数据等等。
- 事件（event）：就是一个正常的操作比如程序A的启动，这个是需要运维指定的 他感兴趣的某个日志模版。 比如“Out of memory” /启动某个磁盘intensive程序/ 启动某个cpu intensive程序/query-time out alerts/某个告警，类似log3C的日志pattern，big pandas 的变更事件。event报告告警和日志。
- 日志(logging）：例如有个应用出错，抛出了NullPointerExcepction。
- 事件序列（event sequence）：记录了特定事件发生的序列。
- 故障（incident）：故障定位，就是要找到这个故障的根因。 （感觉也能跟BigPanda的告警关联之后的结果 对应起来），


## 总结

- 本文将事件数据（E，Event）和时序（S，metric）数据相关关系问题转化为两样本问题（two-sample problem），并使用邻近算法（nearest neighbor method）判断是否相关。主要回答了三个问题：
    - E和S之间是否存在相关关系？ 使用nn方法判断相关性是否存在
    - 若存在相关关系，E和S的时间先后顺序是什么？E先发生，还是S先发生？
    - E和S的单调关系。假设S（或者E）先发生，S的增加还是降低导致的E发生？ 更进一步，相关性 是否 有特定的时序性，以及大小的先后性，通过分析event之前的子序列 和event之后的子序列的相关性。
- 如图，事件为程序A和B的运行，时序数据为CPU使用率。可以发现，事件（程序A的运行）与时序数据（CPU使用率）存在相关关系，并且是程序A启动会导致CPU使用率升高。
  ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FMjzFINogsh.png?alt=media&token=9002fb74-9363-4a57-a182-fac68f04bd60)

- 这篇文章主要是解决的什么问题？希望找到跟incident有关联的events。 
- 具体的思路？先找 服务整体健康度的指标-即 服务是否可用 或者 请求延迟，也就是主指标，再找 这个 总体KPI 有关联关系的 一批系统指标（副指标）
- 「关联关系」怎么定义？正常相关性，异常相关性，还是 业务上的关联性？
- 难点是什么？异构数据 + 异常关联性，异常关联性，反应的是 event 和 metric 的异常相关性，而不是 正常+异常相关性。
- 这篇文章用的什么分析方法？还可以应用到哪些领域？ 用2-sample解决event 和 指标 之间的关联关系，可推广所有离散变量 的相关性分析 和 因果分析方法。
- 这篇文章没有解决的问题是？这些问题有什么解决方法？ 本文还是比较学术派，只是给了一个 分析指标  和 事件 的相关性 方法，至于怎么从众多原始日志和告警中得到事件，以及分析出来的相关性怎么应用到故障定位上面去，没有说。但是这两步 对于实际应用来说 是必不可少的。
- bigpandas是怎么做的？将众多告警基于 业务关系（拓扑数据）收敛成incident，然后将incident跟变更事件进行关联，得到的变更事件就是root cause
- 用来鉴定日志序列 和 时间序列 关联关系  和 因果关系 的方法是什么？ 


### 实验

- ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FGkHOys2alH.png?alt=media&token=95dde9c8-0249-4033-a027-83bca9a543ff)
- ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2Fy5QnzhCHAl.png?alt=media&token=13774051-eb1f-4ada-a978-9cd7d2bee748)



### 技术细节

涉及到具体实现，要解决两个问题：

1. 确定指标和事件的因果关系，即事件A-->指标1； 
2. 确定因果关系中指标的单调性，即事件A-->指标1上升。

这里用到了2个trick:一个是用两个随机分布的相似性指标来分析因果关系，一个是用两个随机变量的大小关系的指标用来判断关联到的指标的方向，两个都都是统计的假设检验的方法。

具体来说，判断因果关系，可转换为以下2个问题：

1. 判断事件发生前的 一段序列 跟 整个时序的相似性。如果差异大，说明指标 --> event
2. 判断事件发生后的 一段序列 跟 整个时序的相似性。如果差异大，说明event --> 指标


判断单调性，是这么做的：

1. 计算event前后，2个子序列的 均值的差
$$ t_{s c o r e}=\frac{\mu_{\Gamma^{f r o n t}}-\mu_{\Gamma^{r e a r}}}{\sqrt{\frac{\sigma_{\Gamma^{f} r o n t}^{2}+\sigma_{\Gamma^{r e a r}}^{2}}{n}}} $$
2. 具体的判断规则：
- 如果$t_{score} > \alpha$, 得到负的单调性，即![img.png](img.png)
- 如果$t_{score} < -\alpha$, 得到正的单调性。 
     
    这里P=0.025对应$\alpha=1.96$，P=0.001对应$\alpha=2.58$


## 参考

1. [微软-指标与事件的关联分析paper](http://www.microsoft.com/en-us/research/wp-content/uploads/2016/07/SIGKDD-2014-Correlating-Events-with-Time-Series-for-Incident-Diagnosis.pdf)
2. [关联分析code](https://github.com/jixinpu/aiopstools/tree/master/aiopstools/association_analysis):
3. [智能运维前沿-微软AIOps工作：时序数据与事件的关联分析](https://mp.weixin.qq.com/s/-NMwaCD4Kzkt4BTnr5JKDQ)
4. H. B. Mann and D. R. Whitney. On a test of whether one of two random variables is stochastically larger than the other. The annals of mathematical statistics,  1947.
