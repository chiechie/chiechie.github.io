---
title: 时间序列资料汇总
author: chiechie
mathjax: true
date: 2021-04-07 11:36:11
tags:
- 时间序列
- 人工智能
categories:
- AI
---

# 算法

## 深度学习算法

- DeepAR,2017, ref 146, p12
    - [paper](https://arxiv.org/pdf/1704.04110.pdf) | [code](https://github.com/awslabs/gluon-ts/tree/master/src/gluonts/model/deepar)
- DeepFactor, 2019 ICML，ref 15,使用全局的RNN模型学习不同时序间共同的模式，以及对每一个时间序列学习一个局部模型
    - [paper](https://arxiv.org/abs/1905.12417)
- Deep State Space Models,2018
    - [paper](https://papers.nips.cc/paper/8004-deep-state-space-models-for-time-series-forecasting.pdf)
- Spline Regression RNN , Gasthaus et. al., 2019
- TCN: ref08， p14，2018
    - [paper](https://arxiv.org/pdf/1803.01271.pdf) | [code](https://github.com/locuslab/TCN)
- TFT:  ref 7次，p27，2019
  - [paper](https://arxiv.org/pdf/1912.09363.pdf) | [code](https://github.com/google-research/google-research/tree/master/tft)
- prophet, 2017,ref 370
  - [paper](https://peerj.com/preprints/3190.pdf) | [code](https://github.com/facebook/prophet)
- stock2vec
    - [paper](https://arxiv.org/abs/2010.01197)
- Forecasting: Principles and Practic 
    - https://otexts.com/fpp2/
- N-BEATS
  - [paper](http://arxiv.org/abs/1905.10437)

# 框架

- GluonTS: Probabilistic Time Series Models in Python.
    - [paper](https://arxiv.org/pdf/1906.05264.pdf)
    - [支持的算法列表](https://github.com/awslabs/gluon-ts/blob/master/REFERENCES.md)
    - [code](https://github.com/awslabs/gluon-ts) | [doc](https://gluon-ts.mxnet.io/)
- Probabilistic Demand Forecasting at Scale
    - [paper](http://www.vldb.org/pvldb/vol10/p1694-schelter.pdf)
- Criteria for classifying forecasting methods,被引用次数：21 相关文章 所有 2 个版本,2020，p11
    - [paper](https://www.sciencedirect.com/science/article/pii/S0169207019301529)
- Forecasting big time series: old and new,2019
    - [paper](https://dl.acm.org/doi/abs/10.1145/3292500.3332289)
- Time Series Forecasting With Deep Learning: A Survey 被引用2次，2020
    - [paper](/Users/stellazhao/Desktop/paper_2020时序预测综述.pdf):
- Neural forecasting: Introduction and literature overview，2020, 被引用5次
    - [paper](https://arxiv.org/abs/2004.10240)
- Forecasting big time series: old and new, 被引用次数：23 相关文章 所有 5 个版本
    - [paper](https://dl.acm.org/doi/abs/10.14778/3229863.3229878)


# 比赛


## 商品销量预测

[Walmart Store Sales Forecasting (2014)](https://www.kaggle.com/c/walmart-recruiting-store-sales-forecasting)

- Use historical markdown data to predict store sales
- 评估指标：weighted mean absolute error (WMAE)
   ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FLDPLxG_AsI.png?alt=media&token=d615d942-e378-43d5-8fc4-a26d146bb0ac)
- 样本id： 商场id+部门id+日期（包含当天，未来一周）
  ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FATkOrisXUk.png?alt=media&token=2dea8db9-d230-4266-9676-9e15bb2ed884)

## 极端天气下的商品销量预测

[Walmart Sales in Stormy Weather (2015)](https://www.kaggle.com/c/walmart-recruiting-sales-in-stormy-weather)
- Predict how sales of weather-sensitive products are affected by snow and rain
- 评估指标：Root Mean Squared Logarithmic Error (RMSLE)
  ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2F_jeBzeQgo1.png?alt=media&token=45740607-7629-4005-bb27-773c8df4e9d2)
- 样本id：商场id + 商品id + 日期
  ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FD_Ba2ZU9Nw.png?alt=media&token=09ce23f2-d4a1-425d-a23c-ee85efea58a4)

## 营业收入预测

[Rossmann Store Sales (2015)](https://www.kaggle.com/c/rossmann-store-sales)

- Forecast sales using store, promotion, and competitor data
- tuneover：营业收入
- 评估指标：rmspe
  ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FEA2frU1QNu.png?alt=media&token=e8991a1c-d50f-4390-833c-3925da591c89)
- 样本id： 商场id+日期
  ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FHbQ0SWOSpS.png?alt=media&token=a01d1b85-c213-45a4-8a5f-d52f1aaf698b)

## 肉类销量预测

[Corporación Favorita Grocery Sales Forecasting (2018)](https://www.kaggle.com/c/favorita-grocery-sales-forecasting)

- accurately predict sales for a large grocery chain?
- 评估指标： Root Mean Squared Logarithmic Error (NWRMSLE)
    - 容易腐烂（Perishable）的品类给了更高的权重1.25, 其他品类给的1.
- 样本id： 商品 id

## 餐厅人流量预测

[Recruit Restaurant Visitor Forecasting (2018)](https://www.kaggle.com/c/recruit-restaurant-visitor-forecasting):

- Predict how many future visitors a restaurant will receive
- 样本id为： 餐馆id+日期
  ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FM2vtlhu-kH.png?alt=media&token=0042496a-1f4e-4800-a982-8f2c0e6a8de7)
- 评估指标：[root mean squared logarithmic error](https://www.kaggle.com/wiki/RootMeanSquaredLogarithmicError).
  ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FkPPvo0OwMp.png?alt=media&token=fd0cfe95-5a29-46ee-8ce6-905edfcf31c5)

## 流行病人数预测

[COVID19 Global Forecasting (2020)](https://www.kaggle.com/c/covid19-global-forecasting-week-5): 

预测新冠疫情世界各地，未来15天，每天确诊人数和死亡人数:

- 输出：两个指标，3个分位数
- 样本id：城市id + 日期 + 指标id
  ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FmBk7V4JsDj.png?alt=media&token=39226fde-0ee6-4f72-bef0-8182809ae110)
- 评估指标： Weighted Pinball Loss.
  ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2F6KVBbgq7pC.png?alt=media&token=a5eb8da1-0f10-45d7-8caf-5119b3d8c46c)

# 公开数据

- [electricity数据集--Electricity dataset from UCI](https://archive.ics.uci.edu/ml/datasets/ElectricityLoadDiagrams20112014)： 
    - 370 个 家庭的用电量 from 2011 to 2014， hourly数据
    - 训练数据（单位points）：30000 points * 370  = 1110 w
        -  [from] 2011-01-01 00:00:00 [to] 2014-06-04 00:00:00
    - 测试数据（单位points）：5065 points  * 370  = 187 w  #   need additional 7 days as given info
        - [from] 2014-06-04 00:00:00 [to] 2015-01-01 01:00:00
- [realTweets](https://github.com/numenta/NAB/tree/master/data/realTweets) :twitter上的品牌（如Apple，Amazon等）的热度，粒度为5min
- [Airplane Crashes](https://data.world/data-society/airplane-crashes)
- [U.S. Air Pollution Data](https://data.world/data-society/us-air-pollution-data)
- [U.S. Chronic Disease Data](https://data.world/data-society/us-chronic-disease-data)
- [Air quality from UCI](http://archive.ics.uci.edu/ml/datasets/Air+Quality)
- [Seattle freeway traffic speed](https://github.com/zhiyongc/Seattle-Loop-Data)
- [Youth Tobacco Survey Data](https://data.world/data-society/youth-tobacco-survey-data)
- [Singapore Population](https://data.world/hxchua/populationsg)
- [Airlines Delay](https://data.world/data-society/airlines-delay)
- [Traffic dataset from UCI](https://archive.ics.uci.edu/ml/datasets/PEMS-SF)
- [City of Baltimore Crime Data](https://data.world/data-society/city-of-baltimore-crime-data)
- [Discover The Menu](https://data.world/data-society/discover-the-menu)
- [Global Climate Change Data](https://data.world/data-society/global-climate-change-data)
- [Global Health Nutrition Data](https://data.world/data-society/global-health-nutrition-data)
- [Beijing PM2.5 Data Set](https://raw.githubusercontent.com/jbrownlee/Datasets/master/pollution.csv)
- [Airline Passengers dataset](https://github.com/jbrownlee/Datasets/blob/master/airline-passengers.csv)
- [Government Finance Statistics](https://data.world/data-society/government-finance-statistics)
- [Historical Public Debt Data](https://data.world/data-society/historical-public-debt-data)
- [Kansas City Crime Data](https://data.world/data-society/kansas-city-crime-data)
- [NYC Crime Data](https://data.world/data-society/nyc-crime-data)
- [Kaggle-Web Traffic Time Series Forecasting](https://www.kaggle.com/c/web-traffic-time-series-forecasting)


# 练手项目

- [coursera-使用tensorfow构建时间序列预测模型](https://www.coursera.org/learn/tensorflow-sequences-time-series-and-prediction/ungradedLti/sFRBW/exercise-4-sunspots)
- [微软-forcasting](https://github.com/microsoft/forecasting/tree/master/fclib/fclib/models):dilated_cnn/lightgbm/multiple_linear_regression
- [微软-example](https://github.com/Azure/DeepLearningForTimeSeriesForecasting)：
    - [1_cnn_dilated.ipynb](https://github.com/Azure/DeepLearningForTimeSeriesForecasting/blob/master/1_CNN_dilated.ipynb)：值得学习的是--如何在keras中 构造空洞因果卷积？
    - RNN_encoder_decoder
- [个人项目-基于深度学习的时间序列预测](https://github.com/Alro10/deep-learning-time-series/tree/master/notebooks)
- pytorch-forecasting
    - [doc](https://pytorch-forecasting.readthedocs.io/en/latest/contribute.html) | [code](https://github.com/jdb78/pytorch-forecasting)

# 比赛 

1. [数据集](https://github.com/jbrownlee/Datasets)
2. [M5 Competition](https://mofc.unic.ac.cy/m5-competition/)
3. [M4 Competition](https://github.com/Mcompetitions/M4-methods)
4. [awesome time series--github](https://github.com/cuge1995/awesome-time-series#Datasets)

