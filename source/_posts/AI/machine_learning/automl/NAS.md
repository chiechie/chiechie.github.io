---
title: 可微的NAS方法
author: chiechie
mathjax: true
date: 2021-05-18 08:32:24
tags:
- auto ml
- 人工智能
categories:
- AI
---

> NAS是auto-ml的一个子领域，相关的技术有随机搜索，强化学习，基于微分的方法，如DARTS 和 FBNet。
> 
> DARTS 和 FBNet是目前做NAS的the state of arts方法
>
> 下面简单介绍一些FBNet的思路



## FBNet的基本idea



1. 定义9个候选block
2. 定义神经网络的层数,eg20，神经网络的每一层都是从9个候选block中选一个，搜索空间9^20
3. 构建一个很大的神经网络--supernet是一个很大的神经网络，每一层由9个候选block并联而成。supernet的待估参数包括9个候选block的参数*20，以及每一层的一个权重向量9*20。人工调参，可以看成是，人工指定其中一条路径，然后灌入数据，来估计指定路径上20个blcock的参数。现在使用FBNet，是人不用指定其中的路径，而是直接灌入数据到这个大的神经网络supernet中，估计出9*20个block的参数，以及9*20个权重。最后，在每一层，选择权重最大的那个block作为机器跳出来的最优超参。
4. 模型为了要落地，还要考虑计算效率。具体地就是事先算法9个block的latency（预测时长，权重随机给，跑多组数据求平均时间延迟），然后将这个信息加入损失函数中，作为另外一个惩罚项。



## chiechie's reflection

联想到我之前构建通用异常检测模型，整体流程大概是这样，定义多个检测器，以及曲线特征，并且用一个随机森林去做集成。希望随机森林可以将不同的曲线智能地路由到适合的检测器上去。

FBNet的思路跟上面的流程有点像，都是数据驱动去选择适配的组件，但是不同的地方在于，FBNet最终会剪纸，也就是说从9*20个组合中，只选择一条。并且，FBNet要解决的任务也更简单，给一个小任务找到适配的解决方案；异常检测有点像，先找到n个小任务的解决方案，并且统一成一个复合方案。


## 参考
1. [wangshusen-slide-github](https://github.com/wangshusen/DeepLearning)
2. [Differentiable Neural Architecture Search-youtube](https://www.youtube.com/watch?v=D9m9-CXw_HY)
