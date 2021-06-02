---
title:  chapter1.4 指标异常关联
author: chiechie
mathjax: true
date: 2021-05-21 22:32:17
tags:
- AIOps
- 异常检测
- 异常分类
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
	- [chapter2.2 多维下钻根因定位](../AIOps-2_2-multi-dimensional-rca/)
	- [chapter2.3 调用链根因分析](../AIOps-2_3-trace_rca/)
	- [chapter2.4 时间序列关联性分析](../AIOps-2_4-metric_event_correlation/)
- chapter3 故障恢复

> CoFlux挖掘KPI之间的异常波动关联, 是清华和阿里巴巴公司的合作成果。
> 
> 波动关联关系结果结果可应用于：报警压缩、推荐TOP-N的可能原因、构建异常波动传播链。

# 问题描述


在实际的运维管理工作中，当服务发生故障时，与故障原因相关的许多KPI数据也会出波动。如果可以自动挖掘KPI之间的异常波动关系，就可以帮助运维进行更加高效地排查故障。

CoFlux的目的是挖掘两个KPI之间的波动关联关系，这种关系包括三个具体问题： 
1. 两个KPI的波动是否相关？如果相关，
2. 波动的先后顺序如何（同时发生或者某根曲线的波动先发生）？
3. 波动的方向是否一致（波动方向一致或者相反）？


# 方案介绍

## 符号说明
- $S = [s_1, s_2,…, s_m]$：表示对于一个时间序列S，$s_i$是KPI S在时刻i的数据，m是KPI的长度。
- $P = [p_1, p_2, …, p_m]$：表示KPI S的预测序列，$p_i$是$s_i$的预测值。
- $F = [f_1, f_2, …, f_m]$：表示预测残差序列，$f_i =  s_i – *p_i$

CoFlux使用KPI的预测误差序列来表示该KPI的波动特征。如果两个KPI的波动特征曲线相关（即波动同时发生或者发生时有一定的相位差），那么两个KPI的波动相关。

## 难点

分析判断KPI的波动关联关系，主要的难点为，KPI曲线数据众多且曲线特征各异（例如不同的周期性、平稳性、趋势等），因此，没有通用的方法可以对所有的KPI进行波动特征提取。


## 方法


CoFlux的输入是两根KPI曲线，输出结果是波动关联关系：两个KPI是否波动相关，如果相关，同时输出波动的先后顺序和波动的方向。

算法整体分为两个大的部分：特征工程和相关性计算。

![CoFlux架构](img_1.png)

### 特征工程

特征工程主要包括两个步骤，特征提取、特征放大。

 特征放大：一个KPI大部分时间范围内都是正常的，没有很大波动，只有随机的噪声。只有当服务受到影响时，才会产生波动。因此，波动的数量是远远小于正常数据的。为了削弱噪声的影响，我们采用改进版的激励函数：一个KPI的波动程度越大，波动特征也就会被放大的越大，这样就使得我们的波动特征更具区别度，对最后的相关性判断也更有帮助。

### 相关性判断

对两个KPI的波动特征进行相关性判断时，需要考虑到KPI波动的轻微形变以及相位差。因此，我们挑选Cross-Correlation来测量两个波动特征的相关性结果。

![相关性判断](img_2.png)

### 应用实例

CoFlux的结果可以在以下三个方面助力故障排查：

- 报警压缩：对于大量的KPI，我们首先利用CoFlux计算所有KPI的两两之间的波动相关程度，然后使用K-means进行聚类（K可以使用轮廓系数方法选择）。每一类内的KPI可以当成一个报警簇，在报警的时候当成一个整体进行报警。
- 推荐TOP N的可能原因：对于任一KPI X，通过CoFlux可找到该KPI的TOP N相关的KPI曲线。在进行故障排查时，运维人员可以优先检查这TOP N个指标。
- 构建异常波动传播链：我们可以利用KPI之间的波动传播关系来构建异常波动传播链，这个异常波动传播链可以反映不同KPI之间的波动是如何关联在一起的。相比较人工的方式，CoFlux在不需要专家领域知识的情况下可以自动准确的构建异常波动传播链。


# 参考

1. [CoFlux英文paper](https://netman.aiops.org/wp-content/uploads/2019/05/CoFlux_camera-ready1.pdf)
2. [CoFlux中文介绍](https://mp.weixin.qq.com/s/SpiIquuz-8Ud_C3e4oVyaQ)