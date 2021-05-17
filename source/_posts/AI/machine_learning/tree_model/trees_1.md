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

bootstrap是统计学中一种抽样的方法，为了获得样本方差或者均值，采取多次有放回的抽样，获得统计量。
针对单个决策树方差过大的问题。
bagging trees 利用bootstrap的思想，提取B份独立的样本集，分别估计出预测值$\hat{f}^{1}(x), \hat{f}^{2}(x), \ldots, \hat{f}^{B}(x)$ ，然后将结果取平均，就可以得到一个方差更小的预测模型，通过多次实验获得多个regression tree或者classification tree，对每一个input，会产生多个输出，regression tree的输出是K个树的输出取average。classification tree是采用投票的方法，大多数相同的那一类就是输出的那一类 。
bagging involves creating multiple copies of the original training data set using the bootstrap, fitting a separate decision tree to each copy, and then combining all of the trees in order to create a single predictive model.

random forest是bagging tree的加强版本，因为他考虑到了tree之间的de correlates，使得组合后结果的variance更小。构造树过程如下：consider 每一个split的时候，会从p个predictors中**随机**的挑$m = \sqrt p$）个作为split candidate，

boosting是另外一种提升决策树效果的方法，Boosting works in a similar way,也是杨原始样本复制n份，然后构建n个模型，跟bagging tree不一样的地方在于，trees are grown sequentially: each tree is grown using information from previously grown trees. Boosting does not involve bootstrap sampling; instead each tree is fit on a modified version of the original data set.

![img.png](trees_1/img.png)
xgboost是一种boosting tree的一种，其他的还是有gbdt，catboost

## 参考资料
1. [youtube-gbdt](https://www.youtube.com/watch?v=2xudPOBz-vs)
2. [xgboost](https://arxiv.org/pdf/1603.02754.pdf)