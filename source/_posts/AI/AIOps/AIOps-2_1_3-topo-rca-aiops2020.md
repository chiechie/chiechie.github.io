---
title: chapter2.1.3 基于拓扑的根因定位-AIOps挑战赛2020-亚信的方案
author: chiechie
mathjax: true
date: 2021-03-10 13:08:27
tags:
- AIOps
- 根因分析
- 微服务
- 图数据
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


总结下: 先从业务指标异常触发定位，然后从调用链中异常的服务，然后服务对应的机器，定位根因指标。

## 亚信的方案

> 对于调用链根因节点定位的模型实现，我们考虑过两种方案，一种是有监督的方法，即把一定时间切片内的调用链数据作特征提取，转化为一个多分类问题或者二分类问题，但这种方法泛化能力较差，严格依赖于当前系统的数据特征，不利于实际生产环境的迁移，另一方面，我们考虑到复赛数据特征可能与预赛数据不尽相同，因此我们选择了泛化性能更强的方法，也就是无监督的方法。 我们对每条调用链沿着调用关系进行动态搜索，使之构成一个节点间的依赖图，并在这个依赖图上定义了两种不同类型的故障模式，
>
>第一种定义为非疑似节点，指子节点延时除以父节点延时大于等于51%的节点,主要用于衡量子节点占用的父节点的资源。
第二种定义为疑似异常节点，指父节点耗时-子节点耗时，除以父节点耗时大于等于51%的子节点，衡量的是父节点自己占用的资源。
>
> 第二种异常模式可能不太容易理解，其实它可以来源于生活中一个十分直观的观察，例如我们从商店订购了一份外卖，2个小时后才收到，这时候我们应该认为是商店备餐慢，还是骑手送货慢呢，想要知道答案，只需要看一看同一时间，点了相同外卖的其他人，是否也出现了收货慢的情况。



![根因定位框架](./framework.png)

![调用链路的异常检测和根因定位](./rca.png)


## 参考资料
1. [AIOps挑战赛2020-官网](http://iops.ai/competition_detail/?competition_id=15&flag=1)
2. [2020AIOps挑战赛亚军亚信科技团队方案分享](https://mp.weixin.qq.com/s/hYiXUMveSprkIiOy8mCmCg)



