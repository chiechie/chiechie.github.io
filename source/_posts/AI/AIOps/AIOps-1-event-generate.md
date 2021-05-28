---
title: chapter1 故障发现
author: chiechie
mathjax: true
date: 2021-05-21 16:03:52
tags:
- AIOps
- 异常检测
- 异常分类
categories: 
- AIOps
---

## 目录

- [chapter0 概览](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-0-summary/)
- [chapter1 故障发现](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-1-event-generate/)
	- [chapter1.1 单指标异常检测](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-1_1-kpi-detector/)
	- [chapter1.3 故障预测](https://chiechie.github.io/2021/03/04/AI/AIOps/AIOps-1_2-fault-prediction/)
	- [chapter1.4 指标异常关联](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-1_4-kpi-correlation/)
	- [chapter1.5 日志聚类](https://chiechie.github.io/2021/02/19/AI/AIOps/AIOps-1_5-log-analysis/)
		- [chapter1.5.1 使用logmine加强版做日志聚类](https://chiechie.github.io/2021/03/04/AI/AIOps/AIOps-1_5_1-log-analysis_logmine/)
		- [chapter1.5.2 美团日志聚类](https://chiechie.github.io/2021/03/04/AI/AIOps/AIOps-1_5_2-log-analysis_meituan/)
	
- [chapter2 故障定位](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-2-event-analysis/)
	- [chapter2.1 微服务系统的故障定位](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-2_1-topo-rca/)
		- [chapter2.1.1 CauseInfer1](https://chiechie.github.io/2021/03/02/AI/AIOps/AIOps-2_1_1-topo-rca-causeinfer-notes1/)
		- [chapter2.1.2 CauseInfer2](https://chiechie.github.io/2021/03/03/AI/AIOps/AIOps-2_1_2-topo-rca-causeinfer-notes2/)
		- [chapter2.1.3 AIOps挑战赛2020-获奖方案分享](https://chiechie.github.io/2021/03/10/AI/AIOps/AIOps-2_1_3-topo-rca-aiops2020/)
		- [chapter2.1.4 AIOps挑战赛2021-demo方案](https://chiechie.github.io/2021/03/09/AI/AIOps/AIOps-2_1_4-topo-rca-aiops2021/)
		- [chapter2.1.5 N-Softbei2020比赛](https://chiechie.github.io/2021/03/10/AI/AIOps/AIOps-2_1_5-topo-rca-cnsoftbei2020/)
		- [chapter2.1.6 MicroCause](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-2_1_6-topo-rca-MicroCause)
	- [chapter2.2 多维下钻根因定位](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-2_2-multi-dimensional-rca/): 暂无
	- [chapter2.3 调用链根因分析](https://chiechie.github.io/2021/03/15/AI/AIOps/AIOps-2_3-trace_rca/)
	- [chapter2.4 时间序列关联性分析](https://chiechie.github.io/2021/04/14/AI/AIOps/AIOps-2_4-metric_event_correlation/)
- chapter3 故障恢复


## 概览

故障发现阶段，根据监控的对象不同，可以细分为以下场景：

1. 单指标异常检测
2. 多指标异常检测
3. 故障预测
4. 指标异常关联
5. 日志聚类
6. 调用链异常检测

