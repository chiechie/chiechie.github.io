---
title: 调用链数据的预处理
author: chiechie
mathjax: true
date: 2021-03-15 16:49:19
tags:
- 调用链
categories: AIOps
---

## 调用链的数据格式

![RPC框架下采集的调用链原始数据&AIOps比赛的子调用数据](img_1.png)

- 最顶上的表格中是最原始的数据，中间表格和下面的表格是处理后的数据
- 中间表格的是做调用链路分析（TraceAnomaly）用到的数据，用到了耗时，以及调用链的结构信息。
- 最下面表格是2021AIOps比赛的调用链的数据，每个span_id表示一次子调用，即一次trace中，从触发该trace，开始的调用路径中，途径当前服务的那次子调用，对应表2中的一个call path。但是耗时是只包括当前服务从接受到call，到完成处理，开始response的时间间隔。
- 【补充】这里的表格信息量很少，它不知道每次子调用，具体是那个应用在调用，也不知道此次子调用之前，该trace已经走过了哪些调用路径了。



## 参考资料
1. [清华微众AIOps新作:基于深度学习的调用轨迹异常检测算法
-微信公众号](https://mp.weixin.qq.com/s/sqYIb6i9z6xF5nDr8fuVsA)
2. [AIOps挑战赛2020-demo方案-chiechie](https://chiechie.github.io/2021/03/09/AIOps/aiops2021-demo/)