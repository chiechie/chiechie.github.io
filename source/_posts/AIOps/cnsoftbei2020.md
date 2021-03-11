---
title: 仅靠指标做根因分析
author: chiechie
mathjax: true
date: 2021-03-10 14:37:13
tags:
- AIOps
- rca
categories: AIOps
---

> 数据来自CN-Softbei2020比赛--《基于网络拓扑及告警的故障根因定位系统实现及算法研究》
>
> 很激进的做法，当涉及的指标过多时, 得到的拓扑图非常复杂.


## 数据解析


![dataoverview](dataoverview.png)

将每个事件看成一个指标的话，可以得到一个节点为2876的图


![img.png](img.png)

对应的 因果图巨大无比，如下：

![图1-关联拓扑](correlation_topology.png)






## 参考
1. [CN-Softbei2020比赛](http://www.cnsoftbei.com/plus/view.php?aid=479)
2. [chiechie的代码-github](https://github.com/chiechie/Insighter/blob/master/aiops_CN-Softbei_2020.ipynb)