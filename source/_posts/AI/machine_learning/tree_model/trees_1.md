---
title: 树模型1_常用树模型介绍
author: chiechie
mathjax: true
date: 2021-05-15 14:59:29
tags:
- 人工智能
- 树模型
categories:
- AI
---

## 树模型

最基本的树模型是cart，优势在于可解释性，但是跟其他的监督学习方法相比，准确性并没有多大的优势。
但是，通过将多棵树以不同的方式组合，会得到很好的准确率，如bagging，random foreast和boosting等等方法。

回一下机器学习的三个要素：
1. 假设函数的空间： 
2. 学习策略or目标函数
3. 优化算法

## 决策树

## random forest


- bootstrap是统计学中一种抽样的方法，为了获得样本方差或者均值，采取多次有放回的抽样，获得统计量。
- bagging是利用bootstrap的思想，但是不是为了获得统计量，而是通过多次实验获得多个regression tree或者classification tree，对每一个input，会产生多个输出，regression tree的输出是K个树的输出取average。classification tree是采用投票的方法，大多数相同的那一类就是输出的那一类 。


random forest是bagging tree的加强版本，因为他考虑到了tree之间的de correlates，使得组合后结果的variance更小。

构造树：
consider 每一个split的时候，会从p个predictors中**随机**的挑$m = \sqrt p$（不一定要这么多）个作为split candidate，


##  xgboost

xgboost是一种boosting tree的一种，其他的还是有gbdt，catboost

## 参考资料
1. [youtube-gbdt](https://www.youtube.com/watch?v=2xudPOBz-vs)
2. [xgboost](https://arxiv.org/pdf/1603.02754.pdf)