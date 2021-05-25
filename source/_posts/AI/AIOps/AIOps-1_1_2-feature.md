---
title: chapter1.1 单指标异常检测
author: chiechie
mathjax: true
date: 2021-05-06 16:37:13
tags:
- AIOps
- 异常检测
- 异常分类
categories: 
- AIOps
---

## 目录


- [chapter0 概览](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-0-summary/)
- [chapter1 故障发现](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-1-event-generate.md/)
	- [chapter1.1 单指标异常检测](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-1_1-kpi-detector/)
	- [chapter1.3 故障预测](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-1_3-fault-prediction/)
	- [chapter1.4 指标异常关联](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-1_4-kpi-correlation.md)
	- [chapter1.5 日志聚类](https://chiechie.github.io/2021/05/06/AI/AIOps/AIOps-1_5-log-analysis/)
		- [chapter1.5.1 使用logmine加强版做日志聚类](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-1_5_1-log-analysis_logmine/)
		- [chapter1.5.2 美团日志聚类](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-1_5_2-log-analysis_meituan/)

- [chapter2 故障定位](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-2-event-analysis/)
	- [chapter2.1 微服务系统的故障定位](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-2_1-topo-rca/)
		- [chapter2.1.1 CauseInfer1](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-2_1_1-topo-rca-causeinfer-notes1/)
		- [chapter2.1.2 CauseInfer2](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-2_1_2-topo-rca-causeinfer-notes2/)
		- [chapter2.1.3 AIOps挑战赛2020-获奖方案分享](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-2_1_3-topo-rca-aiops2020)
		- [chapter2.1.4 AIOps挑战赛2021-demo方案](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-2_1_4-topo-rca-aiops2021/)
		- [chapter2.1.5 N-Softbei2020比赛](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-2_1_5-topo-rca-cnsoftbei2020)
		- [chapter2.1.6 MicroCause](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-2_1_6-topo-rca-MicroCause)
	- [chapter2.2 多维下钻根因定位](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-2_2-multi-dimensional-rca/): 暂无
	- [chapter2.3 调用链数据的预处理](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-2_3-trace_rca/)
	- [chapter2.4 时间序列关联性分析](https://chiechie.github.io/2021/04/14/AI/AIOps/AIOps-2_4-metric_event_correlation/)
- chapter3 故障恢复


## 1. 概述

大型集群系统中，可能存在软件问题和硬件问题导致的系统故障，严重影响了系统的高可用性。这就要求7*24小时，对系统不间断监控。这就意味着需要不间断地监控大量时间序列数据，以便检测系统潜在的故障和异常现象。然而，实际当中的系统异常很多，且不容易发现；从而导致人工方式监控方式效率很低。

异常场景本质上是一个或者多个数据点；数据点一般在系统运行过程中产生，且能反应系统的功能是否正常，多以日志形式呈现。当系统功能发生异常时，就会产生异常数据。快速高效地发现这些异常值，对于快速止损具有重要意义。对此，我们提出一种基于时间序列的异常识别模型，用来及时发现异常。

对于多数系统，一般都有成功率、流量等指标，故障发生时，这些指标也会出现响应的异常。我们将系统成功率、流量统一称为特征值变量，并对其进行建模，从而方便后续其它特征变量的扩展。为了更好地感知这些特征变量的突变，需要对特征变量进行计算处理或者空间转换。那么异常识别问题就转换为以下两个问题：

- 特征变量的计算处理和转换
- 突变的判断

针对这两个关键问题，我们将在下文中进行建模和分析。

## 2. 异常识别

如下图，通过计算器进行特征变量的计算处理和转换，通过异常检测器来判断数值的突变，从而解决上面的两个问题。其中，异常检测器由比较器和决策器组成。

[![image-20180815093422131](http://www.datadriven.top/images/image-20180815093422131.png)](http://www.datadriven.top/images/image-20180815093422131.png)image-20180815093422131

对于给定时间序列二维矩阵X={xmt∈R：∀t≥0,∀m≥0}X={xtm∈R：∀t≥0,∀m≥0} ，xmtxtm为tt时刻的第m个指标的真实数据，umtutm表示时间tt的xmtxtm的计算值，ymtytm为第m个指标的输出结果，ytyt为整体预测结果。

xmtxtm通过计算器得到计算值umtutm，然后xmtxtm 和 umtutm分别作为比较器的输入，得到第m个指标的输出ymtytm。y1tyt1,y2tyt2…ymtytm作为决策器的输入得到ytyt。ytyt是一个二元值，可以用`TRUE`（表示输出数据正常），`FALSE`（表示输入数据异常）表示。下面对计算器和检测器进行说明。

### 2.1 计算器

**计算器**用来对输入值xmtxtm 进行计算或者空间转换，从而得到特征变量的计算值umtutm。一般情况下，特征变量具有趋势性、周期性等特征。基于这些特征，计算值的获取，可以使用以下三种方式：累计窗口均值计算器、基于趋势性的环比计算器、基于周期性的同比计算器。

#### 2.1.1 累积窗口均值计算器

输入值为xtxt（为了方便省略指标参数`m`），如果直接只用单个点xtxt的抖动来判断，受噪声影响较大。因此，使用累积窗口均值的方式：



![image-20191012164029688](../_image/image-20191012164029688.png)



其中，ww为累计窗口的大小。通过窗口平滑之后，会过滤掉尖刺等噪声。

#### 2.1.2 基于趋势性的计算器

为了描述数据的趋势性，引入环比类算法。对$x_t$进行空间转换，得到环比，再使用检测器进行检测。



![image-20191012163924379](/Users/stellazhao/research_space/EasyMLBOOK/_image/image-20191012163924379.png)



其中，分子为当前窗口w内的数据，分母为上一窗口w内数据。通过窗口w对数据进行平滑。

#### 2.1.3 基于周期性的计算器

为了描述数据的周期性，引入同比算法。当同比值过大或者过小时，认为发生故障。同比公式如下：



![image-20191012164057696](/Users/stellazhao/research_space/EasyMLBOOK/_image/image-20191012164057696.png)



其中TT为周期，kk表示第几个周期。一般选取kk为`1`、`7`、`30`，来表示昨天、上周、上个月。

#### 2.1.4 其他类型计算器

计算器还可以使用其他算法，包括：

- 统计类算法：包括同比、环比算法的改进，或者其他统计算法。此时，计算器的输出结果为预测值，预测值和输入值进行比较即可。
- 时序型算法：包含ARIMA、Holter-Winter等时序型算法。计算器的输出结果为预测值。
- 机器学习：根据有监督、无监督、深度学习(LSTM)等算法，训练出的模型即为计算器。此时，计算器的输出结果一般为归一化的值，根据归一化的值进行比较。

这些算法，在这里不再做深入研究和阐述。

### 2.2 异常检测器

当数据出现异常时，计算值会出现较大偏差，该偏差由**异常检测器**来判断。**异常检测器**由比较器和决策器组成，计算值和真实值通过该模块后，得到最终预测结果。

#### 2.2.1 比较器

比较器的本质是求解如下公式的过程：



f(xmt,umt;hm)  =  boolean(4)(4)f(xtm,utm;hm)  =  boolean



其中，xmtxtm为真实值，uu为计算值，hmhm为阈值参数，booleanboolean为结果`TRUE/FALSE`。真实值已知，计算值通过计算器得到；剩下的阈值参数hmhm，则需要根据故障发生时的实际值进行参数估计。

很多场景下，该公式还可以简化为：f(umt;hm)  =  booleanf(utm;hm)  =  boolean ，即计算值直接和阈值比较即可。

##### 2.2.1.1 比较器种类

比较器有两种：相对值比较器和绝对值比较器。给定计算值umtutm和输入值xmtxtm，得到绝对值比较器：



f=xmt−umt  opretor  hm(5)(5)f=xtm−utm  opretor  hm



其中，opretoropretor为比较操作符，比如`> < >= <=`。由于utut由xtxt得到，所以很多情况下公式可以简化为 umtopretorhmtutmopretorhtm，即确定计算值的阈值即可。

对于一些场景来说，需要捕获特征变量的相对性。因此，引入相对值比较器：



f=xmt−umtumt  opretor  hm(6)(6)f=xtm−utmutm  opretor  hm



通过对相对值比较器进行阈值处理，既可以检测异常值，同时还能对期望值进行归一化。

##### 2.2.1.2 比较器阈值h的选取

一般情况下，阈值参数决定了异常检测模块的敏感度。最优阈值的选择，取决于数据分布的性质以及先验数据。一般情况下，**阈值的选取方法为**：

- 方法一：跟踪一组故障数据和正常数据，根据经验估计阈值。
- 方法二：跟踪一组故障数据和正常数据，根据经验，并结合`3σ准则`确定，来确定阈值。（特征变量或者特征变量的组合，服从正态分布）

#### 2.2.2 决策器

如下公式，基于逻辑操作符，对比较器结果进行合并.

- 方式一：逻辑组合



yt=y1t  &|  y2t  &|  y3t  &|  ...  ymt(7)(7)yt=yt1  &|  yt2  &|  yt3  &|  ...  ytm



其中，||表示逻辑或操作，&&表示逻辑与操作。

- 方式二：权重设置法

  

  yt=k1∗y1t  +  k2∗y2t  +  k3∗y3t  +  ...  km∗ymt(8)

  

其中，kmkm为系数，这种方式一般适合基本无负样本的场景，参数的确定需要使用`层次分析法`，将在后面的文章进行说明。

## 3. 故障止损

上面主要阐述了异常识别的方式。如果条件过于严格，刚开始并不容易被识别出来；如果条件过松，可能导致误识别。对此，我们将止损策略分为两级：

- 级别一：预警。对于不能完全确定故障发生的场景，使用级别一。
- 级别二：预警+止损（踢IDC）。对于能确定IDC故障的场景，使用级别二。

## 4. 实际场景应用

下面通过一个规则的场景，进行举例说明。假如存在如下异常场景：

[![image-20181108211144340](http://www.datadriven.top/images/image-20181108211144340.png)](http://www.datadriven.top/images/image-20181108211144340.png)image-20181108211144340

**体现在模型中，则级别一（预警）的模型图**

[![image-20180815093336600](http://www.datadriven.top/images/image-20180815093336600.png)](http://www.datadriven.top/images/image-20180815093336600.png)image-20180815093336600

**级别二（预警+踢IDC）的模型图：**

[![image-20180815093352931](http://www.datadriven.top/images/image-20180815093352931.png)](http://www.datadriven.top/images/image-20180815093352931.png)image-20180815093352931

最终，得到故障识别规则：

- 级别一触发条件: u1<h1  |  (u5<h5  &  u6<h6  &  u7<h7)u1<h1  |  (u5<h5  &  u6<h6  &  u7<h7)
- 级别二触发条件：u1<h2  &  u3>h3u1<h2  &  u3>h3

其中，h1,h2,h3,h5,h6,h7h1,h2,h3,h5,h6,h7为阈值参数。需要结合经验和实际数据估计得到。

## 5. 小结

本文主要基于时间序列的数据，提出了异常场景识别模型，并重点对基于规则的识别进行了说明。

## 参考

1. [Generic and Scalable Framework for Automated Time-series Anomaly Detection](http://dl.acm.org/citation.cfm?id=2788611)
2. [数据驱动应用（一）：整体概述](http://www.datadriven.top/categories/数据驱动/)

