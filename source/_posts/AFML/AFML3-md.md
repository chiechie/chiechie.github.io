---
title: 《Advances in Financial Machine Learning》读书笔记3 回测
author: chiechie
mathjax: true
date: 2021-07-07 11:08:30
tags: 
- 量化
- 投资
categories:
- 阅读
---

## chapter 10  押注大小

### 跟策略无关的押大小方法

1. 即使预测算法，预测的很对，但是头寸配置的不恰当，还是有可能亏钱。
2. 一般来说，倾向于保留部分cash，在交易信号变弱之前会增强，用于这段时间的交易。
3. 常用的方法之一是计算两个bet concurrency序列：number of concurrent short/long bets at time t，然后对long和short求差。计算bet concurrency的方法跟计算label concurrency的方法一致。
4. 第二种方法是，follow预算的方法，计算concurrent long/short bets的最大值，用类似2的方式推导bet size，最大position用于不回答道，在最后一个concurrent信号触发之前。
5. 第三种方法是应用 meta-labeling，拟合一个分类起，确定错误分类的概率，然后使用这个概率来推导bet size。

> label表示 买/卖 信号, 两个label 如果是基于相同时间段的收益率 计算出来的，就说是concurrent的.


### 基于预测概率确定BET SIZING

### 平均主动BETS

### 动态BET SIZES 和 限定价格


## chapter 11 回测的危险

## chapter 12 使用交叉验证做回测

## chapter 13 在拟合数据上做回测

1. 使用历史数据生成一个拟合数据，拟合数据的分布是从真实数据估计得到的，
2. 使用拟合数据去做回测的好处是，我们可以测试很多次，在unseen的情况下，因此可以减少得到一个过拟合策略的概率。
> 妙啊


## chapter 14 回测统计
1. 回测的三种范式：
    - 历史模拟，向前游走
    - 情景模拟：交叉验证
    - 在模拟数据上做simulation
这一章将如何评估策略好坏？
### 回测统计量的几个类型


## chapter 15 了解策略风险

## chapter 16 基于机器学习资产配置


## 参考

1. 《Advances in Financial Machine Learning》
2. https://blog.csdn.net/weixin_38753422/article/details/100179559