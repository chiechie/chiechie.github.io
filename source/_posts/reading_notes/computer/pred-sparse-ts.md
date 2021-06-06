---
title: 对稀疏序列进行预测
author: chiechie
mathjax: true
date: 2021-04-28 09:33:02
tags:
- 时间序列
categories: 
- 阅读
---


## 背景和问题描述

- 我有一个time-series dataset (daily frequency) ，代表某个顾客购买某个商品的数量，如下:
    - [0,0,0,0,24,0,0,0,0,0,0,0,4,0,0,0,0,17,0,0,0,0,9,0,...]
- 每个数据表示 某一天商品的销量
- 已有的算法如arima，holtwinter只能对连续型 和 光滑型数据做预测，不适合现在的稀疏数据。
- 总结下，target 非负&稀疏&整数，有什么技术可以做预测吗？
    
## 方案1:将天的数据聚合成周：

if you don't need daily resolution in your predictions, perhaps by aggregating to weekly you will have an easier time series to forecast.

## 方案2:对两个事件的时间间隔 和 数值 进行建模
    - 假设数据是由两个随机过程驱动生成
        - 1  a distribution over time intervals, （poisson？）
        - 2 a distribution over purchase amounts. （gaussian？）

## 方案3:对用户的库存建模

假设用户的库存是随事件衰减的，到达某个阈值就开始考虑购入，那么可以用手头的购买数据来拟合已经有的曲线（注意 消耗速率和threshold都有可能随时间变化）

## 方案4-建模成一个state-space模型，state是时间间隔，space是取值

![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FbO8qrMgQNa.png?alt=media&token=4078557f-5b4f-4b43-8fed-1eed975a0d80)

## chiechie's refomulation

突然想到，事件序列可以和 时间序列 重新定义到 一个框架下建模：
    - 事件序列 相当于 是中间有缺失值 或者 零值的 时间序列，
    - 一种建模方式是，按照滑动事件窗口的方式，将最近w个事件放入一个样本，特征除了event的值，还有相邻两个event的时间间隔
    - 还有一种建模方式，缺失时刻补0，然后重新聚合，然后取时间窗口，
    - 两种方式都是把序列数据 转换为 表数据。

## 先转换-->cusum, 不仅适合sparse信号，还适合固定值的信号：
- You might be getting caught up on the sparsity characteristic... If you take a cumulative sum then it's just a plain old monotonic time series!
- https://www.reddit.com/r/datascience/comments/a63aq9/sparse_time_series_prediction/


## 参考
1.[ts_prediction-stackexchange](https://datascience.stackexchange.com/questions/17341/forecasting-non-negative-sparse-time-series-data)
