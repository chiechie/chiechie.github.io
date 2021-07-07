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
3. 常用的方法是，计算两个序列：number of concurrent short/long bets at time t，然后对long和short求差。计算bet concurrency的方法跟计算label concurrency的方法一致。

## chapter 11 回测的危险

## chapter 12 使用交叉验证做回测

## chapter 13 在拟合数据上做回测

## chapter 14 在模拟数据上做回测

1. 使用历史数据生成一个拟合数据，拟合数据的分布是从真实数据估计得到的，
2. 使用拟合数据去做回测的好处是，我们可以测试很多次，在unseen的情况下，因此可以减少得到一个过拟合策略的概率。
> 妙啊

## chapter 15 了解策略风险

## chapter 16 基于机器学习资产配置


## 参考

1. 《Advances in Financial Machine Learning》
2. https://blog.csdn.net/weixin_38753422/article/details/100179559