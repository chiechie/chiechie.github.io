---
title: 时序预测最佳实践
author: chiechie
mathjax: true
date: 2021-06-05 00:06:21
tags: 
- 最佳实践
- 人工智能
- 时序预测
- 量化交易
categories: 
- 实践
---

# 建模pipeline

1. 搜集原始数据: 过去十年的成交数据
2. 数据筛选: 选择交易日，固定时间段（９:30~15:30)的数据
3. 对选好的数据进行采样，并不能保证每天都有整数个点
4. 切分训练集 & 验证集
5. 模型训练
    - 归一化训练集，<x， target>
    - 训练
6. 模型评估
    - 归一化验证集
    - 预测&画图：


实践过程中遇到的一些问题，需要逐一解决，下面记录其中一些

# 问题集合

    
## 问题1. 验证集的mse比训练集低，但相关系数训练集远远大于调参集，这是bug？

没有bug，mse和correlation评估的侧重点不一样。

mse小，但是correlation也小的例子，如下图：

![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FNUSWvXNmWb.png?alt=media&token=8c03a9f3-60e1-424d-879e-3371c1516623)

几个距离：

- 欧式距离（mse）量化的 是两个实体（或者两个群体）之间的绝对物理距离
- 余弦距离（correlation）量化的是 两个实体（或者两个群体）相对原点的角度的距离
- pearson衡量的是 两个变量 之间的 线性相关性，具体做法是使用一个直线去拟合多组样本点<变量1，变量2> ，直线斜率就是相关性大小。

## 问题3：怎么解决分布漂移的问题？

分布漂移在时序预测中尤其常见，使用滑动窗口的均值来 做中心化

## 问题4：还有哪些措能提升实验效果？

取效果好和效果差的两组实验进行对比，观察到1个现象：使用短期趋势中心化后，在使用长期趋势归一化，能使效果变好，背后的理论依据是什么？

分析下：因为用到了更长的历史数据作为参考,本身模型是记不住那么长的历史信息的。（输入才半个小时）

![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2Fg6FAdLzkOj.png?alt=media&token=362ed41e-1773-4ff5-ac41-a3095f75bb86)

参考: [doc](https://github.com/Arturus/kaggle-web-traffic/blob/master/how_it_works.md)

## 问题5： 过拟合的原因

训练集 的比例远大于 验证集 和测试集

## 问题6： 是不是预测的准就能挣钱了？

不是，回测的时候要考虑滑点，交易费用，爆仓风险

## 问题7：同样一个预测模型（LSTM），商品期货准确率比股指差这么多？

- 因为1min的股指期货数据平滑，噪声少，而1min的商品期货毛刺非常多
- 将1min的商品期货聚合成5min之后，再使用过去1h的数据预测未来1h的数据，准确率非常之高

## 问题8：直接用回归模型预测价格，可能有负数怎么处理

预测增长率

## 问题9：除了直接回归，还有哪些方法？

转化为分类问题，换预测涨跌幅

