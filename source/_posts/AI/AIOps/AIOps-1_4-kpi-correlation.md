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




《KPI波动关联分析助力故障排查-CoFlux》是清华和阿里巴巴公司的最新AIOps合作成果。
CoFlux挖掘KPI之间的异常波动关联，从而帮助运维人员进行故障排查，具体的，波动关联关系结果可以在以下三个方面帮助进行故障排查：报警压缩、推荐TOP-N的可能原因、构建异常波动传播链。该工作对应的论文《

## 背景

为了对互联网公司的各项服务进行管理，运维人员通常会监控收集成千上万的关键性能指标KPI（Key Performance Indicator），这些KPI（例如服务请求数量、服务请求成功率等）的形式多为时间序列。

在实际的运维管理工作中，由于各种原因（例如网络中断、恶意攻击等），公司的服务中断不可避免。当服务发生故障时，与故障原因相关的许多KPI数据也会出现异常的波动，而且这些波动也会传递到其他有业务关联或者模块调用关系的KPI，形成报警风暴。报警风暴产生的报警邮件和短信会使得运维人员很头大，因为其中的大量报警都是冗余的，只有少数的报警需要运维人员去关注和解决。此外，发生异常波动的KPI交织在一起，也使得故障排查工作费时费力非常困难。如果可以自动挖掘KPI之间的异常波动关系，就可以帮助运维人员进行更加高效智能地进行故障排查。


![img](https://mmbiz.qpic.cn/mmbiz_png/RVMJ0J4z0LzxCnAk4d5fhVM1Gib6rgkCZH8bficj36KtuKiczrZHcqq6NQunbIkMowVN1w0Njyvv3z9AjvsfmD2VA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

## 问题描述

![img](https://mmbiz.qpic.cn/mmbiz_png/RVMJ0J4z0LzxCnAk4d5fhVM1Gib6rgkCZH8bficj36KtuKiczrZHcqq6NQunbIkMowVN1w0Njyvv3z9AjvsfmD2VA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

![img](https://mmbiz.qpic.cn/mmbiz_jpg/RVMJ0J4z0LzxCnAk4d5fhVM1Gib6rgkCZ2oGk0FGldica8CEhhF4yZOTtia9a5mDcpiaP7H1uLj83VKqbfoKVBLXSA/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

图1 时间序列，预测序列，波动特征序列

​       对于一个时间序列$S = [*s**1**, s**2* *,…, s**m*]$，*si*是KPI S在时刻i的数据，m是KPI的长度。对于单一KPI，在数据采集和预处理时要求相邻时刻数据间的时间间隔相同。对于两个KPI，如果时间间隔不同，可以取两个KPI时间间隔的最小公倍数作为公共的时间间隔。KPI S的预测序列P = [*p**1, p2, …, pm*]，*pi*是*si*的预测值。因此，一个KPI的预测误差序列F = [*f**1, f2, …, fm*]，*fi =  si* – *p**i*。对于一个KPI，正常的部分是比较准确容易的被预测出来，但是异常波动部分通常都是由一些不可预见的突发因素导致的，很难被预测。因此，预测误差可以很好的用来表示KPI的波动特征，CoFlux使用KPI的预测误差序列来表示该KPI的波动特征。图1 展示了一个时间序列，预测序列，波动特征序列的实例。此外，用于得到预测序列的预测模型以及对应的参数即为特征检测器。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/RVMJ0J4z0LzxCnAk4d5fhVM1Gib6rgkCZND6kn9hTDotgLwYhlsUKo8dy8YT6cxv1tic71w6ibSZl3ykZPNIXrsaw/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1)

图2 （左）6个KPI，（右）KPI对应的波动特征曲线

​       CoFlux的目的是挖掘两个KPI之间的波动关联关系，这种波动关联关系可以帮助进行故障排查。具体的，这种关系包括三个具体问题：1. 两个KPI的波动是否相关？如果相关，2. 波动的先后顺序如何（同时发生或者某根曲线的波动先发生）？3. 波动的方向是否一致（波动方向一致或者相反）？

​      利用原始的KPI曲线来判断波动关联关系比较困难，我们可以借助KPI的波动特征曲线。如果两个KPI的波动特征曲线相关（即波动同时发生或者发生时有一定的相位差），那么原始的两个KPI的波动相关。图2展示了6根KPI及其对应的波动特征曲线。从图2中可以看出，*K1*和*K2*、*K3*波动相关，但是与*K4*波动无关；*K1*的波动先于*K2*和*K3*发生，且波动方向相反；*K2*和*K3*的波动同时发生，且波动方向相同。

## 挑战



分析判断KPI的波动关联关系，主要有三个挑战：

1. KPI曲线数据众多且曲线特征各异（例如不同的周期性、平稳性、趋势等），因此，没有通用的方法可以对所有的KPI进行波动特征提取。

2. 基于异常检测结果的波动关联关系分析不合适。基于异常检测的方式需要挑选合适的异常检测算法并进行异常的定义（异常的阈值等），并且二分类的异常结果也比波动特征的信息量要少很多。

3. 波动关联关系复杂。正如在问题描述中所说，两个KPI间的波动可能有不同的先后顺序以及波动方向，这给判断波动关联关系时带来了更大的挑战。



![img](https://mmbiz.qpic.cn/mmbiz_png/RVMJ0J4z0LzxCnAk4d5fhVM1Gib6rgkCZH8bficj36KtuKiczrZHcqq6NQunbIkMowVN1w0Njyvv3z9AjvsfmD2VA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



## 方法



图3 CoFlux方法架构图

​       CoFlux的输入是两根KPI曲线，输出结果是波动关联关系：两个KPI是否波动相关，如果相关，同时输出波动的先后顺序和波动的方向。算法整体分为两个大的部分：特征工程和相关性测量。

​       特征工程的目的是为KPI找到合适的特征检测器，由于不同KPI有不同的特征，单一的特征检测器无法适用于所有的KPI。因此，我们基于两个直观假设：如果可以提供足够多的特征检测算法，那么，1.对于任何一个KPI，总有一个算法可以为该KPI提取到合适的波动特征。2.如果两个KPI波动相关，至少有来自两个KPI的波动特征也相关。相关性测量的目的是根据特征工程得到波动特征计算得到相关性结果。接下来，我们具体介绍特征工程和相关性测量。

**第一步，特征工程**。特征工程主要包括两个步骤，特征提取、特征放大。

特征提取：如表1，特征提取主要包含了我们预选取的预测模型及参数，总计一共有86个检测器，因此对于一个KPI，我们可以得到86个波动特征。

表1 CoFlux中的预测模型和检测器

![img]

​       特征放大：一个KPI大部分时间范围内都是正常的，没有很大波动，只有随机的噪声。只有当服务受到影响时，才会产生波动。因此，波动的数量是远远小于正常数据的。为了削弱噪声的影响，我们采用改进版的激励函数：一个KPI的波动程度越大，波动特征也就会被放大的越大，这样就使得我们的波动特征更具区别度，对最后的相关性判断也更有帮助。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

​       **第二步，相关性判断**。对两个KPI的波动特征进行相关性判断时，需要考虑到KPI波动的轻微形变以及相位差。因此，我们挑选Cross-Correlation来测量两个波动特征的相关性结果。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)



![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

## 实验

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

​       实验部分包括了两个数据集：数据集1：波动相关的两个KPI的原始曲线数据特征不同；数据集2：波动相关的两个KPI的原始数据曲线特征相似。由于没有专门的处理波动关联分析的算法，我们选取了已有的五种关联分析工作以及变种（一共有7种方法）作为对比算法，表2展示了CoFlux方法和对比算法在两个数据集上的结果。结果表明CoFlux在两个数据集上都比其他的算法效果要更好，且在三个问题上的F1-score分别大于0.84，0.92，和0.95。

表2 CoFlux和对比方法在两个数据集的结果

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

接下来介绍CoFlux在故障排查工作中的应用实例。

报警压缩：对于大量的KPI，我们首先利用CoFlux计算所有KPI的两两之间的波动相关程度，然后使用K-means进行聚类（K可以使用轮廓系数方法选择）。每一类内的KPI可以当成一个报警簇，在报警的时候当成一个整体进行报警。图4展示了CoFlux利用24根KPI的波动相关结果聚成3类报警簇的实例。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

图4 报警压缩实例热度图表示

推荐TOPN的可能原因：如图5，对于任一KPI X，我们可以通过CoFlux找到该KPI的TOPN相关的KPI曲线。在进行故障排查时，运维人员可以优先检查这TOPN的曲线来对KPI X进行快速异常分析。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)图5 推荐TOPN的可能原因实例

​       构建异常波动传播链：我们可以利用KPI之间的波动传播关系来构建异常波动传播链，这个异常波动传播链可以反映不同KPI之间的波动是如何关联在一起的。图6 展示了CoFlux在数据库服务上自动构建波动传播链的实例，相比较人工的方式，CoFlux在不需要专家领域知识的情况下可以自动准确的构建异常波动传播链。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

图6 （左）人工构建的异常波动传播链；（右）根据CoFlux自动构建的异常波动传播链



![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

## 结论

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

利用KPI的波动特征进行关联分析对于故障排查是非常有用的，本文是服务运维管理领域第一篇定义研究这种波动关联分析的工作。未来的工作计划是采用深度学习的方法更加准确的提取KPI的波动特征。更多细节可以点击“阅读原文”进行阅读。


## 参考

1. CoFlux: Robustly Correlating KPIs by Fluctuations for Service Troubleshooting》发表在IWQoS 2019
2. [coflux中文介绍](https://mp.weixin.qq.com/s/SpiIquuz-8Ud_C3e4oVyaQ)