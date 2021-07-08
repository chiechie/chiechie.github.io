---
title: chapter1 故障发现
author: chiechie
mathjax: true
date: 2021-05-07 16:03:52
tags:
- AIOps
- 异常检测
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


# 概览

故障发现阶段，根据监控的对象不同，可以细分为以下场景：

1. 单指标异常检测
2. 多指标异常检测
3. 故障预测
4. 指标异常关联
5. 日志聚类
6. 调用链异常检测： https://mp.weixin.qq.com/s/sqYIb6i9z6xF5nDr8fuVsA



# 相关资料

## 论文

- 半自动的标记工具Label-Less: A Semi-automatic Labelling Tool for KPI Anomalies
- 非监督异常检测算法-对复杂kpi使用vae进行对抗训练Unsupervised Anomaly Detection for Intricate KPIs via Adversarial Training of VAE
- 对非结构化日志数据进行异常检测-LogAnomaly: Unsupervised Detection of Sequential and Quantitative Anomalies in Unstructured Logs
- 多指标时序异常检测Robust Anomaly Detection for Multivariate Time Series through Stochastic Recurrent Neural Network
- 对周期性时间序列数据进行异常检测-Automatic and Generic Periodicity Adaptation for KPI Anomaly Detection:[paper](https://netman.aiops.org/wp-content/uploads/2019/08/08723601.pdf)
- Robust and Rapid Adaption for Concept Drift in Software System Anomaly Detection
- 无监督异常检测-基于聚类Robust and Rapid Clustering of KPIs for Large-Scale Anomaly Detection
- 无监督异常检测-基于VAE：Unsupervised Anomaly Detection via Variational Auto-Encoder for Seasonal KPIs in Web Applications
- 有监督异常检测-Opprentice: Towards Practical and Automatic Anomaly Detection Through Machine Learning:[paper](http://netman.cs.tsinghua.edu.cn/wp-content/uploads/2015/11/liu_imc15_Opprentice.pdf)
- 雅虎开源的时序预测和异常检测项目 EGAD:[论文的中文翻译](http://www.infoq.com/cn/articles/automated-time-series-anomaly-detection)
- CMU 做多指标模式提取和异常检测的 SPIRIT 系统论文：[paper](https://bitquill.net/pdf/spirit_vldb05.pdf)

## 代码

- 阿里巴巴开源的Time2Graph(基于序列片段的图迁移路径做异常检测)：[code](https://github.com/petecheng/Time2Graph)
- 清华/阿里巴巴开源的 Donut(基于 VAE 算法)：[code](https://github.com/haowen-xu/donut)
- 清华开源的 Bagel(Donut改进型，基于 CVAE 算法)：[code](https://github.com/lizeyan/Bagel)
- 清华/百度的 opprentice 系统(14 个检测器，平均取参数值，随机森林)：[code](https://github.com/tencent/metis)
- 清华/建行做的指标批量标注 Label-less 项目(基于 iForest 异常检测和 DTW 相似度学习)：[code](https://netman.aiops.org/wp-content/uploads/2019/10/Label-less-v2.pdf>)
- 百度开源的指标异常标注工具：[code](https://github.com/baidu/Curve)
- 微软开源的指标异常工具：[code](https://github.com/Microsoft/TagAnomaly)
- 清华OmniAnomaly-多指标异常检测: [code](https://github.com/NetManAIOps/OmniAnomaly/tree/master/omni_anomaly)
- 清华/南开/腾讯的 ADS 系统(ROCKA+opprentice+CPLE)：[paper](https://netman.aiops.org/wp-content/uploads/2018/12/bujiahao.pdf)|[code](https://github.com/tmadl/semisup-learn)
- 开源的时序特征值提取库 tsfresh：[code](http://tsfresh.readthedocs.io/en/latest/)
- twitter 的异常检测库，R 语言[code](https://github.com/twitter/anomalydetection)
- numenta公司，HTM 算法：[code](https://github.com/numenta/nupic)|[code-评估](https://github.com/numenta/NAB/blob/master/nab/scorer.py)|[data](https://github.com/numenta/NAB/tree/master/data)
- 雅虎开源的时序预测和异常检测项目 EGADS:[code](https://github.com/yahoo/egads)
- 亿客行Expedia开源的异常检测项目 adaptive-alerting:[code](https://github.com/ExpediaDotCom/adaptive-alerting)
- 新加坡国立大学做传感器多变量指标异常检测的开源项目(基于 GAN 算法)：[code](https://github.com/LiDan456/MAD-GANs)


## 比赛
1. 清华裴丹团队举办的 KPI 异常检测比赛：[data](http://iops.ai/competition_detail/?competition_id=5&flag=1)
2. 清华裴丹团队举办的 KPI 异常检测比赛方案-分享：[slide](http://workshop.aiops.org/)
