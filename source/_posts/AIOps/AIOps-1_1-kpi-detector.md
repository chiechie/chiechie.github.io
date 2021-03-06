---
title: chapter1.1 单指标异常检测
author: chiechie
mathjax: true
date: 2021-05-08 16:37:13
tags:
- AIOps
- 异常检测
categories: 
- AIOps
---



# 异常检测背景

运维场景的指标监控就是对时序数据做异常检测。但是，很难通过构建一个异常检测策略一劳永逸搞定所有监控需求。因为异常各异，监控指标各异（正常模式各异）。

先看一下异常有哪些。

## 异常分类

有几类异常：全局异常，条件异常，联合异常


- 全局异常点：单个点跟历史所有的数据点比，差异很大，有点像黑天鹅事件。
	![](./global_anomalie.png)
- 条件异常： 条件异常(contextual or conditional anomalies)：即相对异常，跟剩下的数据点比比较异常。举个例子：
  
	1. 我们办公室出现了一个穿西装打领带的人，会很引人注目。但是放到另外一个context中，比如房屋中介行业很正常。
	2. 凌晨销量突然升高
	![](img.png)
- 联合异常：单个点不异常，但是群体表现出某种特殊模式，就可疑了，例如，一个小区中，有人去医院很正常，但是整个小区的人同时去医院就不正常了。
![img.png](./img78.png)
  

# 异常检测方案


时序异常检测原理，由于异常的判断依据是context，所以如何表达context信息是一个重点：

## 曲线分类 

如何构建曲线类型特征？

1. 周期性： autocorrelation
2. 周期offset：高斯核函数拟合分布极值
3. 趋势判断：指数滑动平均
4. 极值分析方法: 假设检验。
	

## 异常检测算法


- 基于预测（sequence based）
    - ARIMA/STL/Holt-winters
    - Regression
        - SVM
        - Random Forest
        - Neural Network
            - DNN
            - Recurrent Neural Networks (RNN/LSTM)
- 无监督（iid）
    - Isolation Forest
    - One-class SVM
    - LOF
    - PCA
    - Autoencoder
    - vae
- 统计分布
    - N-sigma
    - Grubb TEST
    - ESD TEST
    - t-test
- 有监督-分类
    - LR
    - xgboost（增量学习）
    - random forest
    - svm
    - CNN
	
## 根据不同业务适配不同的检测策略

针对不同的业务需求，构建检测策略。

一般来说，突变点检测+上升下降屏蔽+时间收敛的策略已经可以覆盖80%的检测指标了。

对于特别重要的指标-业务核心KPI，还是有必要手动配置的。


| 案例       | 指标              | 策略                               |
|------------|-----------------|----------------------------------|
| 历史数据有中断    | 在线数据或者其他数据，入库失败 | 先使用基于插值的方法填充后再检测。                    |
| 趋势漂移是正常模式  | 游戏收入在开学季趋势漂移[1]    | 学习并且剔除这个趋势漂移，检测残差。               |
| 周期性有数据     | 定时任务日志数；礼包成交数量[2]    | 检测规律行为数据缺失                       |
| 合理范围的突变异常| 登录在线等周期性曲线      | 检测突变点                      |
| 历史数据无规律  |  刚上线的游戏和测试服               | 相对历史分布的极端异常值。                    |
| 周期性毛刺点   | 由于数据质量导致的监控指标周期性毛刺，并且间隔不固定           | 先剔除极值再检测           |
| 周期性陡增/降[3]     | 业务活动特性导致        | 1.按时间收敛；2. 模式识别[4]:         |
|无历史数据| 新接业务 ||
|历史数据无异常| 新接业务 |无监督|
|不同业务曲线对告警的敏感度不一样|在线和cpu|敏感度|


- [1]游戏收入在开学季趋势漂移: 进入开学季之后会，游戏在线人数持续走低，业务觉得正常，但是一般算法会告警
- [2]礼包成交数据，礼包只在周末某个固定时间段上架。
- [3]周期性凸起：定时启动任务导致日志数指标定时凸起，业务觉得正常，但是一般算法会告警
- [4]模式识别： 1. 使用dtw衡量两个窗口的距离，兼容偏差； 2 在历史数据中使用滚动窗口的方法，找到一个最近的窗口。

# 产品运营

## 1. 如何让用户认可告警？

- 解释异常
- 解释正常是什么样---->上下界
  

## 2. 怎么解决标记成本高，然而又想针对自己的业务定制化？

- 方案1--人工注异常，可行吗？
	- 前期： 采用自监督的方法训练一个rnn，注入异常的方法来评估；提供公共样本集，用户调整模型 + auto-ml。
	- 后期：标记样本积累足够多，用有监督方法来训练。
- 方案2--使用小样本学习，可行吗？
	- 用户标记小样本
- 方案3-使用无监督 + 有监督调超参的方式，
	- 前者是 大量样本（训练集），后者只用 少量的样本（调参集）

# 一些未经验证的自己的思考

## 1. 选无监督 or 有监督算法？

- 有监督算法的优势：
	- 如果原始的样本池子 包含 全部模式的样本，那么 很轻易在训练数据和验证集上达到一个很高的准确率。
	- 如果原始的样本池子 只包含部分模式的样本，
		- 线上数据的模式没有出现在样本池子中，这个模型对线上样本来说就是过拟合，效果不好，可以继续加样本，让模型去学习。但是如果 没法源源不断地 低成本地 获取到新标记样本，那么这个有监督算法就没用了。
- 无监督（或者自监督）的优势？
	- 有监督算法 虽然 轻易可以达到一个很高的准确率，但是往往也**很容易过度拟合样本数据**，导致只要应用数据的模式跟 样本集模式(<x, y>的联合分布)有些许区别，就会差之毫里，谬以千里。
	- 无监督的优势就是，就算过拟合（只可能<x >的分布了），也没关系，因为样本多的是再把线上样本加进来就好了
- 无监督（或者自监督） 是否能 完全替代 有监督算法？
	- 除了用作评估的那部分样本，目前觉得可以诶。
- 为什么要用无监督？
	- 异常是无法枚举or获取成本很高，那么我想尝试 使用反向方法，把注意力放到正常上面来。（无监督的方法 都是要基于 正常的方法，当然为了适当的放宽弹性，会容忍一些噪声，比如高斯噪声）

## 2. 为什么推荐系统这种有监督的场景能够落地？

推荐场景可以**零成本地** **持续不断的获取到新的标记样本**。

## 3. 如果要落地一个有监督的算法，需要具备什么条件？

- 持续不断地，低成本（包含零成本）地 获取到新的标记样本。

## 4.  关于曲线分类+异常检测

通用的异常检测方案中，曲线分类 是不是必须的？ 或者说有没有什么场景，只用预测搞不定呢？

- 用户1：
	- 光滑的有周期性的KPI--使用预测算法；
	- 成功率类型的曲线--使用多指标异常检测。
	- 将可以搞定的曲线 和 模式混乱的 曲线分开。
- chiechie：分很粗糙的大类

## 5. 曲线分类的可预见的坑 和 已经踩过的坑？

- 已经踩过的坑
	- 开发过程痛苦，按着葫芦起了瓢
	- 实际上，很多曲线都落入了无法识别那一类，因为说不清它的正常模式是什么（btw，这部分曲线只能通过3-sigma这样的策略粗暴解决，那么曲线分类的价值在哪里？）
- 可预见的坑：
	- 流程太长，曲线分类和异常检测一定是配套的，不好拓展和维护
	- 风险暴露因子太多了，这两步只要1个有问题就整体都会有问题
	- 工程化风险：搞一个复杂度高的模型，对调试和定位问题的压力会很大。

## 6. 关于分组训练

- 使用生成模型（时序预测），需要对 正常模式相近 的曲线 聚成一类，比如某个类型或者某个品牌的商品放到一起训练，
- 使用判别模型（有监督异常检测）， 需要把 异常模型 相近的 曲线聚成一类，比如异常判断规则一致的曲线。

	
## 7. 3-sigma的局限性是什么？

- 相当于从时空二维空间 降维 到 空间这么一个空间： 把时间轴上不同时刻对应的观测值，全部挤到了t=0时刻，得到了一个bin，可以得到value的分布。
- 二维空间的连续性捕捉不到： 在二维空间来看，样本之间是有 更复杂的 依赖模式的，也就是，x轴（时间轴）上相近的 点，在y轴（space轴）也会更相近，并且一般来说，这种相近的模式  是 顺着x轴 具有延续性。（

## 8. 做多指标异常检测能不能替代多个单指标异常检测？

不行，举个例子：

单从体重这个维度来描述人的肥胖程度，没法区分胖子和瘦子，因为胖瘦还跟身高有关。体重超过200kg肯定是个胖子，但是如果体重200kg以内的话，还要看另外一个指标就是身高（类似这里的时间），要知道跟他身高差不多的同伴的 体重的范围，才能知道正常不正常。
	
## 9. 深度模型 是否能 完全替代 传统的机器学习模型 ？

不管是ml or dl本质都是用样本数据来拟合出一个函数，至于这个函数族是  简单的好 还是 复杂的好，取决的具体的应用数据。