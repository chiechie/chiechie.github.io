---
title: chapter2.1.5 基于拓扑的根因定位-CN-Softbei2020比赛
author: chiechie
mathjax: true
date: 2021-05-21 14:37:13
tags:
- AIOps
- 根因分析
- 微服务
- 图数据
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


> 数据来自CN-Softbei2020比赛--《基于网络拓扑及告警的故障根因定位系统实现及算法研究》
>
> 现在实验一种很激进的做法---仅靠指标做根因分析
> 
> 当涉及的指标过多时, 得到的拓扑图非常复杂.

**赛题名称:基于网络拓扑及告警的故障根因定位系统实现及算法研究**
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FQm8N2Yas2F.png?alt=media&token=110a4f63-df4b-4080-96e7-538078152112)
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FiscvbTqPV9.png?alt=media&token=0fd1674e-9bc7-4ac2-8f48-add9cdf2c946)
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2Fn2jiCvLL8K.png?alt=media&token=6ee81f61-a008-4040-b1d0-5968b6ab5765)
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FbiDgXGazeM.png?alt=media&token=60d55dac-68c7-42df-a7c4-4b8270ce0578)
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FW1v20u-7tH.png?alt=media&token=0702a100-6ee5-4e4d-98b1-a42e92ac1fa0)
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2Fsvpv3-38IA.png?alt=media&token=5b7e7f4f-0ac7-4180-bfa2-1e2447bffa3a)
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FzflLEiJ2fa.png?alt=media&token=d07f20d5-c2b7-4011-acd4-c6df97b7964b)
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FherPwjhHIR.png?alt=media&token=8cbd3491-37d9-46d6-9c12-7d6292e3689f)
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FF47x1XYXV2.png?alt=media&token=f837d32c-a170-4bf6-8a20-7be096537ba3)
    - 大型电商平台内部涉及上百个系统间的相互调用，每天会产生上万条告警数据。如何利用网络拓扑信息及告警数据，及时有效的对告警进行过滤、分析，最终给出有效告警及疑似根因是网络运维面临的主要挑战。
    - 本赛题的目标是分析实际生产中的网络拓扑及告警数据，设计并实现一种故障根因定位算法，准确的定位出故障根因节点。
    - 赛题的价值在于准确、快速的定位网络故障，提升一线网络运维的效率，降低网络故障产生的损失。
    - 该赛题主要涉及数据分析、图论、信息论、自然语言处理、神经网络等技术，要求参赛队员掌握相关的基础知识，具有python数据分析、处理及功能开发能力。
# 业务背景：

- 网络拓扑中一个节点出现故障，往往会导致与其相连的其他节点也发生异常，进而产生大量告警，将真正根因淹没掉。当出现大量告警时，为保证电商业务的稳定运行，我们需要对这些告警进行分析处理，过滤掉无效的告警，准确定位出可能的疑似根因节点，缩短故障定位时间。
- 在实际中，同一故障触发的告警信息往往相似的。网络故障定位的一般过程为：先根据历史告警数据及标记进行离线分析，总结因果关系的规律；然后在线分析阶段，首先对告警数据进行去噪和聚类，然后对类内的告警进行因果关系匹配，得到当前告警的可能根因。

# 赛题场景：

- 提供实际生产中的网络拓扑数据，100组不同时间段的告警数据，每组中可能有根因，也可能没有根因，根据拓扑数据及告警数据，设计并实现故障根因定位算法，最终在测试数据上评估告警根因定位的F1值，可视化展现根因信息及根因关联的局部拓扑图。

# 要求

- 1、基于提供的训练数据，设计并实现故障根因定位算法。
- 2、故障根因定位算法中要包含告警数据预处理和故障根因定位两个模块；
- 3、可视化展现根因信息及根因关联的局部拓扑图。

# 训练数据

- 1、实际生产中的部分网络拓扑数据，json格式，key为父亲节点，value为孩子节点list。
- 2、告警数据，csv格式，包括告警节点，告警信息，告警时间，告警级别，是否为根因。
- 3、提供100组不同时间段的告警数据，每组中可能有根因，也可能没有根因，如果有根因，只有1个准确的根因。
- 数据样例：

1. 拓扑图样例：
- key为源节点，value为目标节点list，类似下面这样：AAA表示系统名，NG表示是Nginx。
- ```javascript
{
  "AAA-10.10.0.1":
	["AAA-NG-10.10.0.2",
     "AAA-NG-10.10.0.3",
     "AAA-NG-10.10.0.4"],
   "AAA-NG-10.10.0.2":
   ["AAA-JBOSS-10.10.0.5",
    "AAA-JBOSS-10.10.0.6", 
    "AAA-JBOSS-10.10.0.7"]
}```
2。告警数据样例：

告警数据为csv格式，如下面所示，其中sysEname为系统名称, software为软件类型,time告警时间,triggername告警信息，里面包含ip信息，isrootcause表示是否为根因，0-否，1-是。
```javascript
,sysEname,software,time,告警信息,isrootcause
0,AAA,Elasticsearch 1.7.0 ,2019-06-01 00:01:04,"eth0 网卡接受数据流量,net.if.in[eth0],主机10.10.0.8(VM)网卡eth0接收流量持续10分钟平均值大于180M",0
1,BBB,WildFly standalone ,2019-06-01 00:01:05,主机10.10.0.9日志中有READONLY信息,0
2,BBB,WildFly standalone ,2019-06-01 00:01:06,主机10.10.0.10日志中有READONLY信息,0
```
测试数据：
- 1、提供20组训练数据，格式同训练数据，不提供根因标注（没有is_root列）
- ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2F0iBs9_I--S.png?alt=media&token=3963d6db-13df-4b57-9318-a0eb008c6766)

总结一下，
-取发生故障的这段时间内(每次10到15min)：
	- ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FhT-H9-YnH5.png?alt=media&token=2b7a6f5f-c828-49c0-b6be-f274de231c1d)
	- 所有告警数据： 来自哪台机器，那个系统，哪个软件，告警描述
	- 软硬件拓扑数据：静态的，
	- 根因incident可能存在也可能不存在
	- 根因incident可能是1个也有可能是多个（多个是因为持续异常，所以就持续告警）
		- ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FRgMOthtgUp.png?alt=media&token=37da69ab-2425-472f-acc1-7ea85c517cf3)
		- 
	- 训练数据有标记，人工标的根因事件
	- 根因分析的样本粒度是 event，即一个告警事件，它的特征包括，主机id，系统名称，软件类型（optinal），告警信息

- 还没有搞清楚，如何处理这些数据，已经定位到根因？
    - 如何利用拓扑数据？
    - 如何利用标记数据？
    - 如何利用历史数据？--历史的告警数据
- 看一下nokia的解决方案把[[bell给nokia做 故障诊断的 方案]]

## 数据解析


![dataoverview](dataoverview.png)

将每个事件看成一个指标的话，可以得到一个节点为2876的图

## 关联分析

![corr](./orr.png)

![节点个数](./img.png)

对应的 因果图巨大无比，如下：

![图1-关联拓扑](./correlation_topology.png)




## 参考
1. [CN-Softbei2020比赛](http://www.cnsoftbei.com/plus/view.php?aid=479)
2. [chiechie的代码-github](https://github.com/chiechie/Insighter/blob/master/aiops_CN-Softbei_2020.ipynb)