---
title: 投资、投机与套利
author: chiechie
mathjax: true
date: 2021-07-04 15:10:19
tags:
- 量化
- 投资
categories:
- 阅读
---

1. 巴菲特希望通过分析公司基本面来做精准判断，通过每次投资的长期持有获得收益，而西蒙斯用机器算法来发现短期胜率，通过短期胜率超过50%加上大数定律（law of large numbers）来保证长时间的盈利
2. 人在做投资决策的时候都会根据自己的经历和知识储备选取合适的策略。
3. 从交易的方式来看，投资的策略有三种形式：投资（investment）、投机（speculation）和套利（arbitrage），三种形式都可以用量化的方法来提高胜率。
4. 1949年，格雷厄姆在《聪明的投资者》里写到：“投资者与投机者最实际的区别在于他们对股市运动的态度上：投机者的兴趣主要在参与市场波动并从中谋取利润，投资者的兴趣主要在以适当的价格取得和持有适当的股票。”
5. 所谓价值投资其实是“价值和价格之差”投资，然后低买高卖。他说投资的秘诀是“不要赔钱”。
6. 在金融市场里，如果看不清30年这么长的时间线，分析交易量、交易价格和其它技术和基本面的数据，对股票未来短期内的变化，可以在日或周这样的时间线上找到机会。这样的投机行为并不关心股票未来的成长，但是只要在短期内做到足够好的判断，则可以成功地投机获利。
7. 如果把时间线再缩短，就会有套利的机会。套利背后的逻辑是低买高卖
8. 很多高频交易的策略，在市场的大量卖单和买单中找到规律，用一个较低的价格买回股票，然后找到下一个买家用高一点的价格卖出去。
9. 从投资到投机再到套利，随着交易速度的提升，超额收益会越来越高，但是这样的速度提升也有缺点，那就是随着交易速度的提升，机会的窗口就比较小，盈利的容量会越来越小。如果把股市比作赌场，随着交易速度的提升，赢钱的概率会提高，但是能赢的钱的总量却是在下降的。
10. 高频的量化投资有点像从沙子里捞金子，每捞一次在付出成本的同时都有一个概率找到金子，捞金子的收益可以从两方面得到，一是捞金子的成功率，这个可以通过优化算法加强预测准确率来得到；二是交易频率，可以通过增加单位时间的交易次数来达到。总收益大致和正确率与交易次数平方根的乘积是成正比的。这个原理叫主动管理基本定律The Fundamental Law of Active Management。
11. 机器学习做交易策略的一个误区。大家一上来就在想办法预测股价，这个思路是最直接的，从数据分析的层面看，这并没有错，很多对算法很熟悉的人都非常厉害，可以迅速的找到一些算法（例如xgboost）来做非常好的样本内预测。但是一旦在实际股市中使用的时候就会发现预测准确度远远不如历史数据所做出来的。
12. 问题不是机器学习的算法不够好，而是所有的预测模型都假设底层的市场逻辑没有变化，这样的假设是错误的，导致了过度拟合
13. 如今的市场上，找一个会用厉害的算法做预测的码农并不难，难的是管理者和投资决策者需要知道如何从最底层理解金融市场的量化思维并发挥出算法的优势。



## 参考
1. https://zhuanlan.zhihu.com/p/362383721