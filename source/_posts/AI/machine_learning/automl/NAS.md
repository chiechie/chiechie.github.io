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
> 下面简单介绍FBNet的思路


## FBNet的基本idea

1. 定义候选block, eg10个
2. 定义神经网络的层数,eg20，神经网络的每一层都是从9个候选block中选一个，搜索空间9^20
3. 构建一个很大的神经网络--supernet，每一层由9个候选block并联而成。supernet的待估参数包括9个候选block✖️20层，以及长度为9的权重向量✖️20。
	
	> 人工调参，可以看成是，指定supernet中的一条路径，然后灌入数据，来估计路径上20个blcock的参数。使用FBNet就不用人为指定其中某条路径，而是这个大网络supernet从训练数据中，学习出9个block✖️20（层）个参数，以及9✖️20（层）个权重。训练完之后，选择每一层权重最大的block并且连成一条路径，就是机器选定的20层的网络结构。
4. 模型要落地，还要考虑计算效率。具体的做法，事先算好候选block各自的latency(做一次推断平均耗时多久，block的权重随机给，跑多组数据求平均时延），然后将latency信息加入损失函数中，作为错误率之外的另外的一个惩罚项。


## chiechie's reflection

联想到之前构建的通用异常检测模型，整体流程大概是这样：定义多个检测器，曲线特征，并用一个随机森林去做集成。希望随机森林可以将不同的曲线路由到适合的检测器上去。

FBNet的思路跟上面的流程有点像，都是数据驱动去选择适配的组件。不同的地方在于，FBNet最终会减枝，即从9✖️20个可能的路径中，裁剪成一条路径。

FBNet要解决的任务更简单，给一个小任务找到适配的解决方案；通用异常检测更难，要给n个小任务找解决方案，并且用一套复合的解决方案。


## 参考
1. [wangshusen-slide-github](https://github.com/wangshusen/DeepLearning)
2. [Differentiable Neural Architecture Search-youtube](https://www.youtube.com/watch?v=D9m9-CXw_HY)
