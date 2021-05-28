---
title: chapter1.1 单指标异常检测
author: chiechie
mathjax: true
date: 2021-05-21 16:37:13
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

# 异常检测相关问题和技术的综述

## 异常检测的难点

### 业务共性

- 历史数据有中断—使用正常数据填充
- 不同业务曲线对告警的敏感度不一样
- 新接进来业务，历史数据太少
- 新接业务没有故障

### 某些业务特有

- 整体趋势变化：比如游戏在线人数进入开学季之后会持续走低，业务觉得很正常，不要告警（一般算法都要告警）。
- 定时任务日志数：周期性凸起，业务说不要告（但是一般的算法都会告）
- 无规律指标的异常识别：常见于刚上线的大区，不稳定。
- 周期性毛刺点并且间隔不固定：数据质量问题。
- 稀疏数据：比如成交数据，商品只在某个固定时间上架。


## 异常分类

有几类异常：全局异常，条件异常

### 全局异常

- 全局异常点有点像黑天鹅事件，过去没发生过的。

![](./global_anomalie.png)


### 条件异常

条件异常(contextual or conditional anomalies)：value deviates quite a lot from the rest of the data points，举2个例子：

1. 办公室出现了一个穿西装打领带的人，跟我们格格不入，但是放到另外一个context中，比如房屋中介行业很正常
2. 凌晨销量突然升高
  
![](./img.png)


### 联合异常

![img.png](./img78.png)


每一个点不是异常的（既非 contextual 也非 global），但是一起出现就异常了，例如

1. 一个小区中，有人去医院很正常，但是整个小区的人同时去医院就不正常了。
2. 投放广告时，预算增加，曝光和点击同时上升是很正常的。但是曝光增加，点击却下降（市场营销策略失灵--glitch）就意味着有问题，可能是广告中心设置了一个空的广告位，或者将广告曝光给了错误的用户。
   ![img.png](./img0890.png)


## 用户在使用过程中的疑问

### 模型开发者

模型开发者关心的是

- 为什么告警？
	- 告警时，知道特征有什么变化，在模型输出的时候，把特征也带上
- 屏蔽周期性的告警

### 模型应用者

模型服务的使用者关心的是

- 应用时的阈值的修改记录在哪看？
- 当前模型是不是最新版本？
- 为什么告警？ 
  - 模型开发者需要在输出中加入告警描述字段
  - 可视化saas，对告警的决策逻辑进行解释，eg上下界
- 这么多模型，我该选择哪一个？
- 业务相关的需求：
  - 屏蔽周期性的告警。


[TOC]

# 时序异常检测

时序异常检测就是对时间序列数据监测出异常的模式。

但是，很难用一些策略去构建一个通用的异常检测服务，因为监控指标各异（正常模式各异），异常各异（异常类型多种）。其中，比较难识别的几种曲线和异常如下：

## 应用案例

### 1.历史数据有中断缺失

  指标：在线数据由于数据质量有中断缺失
  正常模式： 历史数据有缺失
  检测策略：先使用正常数据填充再检测。

![image-20190114173717116](../_image/image-20190114173717116.png)

### 2.整体趋势变化：趋势漂移是正常模式。
指标： 游戏收入
正常模式：游戏收入在开学季趋势漂移（正常模式很奇怪）。
检测策略：学习并且剔除这个趋势漂移。



![image-20190114173747851](../_image/image-20190114173747851.png)



### 3.指标： 定时任务日志数
正常模式：周期性有数据。（正常模式很奇怪）
检测策略：检测规律行为数据缺失

![image-20190114174212541](../_image/image-20190114174212541.png)



### 4.合理范围的突变异常。
指标：登录在线等周期性曲线
正常模式：趋势呈周期性
检测策略：检测突变点。

![image-20190114181500368](../_image/image-20190114181500368.png)



### 5.无规律指标识别。
指标：毫无规律指标
正常模式：在一定统计范围内波动
异常检测策略：相对历史分布的极端异常值。

![image-20190114181832710](../_image/image-20190114181832710.png)



### 6.周期性毛刺点-且周期间隔不明显

指标：跑批数据
正常模式：周期性毛刺点，但是周期间隔不明显，可能会有偏移（正常模式很奇怪）。
检测策略：使用高斯核函数拟合分布极值？

1. 使用dtw去衡量两个窗口的距离，兼容偏差。
2. 在历史数据中使用滚动窗口的方法，找到一个最近的窗口。

![image-20190114182211015](/Users/stellazhao/Library/Application Support/typora-user-images/image-20190114182211015.png)



### 7. 周期性跌零数据
指标：银行的业务数据
正常模式：周期性跌零，周期性有数据

![image-20190114182435743](../_image/image-20190114182435743.png)



如何获取曲线的关键特征：

1. 周期性： autocorrelation
2. 周期offset：高斯核函数拟合分布极值
3. 趋势判断：指数滑动平均
4. 分析数据极值: 假设检验。



![image-20190114190407015](../_image/image-20190114190407015.png)







一些架构调研

![image-20190703200024522](../_image/image-20190703200024522.png)



![image-20190703200138721](../_image/image-20190703200138721.png)

![image-20190703203239703](../_image/image-20190703203239703.png)

![image-20190703203352352](../_image/image-20190703203352352.png)

## 难点1：周期性陡增/降引起的误告


业务活动特性导致的周期性陡增和陡降怎么避免误告？

1. 在时间上去收敛

2. 使用特征去做模式识别（余弦相似度，比较两两之间的余弦距离），下面用的方法是用余弦相似度去刻画勾勾的形状，但是实时检测的时候，勾勾还没来要怎么做，历史数据里面的勾勾有偏移怎么办？

余弦相似度比欧式距离这种好的地方。。在于可以解决时间上出现偏移的问题。

1. ![企业微信截图_6db7bb33-160d-471b-a483-05cdc207bc43](../_image/ali周期性陡降.png)


## 总结

时序异常检测的难点在于，每一个时刻的异常情况都是context-based，但是这个context非常不好描述。

这个context需要包含的信息有：

- 用来刻画该曲线的正常模式的特征（包含是否有趋势周期性，是否有随时间波动的方差， 是否波动剧烈）----（特征选择：这些特征，来描述“异常/正常行为”

- 用来判断该时刻是否异常的参考信息。（特征计算）


## 曲线平滑对参数极为敏感


# 开发流程

![image-20190521143942651](../_image/image-20190521143942651.png)



注入的异常为


由于曲线的毛刺比较多，这里使用局部加权回归的方法。



但是该方法对参数极为敏感，

不合适的参数如下：(frac=0.0252, it=2)



![image-20190522144016678](../_image/image-20190522144016678.png)



合适的参数(frac=0.2, it=2)



![image-20190522144141723](../_image/image-20190522144141723.png)

-------







timestamp,dim1,dim2,dim3,dim4,dim5
2018-10-03 17:25:00,16,0,0,0,0



mcts跑出来的结果

{"1538558700": [[8.946669749442899, [[0, 0, 0, 0, 3]]]]}

## 参考资料
1. https://www.youtube.com/watch?v=pXGqDiE4N0I
