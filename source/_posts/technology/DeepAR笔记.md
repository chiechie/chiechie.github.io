---
title: DeepAR笔记
author: chiechie
mathjax: true
date: 2021-04-14 17:21:19
tags:
- 人工智能
- 时间序列
- 预测
- 神经网络
- DeepAR
categories:
- 技术
---


## 一些疑惑

- 时序预测的技术难点在哪里？
- 模型的输入是来自多个序列（有点点像类似迁移学习中的样本的横向扩展，以及持续学习中灌多个样本：
    - 为了保证模型不会出现灾难性遗忘掉先学到的模式，需要把来自不同的曲线的样本打散？
    - 如果说序列之间的模式本身就不一致，会不会导致模型学习到一个混乱的模式？
    - 要不要求序列本身就是相似的？
    - 不同scale的曲线如何share一个模型 ？
    - 1个deepAR 模型可以容纳的最大的曲线条数是多少？每个曲线的样本个数是多少？
- deepAR不适用的场景有哪些？
- encoder和decoder的权重是共享的么？
- 预测target为count类型或者为正的数值类型，模型是怎么做的？


## 论文重点结论

## 相关技术综述

- classic时序预测： arima/state space/exponential smoothing
    - 只对单个曲线做预测，缺点是要一个个曲线去做模型选择，并且每个样本曲线需要积累大量的历史数据
- 基于神经网络：
    - 单个曲线训练
    - 分组训练：发现即使分了组，每个组里面的曲线差异也很大
    - 全局训练：DeepAR是第一个
- 【支线】其他领域：nlp，图像生成，语音生成技术 都可以用于时间序列，但是要注意有两个不同点：
    - 时序场景中，下游的决策应用依赖输出概率分布
    - 时序场景中，输出为count类型，使用负的二项分布计算likelihood，这个时候不能使用标准的归一化方法，参考
        - 归一化层，目前主要有这几个方法：Batch Normalization（2015年）、Layer Normalization（2016年）、Instance Normalization（2017年）、Group Normalization（2018年）、Switchable Normalization（2018年）；
- 技术难点：
    - 现实中需要对多个曲线建模，分别建模人工成本高
    - 不同曲线scale不一样，但是学习一个global model对nn来说挑战很大（nn擅长局部建模）
    - 预测概率分布

## 总结
- deepAR是一种时间序列预测算法.
- deepAR不做点估计，而是估计概率分布
- 由于deepAR输出的是概率分布，所以对未来一段时间的预测，需要采样递归生成，但是采样只是1个路径，如果希望得到期望，需要重复采样
- 适用于噪声较大的数据：相较于点估计能提供更有用的信息，如方差，在金融领域这个值代表风险，价值的(比如var用于计算准备金)。
- 在交易上的启发：构建基于波动率的策略，风险大的情况下不交易。

## deepAR原理

基本概念&符号说明：

- $z_{i,t}$:  第i个时间序列在t时刻的值
- $P\left(\mathbf{z}_{i, t_{0}: T} \mid \mathbf{z}_{i, 1: t_{0}-1}, \mathbf{x}_{i, 1: T}\right)$: 模型输出
- conditioning range： $t_0$表示历史依赖长度
- prediction range：预测区间
- $\mathbf{Z}_{i, t_{0}: T}$: 预测区间内，Z的取值
- model distribution:
- likelihood factors:
- 自回归循环神经网络
- ancestral
- 模型架构：训练过程（左边）和 预测过程（右边）：
	![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FjTfoJccA7q.png?alt=media&token=6850c357-b303-4053-81b8-2756678deb58)
- 学习目标： 最大化likelyhood:
     $$\mathcal{L}=-\sum\limits_{i=1}^{N} \sum\limits_{t=t_{0}}^{T} \log \ell\left(z_{i, t} \mid \theta\left(\mathbf{h}_{i, t}\right)\right)$$
- 【重点】对不同scale曲线怎么处理的？
    - 1.  不同曲线怎么做归一化和逆归一化？缺失值多&方差变化大
        - context-length内的所有z求平均值，作为scale来归一化原始输入并且记下来，模型输出的u 和 sigma 再 使用这个scale来逆归一化回去。
        - 相当于在第一层做了layer Normalization
    - 2. 不同曲线的长度和重要性不一样，如果是按照曲线id来均匀采样，会导致重要的曲线学的不够（underfit）
        - 不做均匀采样，而是做加权采样--使用1里面计算得到的scale作为权重
- 【重点】deepAR是怎么得到概率分布的？
    - 比较intutive的推导--updata0926
        - 我们希望得到条件概率分布，即p（z｜x），
        - Q 如何得到概率分布呢？
        - A ：分两步走
            - 先拍一个概率分布的形式（函数族），eg高斯分布
            - 使用量化指标$$< \mu, \sigma> $$进一步确定这个分布
        - Q  剩下的问题（不确定的因素）为，如何确定量化指标？
        - A ：构建一个神经网络，输入x，输出 $$< \mu, \sigma> $$
        - Q 那么nn怎么知道输出的u，sigma对不对呢，又没有监督信号 来指引，只有<x, y> pair
        - A： 使用likelihood 和 y 来度量输出的$$< \mu, \sigma> $$质量好不好
            - 有点像，有个人号称 他的理论知识渊博<u，sigma>(覆盖了y空间中所有的案例），现在要检验他的方法论对不对呢？只有一个实践案例----- 假设是对的，然后看是否适用该案例
        - 总结下：
            - 模型输出<u, sigma>, 实际给的ground truth：y
            - loss为：$$- log Prob(\mu，\sigma，y） $$
    - 前提，先假设预测的数据服从的分布族，有几个待估参数
        - 实值--假设是高斯分布
        - 整数--假设是一个二项分布
    - 设计网络的时候，需要某一层输出是 分布的参数（例如$$\mu$$ 和 $$\sigma$$）,
        - 训练的时候，将$$< \mu, \sigma> $$ 带入pdf，计算pdf中 ground truth的概率，取negtive log，作为反馈信号（-log prob就是loss），这个值越大（loss越小）说明模型预测的越准。
        - 做推断的时候，将$$< \mu, \sigma> $$ 带入pdf， 进行采样，得到预测值
- 【重点】输出的概率分布，怎么评估模型效果？
