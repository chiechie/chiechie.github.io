---
title: 《Advances in Financial Machine Learning》读书笔记0 为什么金融领域的机器学习项目经常失败？
author: chiechie
mathjax: true
date: 2021-07-05 09:49:02
tags: 
- 量化
- 投资
categories:
- AFML
---

# 概览

1. 市场上关于投资的书籍大致分为两类：一类是理论家写的，自己都没有实践过；一类是实践家写的，他们误用了数学工具。
2. 金融市场上现在鱼龙混扎，小散受到到不良媒体的诱导会冲动投资，造成市场动荡。量化工具可以减少这种套利机会，肃清这种风气。
3. 常见的陷阱：
![img.png](img1.png)
   
   ![img.png](img.png)
4. 是否意味着有了ai就没有human投资者的空间了？不是，可以人+ai

# Pitfall 1: 西西弗斯模式


> Pitfall 1: 西西弗斯模式（THE SISYPHUS PARADIGM） 
> 
> Solution 1: 元策略模式（THE META-STRATEGY PARADIGM）

1. 自由基金经理（Discretionary portfolio managers～DPM）的投资理念比较玄学，不会遵循特定的理论，这样的一群人开会时往往漫无目的、各执一词。DPMs天然地不能组成一个队伍：让50个DPM一起工作，他们的观点会相互影响，结果是老板发了50份工资，只得到一个idea。他们也不是不能成功，关键是要让他们为同一个目标工作，但尽量别交流。
2. 很多公司采用DPM模式做量化/ML 项目：让50个PhD分别去研究策略，结果要么得到严重过拟合的结果，要么得到烂大街&低夏普率的多因子模型。即使有个别PhD研究出有效的策略，这种模式的投入产出比也极低。这便是所谓让每个员工日复一日搬石头上山的西西弗斯模式。
3. 量化是一项系统工程，包括数据、高性能计算设备、软件开发、特征研究、模拟交易系统……如果交给一个人做，无异于让一个工人造整辆车——这周他是焊接工，下周他是电工，下下周他是油漆工，尝试--->失败--->尝试--->失败，循环往复。
4. 好的做法是将项目清晰地分成子任务，分别设定衡量质量的标准，每个quant在保持全局观的同时专注一个子任务，项目才能得以稳步推进。这是所谓元策略模式（THE META-STRATEGY PARADIGM）。


# Pitfall 2: 根据回测结果做研究

> Pitfall 2: 根据回测结果做研究（RESEARCH THROUGH BACKTESTING）
>
> Solution 2: 特征重要性分析（FEATURE IMPORTANCE ANALYSIS）

1. 金融研究中很普遍的错误是在特定数据上尝试ML模型，不断调参直到得到一个比较好看的回测结果——这显然是过拟合,学术期刊往往充斥面向测试集调参。
2. 考虑一个ML任务，我们可以构建一个分类器，在交叉检验集上评估其泛化误差。假定结果很好，一个自然的问题是：哪些特征对结果的贡献最大？“好的猎人不会对猎狗捕获的猎物照单全收”，回答了这个问题，我们可以增加对提高分类器预测力的特征，减少噪声特征。
3. 需关注ml发现的跟特征相关的模式：什么特征最重要、这些特征的重要性会随时间改变么、这种改变能否被识别和预测。
4. 总之，特征驱动的分析比回测结果驱动的分析更重要。

# Pitfall 3: 按时间采样

> Pitfall 3: 按时间采样（CHRONOLOGICAL SAMPLING）
> 
> Solution 3: 交易量钟（THE VOLUME CLOCK）

1. Bars：表格数据的一行或者说一个样本，叫一个bar。
2. Time Bars:以固定的时间区间对数据进行取样（如每分钟一次）后得到的数据。
3. Time bars使用广泛，但是有两个不足：第一，市场交易信息的数量在时间上的分布并不是均匀的。开盘后的一小时内交易通常会比午休前的一小时活跃许多。因此，使用Time bars 会导致交易活跃的时间区间的欠采样，以及交易冷清的时间区间的过采样。第二，根据时间采样的序列通常呈现出较差的统计特征，包括序列相关、异方差等。
4. Tick bars 是指每隔固定的（如1000次）交易次数提取上述的变量信息。一些研究发现这一取样方法得到的数据更接近独立正态同分布 [Ane and Geman 2000]。
使用Tick Bars 还需注意异常值 (outliers) 的处理。一些交易所会在开盘和收盘时进行集中竞价，在竞价结束后以统一价格进行撮合。
5. Volume Bars & Dollar Bars：Volume Bars 是指每隔固定的成交量提取上述的变量信息。Dollar Bars 则使用了成交额。
使用 Dollar Bars更有优势的。假设一只股票在一定时间区间内股价翻倍，期初10000元可以购买的股票将会是期末10000元可购买股票手数的两倍。在股价有巨大波动的情况下，Tick Bars以及Volume Bars每天的数量都会随之有较大的波动。除此之外，增发、配股、回购等事件也会导致Tick Bars以及Volume Bars每天数量的波动

# Pitfall 4: 整数差分

> Pitfall 4: 整数差分（INTEGER DIFFERENTIATION）
> 
> Solution #4: 非整数差分（FRACTIONAL DIFFERENTIATION）

我们需要在数据平稳性和保留数据信息之间做取舍，非整数/分数差分就是一个较好的解决方案


# Pitfall 5: 固定时间范围标签

> Pitfall 5: 固定时间范围标签（FIXED-TIME HORIZON LABELING）
> 
> Solution 5: 三边界方法（THE TRIPLE-BARRIER METHOD）

1. 固定时间范围标签方法应用广泛, 但是有若干不足：time bars 的统计性质不好;常数阈值不顾波动性是不明智的;可能被强制平仓.
2. 三边界方法（THE TRIPLE-BARRIER METHOD）考虑到平仓的触发条件，是更好的处理方式，其包括上下水平边界和右边的垂直边界。水平边界需要综合考虑盈利和止损，其边界宽度是价格波动性的函数（波动大边界宽，波动小边界窄）；垂直边界考虑到建仓后 bar 的流量，如果不采用 time bars，垂直边界的宽度就不是固定的（翻译太艰难了，附上原文）
3. 如果未来价格走势先触及上边界，可以取1；先触及下边界，则取-2；先触及右边界，可以 0，或者根据盈利正负，取1或者-1 。


# Pitfall 6: 同时学出方向和规模

> Pitfall 6: 同时学出方向和规模（LEARNING SIDE AND SIZE SIMULTANEOUSLY）
> 
> Solution #6: 元标签（META-LABELING）


1. 金融中用ML的另一常见错误是同时学习仓位的方向和规模。
2. 具体而言，方向决策（买/卖）是最基本的决策，规模决策（size decision）是风险管理决策，即我们的风险承受能力有多大，以及对于方向决策有多大信心。
3. 我们没必要用一个模型处理两种决策，更好的做法是分别构建两个模型：第一个模型来做方向决策，第二个模型来预测第一个模型预测的准确度。
4. 很多ML模型表现出高精确度（precision）和低召回率（recall）。这意味着这些模型过于保守，大量交易机会被错过。
5. F1-score 综合考虑了精确度和召回率，是更好的衡量指标，元标签（META-LABELING）有助于构建高 F1-score 模型。首先（用专家知识）构建一个高召回率的基础模型。随后构建一个ML模型，用于决定我们是否应该执行基础模型给出的决策。
6. 元标签+ML有以下4个优势：
   1. 元标签+ML则是在白箱（基础模型）的基础上构建的，具有更好的可解释性；
   2. 元标签+ML减少了过拟合的可能性，即ML模型仅对交易规模决策不对交易方向决策，避免一个ML模型对全部决策进行控制；
   3. 元标签+ML的处理方式允许更复杂的策略架构，例如：当基础模型判断应该多头，用ML模型来决定多头规模；当基础模型判断应该空头，用另一个ML模型来决定空头规模；
   4. 赢小输大会得不偿失，所以单独构建ML模型对规模决策是有必要的。

> achieving high accuracy on small bets and low accuracy on large bets will ruin you

# Pitfall 7: 非IID样本加权

> Solution #7: （UNIQUENESS WEIGHTING AND SEQUENTIAL BOOTSTRAPPING）

样本不是iid的，比如按照volumn bars，到达1000的成交量才采集一个样本。

如果实际数据中，交易发生拥堵，都集中在了t=10，那么：
- t=1要到t=10才达到1000，
- t=2其实也是到t=10达到1000，
- t=3其实也是到t=10达到1000，

因此，t=10这个样本就被用了很多次（最多9次），当然time bars是没有这个问题的。

为了缓解这个样本重复出现的问题，作者定义了一个衰减因子即$1/c_t$, $c_t$表示t时刻的行情被用了多少次，所以t时刻的行情对应的return应该除以这个次数。 

这些只对那种按成交量或者其他非等时间划分样本的方法有意义
(1) labels are decided by outcomes;
(2) outcomes are decided over multiple observations;
(3) because labels overlap in time, we cannot be certain about what observed features caused an effect.

# Pitfall 8: 交叉检验集泄露信息（CROSS-VALIDATION LEAKAGE）

> Pitfall 8: 交叉检验集泄露信息（CROSS-VALIDATION LEAKAGE）
> 
> Solution #8: 清理和禁止（PURGING AND EMBARGOING）

1. 金融中需要警惕在训练集 / CV 集中引入未来信息。
2. 好的做法应该是在训练集和CV集之间设定一个间隔


# Pitfall 9: 前向回测

> Pitfall 9: 前向回测（WALK-FORWARD / HISTORICAL BACKTESTING）
> 
> Solution #9: CPCV（COMBINATORIAL PURGED CROSS-VALIDATION）

1. 常用的回测方法是前向回测（Walk-forward Backtesting）：根据当前时刻以前的数据做决策。这种方式容易解读同时也很直观，但存在几点不足：
   1. 前向回测只测试了单个场景，容易过拟合；
   2. 前向回测的结果未必能代表未来的表现。
2.作者提出了一种切分方法：将所有数据分为 N 份（注意避免信息泄露），从中任意取 k份作为测试集，剩下作为训练集，总共有很多种取法。这种方法最大的优势是允许我们得到某策略在不同时期的夏普率分布，而不是计算一个夏普率值。


# Pitfall 10: 回测过拟合

> Pitfall 10: 回测过拟合（BACKTEST OVERFITTING）
> 
> Solution 10: 保守夏普率（THE DEFLATED SHARPE RATIO）

1. 假设 ${y_i}$ 独立同分布，可证明$E\left[\max \left\{y_{i}\right\}_{i=1, \ldots, I}\right] \leq \sigma \cdot \sqrt{2 \log (I)}$。
2. 若$y_i$ 代表一系列回测结果的夏普率，则只要回测次数足够多，或者每次回测结果方差足够大，从中都能选出任意高的结果，尽管有可能 $E(y_i)=0$ 。
2. 这提醒我们要考虑到回测次数过多会造成过拟合，一种解决方案是保守夏普率（THE DEFLATED SHARPE RATIO，DSR），其思想是给定一系列对夏普率SR的估计值，通过统计检验的方法估计能否推翻零假设 SR=0 。


## 参考

1. 《Advances in Financial Machine Learning》
2. https://blog.csdn.net/weixin_38753422/article/details/100179559
3. https://zhuanlan.zhihu.com/p/69231390