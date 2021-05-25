---
title:  chapter1.5 日志聚类
author: "chiechie"
header-style: text
categories: 
- AIOps
date: 2021-02-19 20:05:20
mathjax: true
tags:
- NLP
- 日志分析
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

## 做项目之前的分析

最近要做一个nlp的项目，需求就是从一堆杂乱无章的日志中，发现一些规律。
看看哪些日志其实是在说一个事情，可归为一类；哪些日志暗示着新的事情在发生，就告警。

本质是一个EDA,有点类似给用户分群。跟用户分群相比，日志分析有一些特殊的需求，比如，我们通常先关注目前发生的事情严不严重，然后才关注细节。描述前者的典型字段包括时间，日志级别，其他关键词。描述后者的典型字段包括具体ip，网站，端口号等等。


## 看看别人怎么做的

### log3C

《identifies service system problems from system logs》-log3C》【chiechie-high-level的总结】：

- 讲了通过数据挖掘-对纯文本进行分析的一种思路：原始日志-->提取事件模版--> 聚合成 时间戳 + session id的日志序列计数向量--->聚类得到典型日志序列技术向量模式---> 日志序列模式 和 系统状态（核心KPI 之间的关系）相关性分析
- 将相关性分析 应用到了 具有 物理依赖关系的 数据上面，可以得到 因果关系。
- 跟指标异常检测的思路不一样，这个不看日志中提取出来的数值指标，看的是日志间的序列关系，出现的先后顺序，更适合用来做辅助 系统的故障定位（复杂的网状调用调用链路），而前者更适合做系统的性能监控，需要人工定义感兴趣的测量方式（人工指定特征）。
- 跟日志异常检测相比，本文的应用更加的end2end。

可以了解，实际中先不考虑这个方案，复杂且黑盒，不好调效果。


### logmine
1. 


## 一些思考

1. 要不要用一些文本模型，比如word2vec，doc2vec，transformer，bert？

在当前日志的场景下，意义不大。
这些深度模型的本质是想将语义上的相似性通过embedding vector表达。
日志场景下，日志都是同一个print语句打出来的，可以看成是机器的语言，比较少有语言歧义（真的有这种近义词的，肯定不是一个组件打出来的日志），所以没必要去寻找语意上的相似性。
分词完之后，当成charactor就可以解决问题了。


## 参考资料

1. [apache_2k.log](https://github.com/logpai/logparser/blob/master/logs/Apache/Apache_2k.log)
2. [硕士论文-模式识别在海量日志分析中的应用研究  "施佳奇"](https://www.ixueshu.com/h5/document/814a23b6b51168d40153bcb23ef479f1318947a18e7f9386.html)
3. [关于logmine的改进版--chiechie's github](https://github.com/chiechie/LogRobot)
4. [关于logmine的实践--chiechie's blog](https://chiechie.github.io/2021/03/04/AIOps/logmine-notes/)
