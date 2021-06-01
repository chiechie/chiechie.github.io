---
title: chapter2.3 调用链根因分析
author: chiechie
mathjax: true
date: 2021-05-21 16:49:19
tags:
- 调用链
- AIOps

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




## 调用链的数据格式

- 做调用链路分析（TraceAnomaly）用到了耗时，以及调用链的结构信息。
- 2021AIOps比赛的调用链的数据：每个span_id表示一个trace中的一次子调用，耗时表示当前服务从接受call，到开始response的时间间隔。这里的表格信息量很少，它不知道每次子调用，具体是那个应用在调用，也不知道此次子调用之前，该trace已经走过了哪些调用路径了。

## 代码

放这里: https://github.com/chiechie/TraceAnomaly


## 参考资料
1. [清华微众AIOps新作:基于深度学习的调用轨迹异常检测算法
-微信公众号](https://mp.weixin.qq.com/s/sqYIb6i9z6xF5nDr8fuVsA)
2. [chapter2.1.4 AIOps挑战赛2021-demo方案](https://chiechie.github.io/2021/03/09/AI/AIOps/AIOps-2_1_4-topo-rca-aiops2021/)
