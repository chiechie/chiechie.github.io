---
title: 《Advances in Financial Machine Learning》读书笔记0 为什么金融领域的机器学习项目经常失败？
author: chiechie
mathjax: true
date: 2021-07-08 09:49:02
tags:
categories:
---



1. 市场上关于投资的书籍大致分为两类：一类是理论家写的，自己都没有实践过；一类是实践家写的，他们误用了数学工具。
2. 金融市场上现在鱼龙混扎，很多小散户收到不良媒体的诱导会冲动投资，结果造成市场动荡。量化工具可以肃清这种风气，减少这种套利机会。
3. 常见的陷阱：
![img.png](img1.png)
   
   ![img.png](img.png)
4. 是否意味着有了ai就没有human投资者的空间了？不是，可以人+ai

# Pitfall 1: 西西弗斯模式（THE SISYPHUS PARADIGM）

1. 西西弗斯范式（Sisyphus Paradigm）：大概是说请了一堆投资人，但是有用的策略很少，导致入不敷出？
2. 元策略范式（Meta-Strategy Paradigm）：构建一个成功的策略和构建100个成功的策略话费的心思一样多。构建一个团队，让成员各司其职，成功率要远大于单兵作战。

Solution #1: 元策略模式（THE META-STRATEGY PARADIGM）

作者首先讨论了自由基金经理（Discretionary portfolio managers～DPM），他们的投资理念比较玄学，不会遵循特定的理论，这样的一群人开会时往往漫无目的、各执一词。DPMs天然地不能组成一个队伍：让50个DPM一起工作，他们的观点会相互影响，结果是老板发了50份工资，只得到一个idea。他们也不是不能成功，关键是要让他们为同一个目标工作，但尽量别交流。

很多公司采用自由基金经理模式做量化/ML 项目：让50个PhD分别去研究策略，结果要么得到严重过拟合的结果，要么得到烂大街&低夏普率的多因子模型。即使有个别PhD研究出有效的策略，这种模式的投入产出比也极低。这便是所谓让每个员工日复一日搬石头上山的西西弗斯模式（THE SISYPHUS PARADIGM）。

做量化是一项系统工程，包括数据、高性能计算设备、软件开发、特征研究、模拟交易系统……如果交给一个人做，无异于让一个工人造整辆车——这周他是焊接工，下周他是电工，下下周他是油漆工，尝试--->失败--->尝试--->失败，循环往复。

好的做法是将项目清晰地分成子任务，分别设定衡量质量的标准，每个quant在保持全局观的同时专注一个子任务，项目才能得以稳步推进。这是所谓元策略模式（THE META-STRATEGY PARADIGM）。


# Pitfall 2: 根据回测结果做研究（RESEARCH THROUGH BACKTESTING）

Solution #2: 特征重要性分析（FEATURE IMPORTANCE ANALYSIS）

金融研究中很普遍的错误是在特定数据上尝试ML模型，不断调参直到得到一个比较好看的回测结果——这显然是过拟合。听说过一个笑话，”如果你的结果不好，说明你调参还不够努力“，学术期刊往往充斥这类虚假的发现，甚至很多是面向测试集调参。

考虑一个ML任务，给定 [公式] ，我们可以构建一个分类器，在交叉检验集上评估其泛化误差。假定结果很好，一个自然的问题是：哪些特征对结果的贡献最大？“好的猎人不会对猎狗捕获的猎物照单全收”，回答了这个问题，我们可以增加对提高分类器预测力有帮助的特征，减少几乎是噪声的特征。

很多人质疑ML是黑箱，因为ML的“学习”无需人类的指导，但这不意味着人不应该看看ML学出了什么东西。理解了ML发现的模式（pattern），才能更好地推动下一步工作：什么特征最重要、这些特征的重要性会随时间改变么、这种改变能否被识别和预测。总之，特征的重要性分析是比回测更好的研究策略。



# Pitfall 3: 按时间采样（CHRONOLOGICAL SAMPLING）

Solution #3: 交易量钟（THE VOLUME CLOCK）

1. Bars
为了对非结构化的数据使用机器学习算法，我们需要对原始数据进行解析，从中提取出有价值的信息，最后将提取结果进行规范化的存储。最常见的表示提取后信息的方法就是表格。金融领域中将这样的表格内的一条记录（或一个样本）叫做一个Bar。
1. Time Bars
Time bars 指的是以固定的时间区间对数据进行取样（如每分钟一次）后得到的数据。
尽管Time bars 是实践中最流行使用的处理方式，但这里我们指出它的两个不足：第一，市场交易信息的数量在时间上的分布并不是均匀的。开盘后的一小时内交易通常会比午休前的一小时活跃许多。因此，使用Time bars 会导致交易活跃的时间区间的欠采样，以及交易冷清的时间区间的过采样。第二，根据时间采样的序列通常呈现出较差的统计特征，包括序列相关、异方差等。
2. Tick Bars
Tick bars 是指每隔固定的（如1000次）交易次数提取上述的变量信息。一些研究发现这一取样方法得到的数据更接近独立正态同分布 [Ane and Geman 2000]。
使用Tick Bars 还需注意异常值 (outliers) 的处理。一些交易所会在开盘和收盘时进行集中竞价，在竞价结束后以统一价格进行撮合。
3. Volume Bars & Dollar Bars
Volume Bars 是指每隔固定的成交量提取上述的变量信息。Dollar Bars 则使用了成交额。
使用 Dollar Bars 相对而言是有一定优势的。假设一只股票在一定时间区间内股价翻倍，期初10000元可以购买的股票将会是期末10000元可购买股票手数的两倍。在股价有巨大波动的情况下，Tick Bars以及Volume Bars每天的数量都会随之有较大的波动。除此之外，增发、配股、回购等事件也会导致Tick Bars以及Volume Bars每天数量的波动

# Pitfall 4: 整数差分（INTEGER DIFFERENTIATION）

Solution #4: 非整数差分（FRACTIONAL DIFFERENTIATION）

我们需要在数据平稳性和保留数据信息之间做取舍，非整数/分数差分就是一个较好的解决方案


# Pitfall 5: 固定时间范围标签（FIXED-TIME HORIZON LABELING）
Solution #5: 三边界方法（THE TRIPLE-BARRIER METHOD）

大部分ML的论文几乎都用以下固定时间范围标签方法,该方法有若干不足：time bars 的统计性质并不好;常数阈值而不顾波动性是不明智的;可能被强制平仓.

三边界方法（THE TRIPLE-BARRIER METHOD）考虑到平仓的触发条件，是更好的处理方式，其包括上下水平边界和右边的垂直边界。水平边界需要综合考虑盈利和止损，其边界宽度是价格波动性的函数（波动大边界宽，波动小边界窄）；垂直边界考虑到建仓后 bar 的流量，如果不采用 time bars，垂直边界的宽度就不是固定的（翻译太艰难了，附上原文）

如果未来价格走势先触及上边界，可以取1；先触及下边界，则取-2（如下图）；先触及右边界，可以 0，或者根据盈利正负，取1或者-1 。


# Pitfall 6: 同时学出方向和规模（LEARNING SIDE AND SIZE SIMULTANEOUSLY）

Solution #6: 元标签（META-LABELING）

金融中用ML的另一常见错误是同时学习仓位的方向和规模（据我所知很多论文仅对买/卖方向做决策，每笔交易的金额/股数是固定的）。具体而言，方向决策（买/卖）是最基本的决策，规模决策（size decision）是风险管理决策，即我们的风险承受能力有多大，以及对于方向决策有多大信心。我们没必要用一个模型处理两种决策，更好的做法是分别构建两个模型：第一个模型来做方向决策，第二个模型来预测第一个模型预测的准确度。

很多ML模型表现出高精确度（precision）和低召回率（recall）。这意味着这些模型过于保守，大量交易机会被错过。

F1-score 综合考虑了精确度和召回率，是更好的衡量指标，元标签（META-LABELING）有助于构建高 F1-score 模型。首先（用专家知识）构建一个高召回率的基础模型，即对交易机会宁可错杀一千，不可放过一个。随后构建一个ML模型，用于决定我们是否应该执行基础模型给出的决策。元标签+ML有以下4个优势：

1. 大家批评ML是黑箱，而元标签+ML则是在白箱（基础模型）的基础上构建的，具有更好的可解释性；
2. 元标签+ML减少了过拟合的可能性，即ML模型仅对交易规模决策不对交易方向决策，避免一个ML模型对全部决策进行控制；
3. 元标签+ML的处理方式允许更复杂的策略架构，例如：当基础模型判断应该多头，用ML模型来决定多头规模；当基础模型判断应该空头，用另一个ML模型来决定空头规模；
4. 赢小输大会得不偿失，所以单独构建ML模型对规模决策是有必要的。

> achieving high accuracy on small bets and low accuracy on large bets will ruin you

# Pitfall 7: 非IID样本加权

Solution #7: （UNIQUENESS WEIGHTING AND SEQUENTIAL BOOTSTRAPPING）

实验室希望研究血液胆固醇含量受什么因素影响，从人群中随机实验者采集的血液样品服从独立同分布（IID），假如有人把每瓶血液都溅出一点到临近试管中，即试管10的血样包含试管1-9的部分血样，试管11的血样包含试管2-10的部分血样，现在要确定什么因素会影响血液胆固醇含量会非常困难。金融ML也面临同样的问题：

(1) labels are decided by outcomes;
(2) outcomes are decided over multiple observations;
(3) because labels overlap in time, we cannot be certain about what observed features caused an effect.

作者提出一种基于权重的采样方法。



# Pitfall 8: 交叉检验集泄露信息（CROSS-VALIDATION LEAKAGE）

Solution #8: 清理和禁止（PURGING AND EMBARGOING）

金融中需要警惕在训练集 / CV 集中引入未来信息。举个栗子，由于金融数据时序的自相关性， [公式]、 [公式] ；如果将 [公式] 划为训练集，将 [公式] 划为CV集，必然会将训练集的信息泄露（leakage）到CV集。好的做法应该是在训练集和CV集之间设定一个间隔：若 [公式] 是训练集最后一个数据，则CV集第一个数据可以为 [公式] 。


# Pitfall 9: 前向回测（WALK-FORWARD / HISTORICAL BACKTESTING）

Solution #9: CPCV（COMBINATORIAL PURGED CROSS-VALIDATION）

文献常用的回测方法是前向回测（Walk-forward Backtesting）：根据当前时刻以前的数据做决策。这种方式容易解读同时也很直观，但存在几点不足：

1. 前向回测只测试了单个场景，容易过拟合；
2. 前向回测的结果未必能代表未来的表现。

这里作者提出了一种更加丧心病狂的切分方法：将所有数据分为 [公式] 份（注意避免信息泄露），从中任意取 [公式] 份作为测试集，剩下 [公式] 份作为训练集，总共有 [公式] 种取法。这种方法最大的优势是允许我们得到某策略在不同时期的夏普率分布，而不是计算一个夏普率值。


# Pitfall 10: 回测过拟合（BACKTEST OVERFITTING）

Solution #10: 保守夏普率（THE DEFLATED SHARPE RATIO）

假设 $y_i$ 独立通过分布，可证明$E\left[\max \left\{y_{i}\right\}_{i=1, \ldots, I}\right] \leq \sigma \cdot \sqrt{2 \log (I)}$。
若$y_i$ 代表一系列回测结果的夏普率，则只要回测次数足够多，或者每次回测结果方差足够大，从中都能选出任意高的结果，尽管有可能 $E(y_i)=0$ 。

这提醒我们要考虑到回测次数造成的过拟合，一种解决方案是保守夏普率（THE DEFLATED SHARPE RATIO，DSR），其思想是给定一系列对夏普率SR的估计值，通过统计检验的方法估计能否推翻零假设 SR=0 。


## 参考

1. 《Advances in Financial Machine Learning》
2. https://blog.csdn.net/weixin_38753422/article/details/100179559
3. https://zhuanlan.zhihu.com/p/69231390