---
title: chapter1.2 故障预测
author: chiechie
date: 2021-05-21 11:01:18
categories: 
- AIOps
mathjax: true
tags:
- NLP
- AIOps
- 日志分析
- 论文笔记 
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

> 硬盘故障预测实战

# 背景

最近10年，随着服务器数量快速增长, 服务器老化的问题（商家超过5）日趋严重。越老的服务器发生故障的概率越高，这个会带了不小的负面影响，从而提高运营成本。

服务器的所有故障中，占比最高的类型属于磁盘故障，达到70%。第二名是主板故障。
因为磁盘生命周期短，一般只有3到5年，而整个服务器使用和年限一般都会超过5年，所以一台服务器的生命周期中几乎必然会发生磁盘故障。

如何降低磁盘故障带来的影响？业务的方案是，实施监控以及事后自动修复，修复不了就更换磁盘，并且做业务迁移。

但是这个时候损失已经发生，如果能够在磁盘故障之前，就预测到即将到来的故障，并且提前做预案，是可以避免业务损失的。

下面就介绍一下，如何预测磁盘故障，从而减少业务损失。

 
# 方案

##  搜集数据

范围：即将故障的硬盘以及其宿主的信息
根据经验可以提取如下指标：磁盘负载，磁盘I/O，具体来说

- 每秒磁盘读写块数： 902-读，903-写
- 每秒中用于I/O操作的CPU时间占的比例：即999，

其中，I/O指标差分，简称为IO跳变跟故障相关度较高

此外考虑了磁盘SMART（Self-Monitoring Analysis and Reporting Technology——自我检测、分析及报告技术）。之所以考虑用SMART作为核心数据，因其相关技术发展比较成熟，至今20年了，业界对其认可度较高。


##  预测模型

使用一个分类器（SVM），搜集历史故障样本和非故障样本，


## 总结

1. 现已完成希捷SATA硬盘预测，各型号预测正确率和故障覆盖率如表3所示。其中前六种型号达到运营要求，已进入推广，具体覆盖65%的SATA盘
2. 关于正样本数据的搜集：刚开始我们把预测单与实际故障单作对比时，得到的准确率很低。因为模型报出来的预故障单都是从SMART上看，属于“亚健康”甚至“病危”状态的硬盘。而实际故障单是以磁盘自检失败并且不可在线修复和系统dmesg信息中的错误关键字（主要包括SCSI设备掉线，ATA设备超时和设备故障）为准发起故障处理流程，并且结单故障类型为硬盘故障，也就是说一定要有实际换盘的，这个定义其实是相当严苛的，会漏掉很多情况。因为结单故障类型是现场去确认后填写的，如果没有换盘或者没有填写结单故障类型就会算是我们的误报。 并且，实际故障单更多的侧重于依赖OS层面的判断，一定程度漏掉了一些健康堪忧但并未报障的硬盘。我们把这部分「误报」的部分提供给厂商确认，的确被确诊为病态盘甚至于高危盘。所以是否故障的判断，不能仅依赖结单故障类型，还要关注单块硬盘自身的健康状态--如果它的SMART核心指标远离安全阈值，也应判断为故障。
3. 下一步工作-现在核心主要侧重于SATA盘的故障预测，后续二期我们会把SAS、SSD作为核心，希望能有更大的作为。
   


# 参考
1. [部件故障预测初论](http://km.oa.com/group/23255/articles/show/251071)
2. [磁盘IO与磁盘故障率研究](https://km.woa.com/group/11783/articles/show/170959?sessionKey=Q3i7YOn2ZcLaDSNohaOIZgxO5ztyEuU8)
