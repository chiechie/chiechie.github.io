---
title: 树模型3_GBDT介绍
author: chiechie
mathjax: true
date: 2021-05-15 10:39:45
tags:
- 机器学习
- 树模型
- 人工智能
categories:
- 技术
---

> boosting tree的代表是gbdt，以及一系列变形xgboost,catboost,下面重点介绍一下xgboost

## high-level理解

1. GBDT是一种data-driven的预测方法。
2. GBDT的预测逻辑，分两个层次，第一层，保证每一个样本的准确率；第二层，构造决策路径，即希望通过特征来达到第一层的准确率。实现第一层目的的方法是基于梯度确定阶段性目标；实现第二层的目的的方法通过特征去拟合阶段性目标（残差），也就是cart。


### Adaboost, GBDT 与 XGBoost

1. Adaboost是GBDT的一个特例
2. XGB在GBDT的基础上引入了二阶导数，并且在loss中加入了正则项。


## detail

![adaboost](./img.png)

### 公式推导


### 代码实现


## 参考资料
1. [youtube-gbdt](https://www.youtube.com/watch?v=2xudPOBz-vs)
2. [xgboost](https://arxiv.org/pdf/1603.02754.pdf)