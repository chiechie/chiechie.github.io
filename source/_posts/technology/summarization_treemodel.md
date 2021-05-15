---
title: 常用树模型介绍
author: chiechie
mathjax: true
date: 2021-05-15 14:59:29
tags:
- 人工智能
- 树模型
categories:
- 技术
---

## 1.总览

最基本的树模型是cart，优势在于可解释性，但是跟其他的监督学习方法相比，准确性并没有多大的优势。

但是，通过将多棵树以不同的方式组合，会得到很好的准确率，如bagging，random foreast和boosting等等方法。

回一下机器学习的三个要素：
1. 假设函数的空间： 
2. 学习策略or目标函数
3. 优化算法


## 2. cart

1. 决策树的原理？ 将predictor space划分为J个distinct and non-overlapping regions,$R_1,\dots,R_J$
2. 怎样构造这些region呢？ 对predictor space进行一个切割，切成boxes
   
   > 思考？为什么不是切成一个球？没法填充整个redictor space
3. 落入每个区间的response variable的值怎么确定？
  - 如果是连续变量，求该region上训练集的response均值，以后赋值给新落入该区间的predictor的response，
  - 如果是离散变量，求众数。

4.怎样表示成一个优化问题?
  the goal is to find boxes$R_1,\dots,R_j$ that minimize the SSE，即为了获得最小组内方差(within variance)。

$$\min\ SSE = \sum\limits_{j=1}^J \sum\limits_{i\in R_j}(y_i - \hat y_{R_j})^2$$

怎么找到最佳的一个partition？如果考虑每一个partition of predictor space，那计算量太大了。

自然而然的，用greedy的方法。

  > 如果response var是imbalance, 全部预测为label占比更多的类，怎么办？

5. 为何要剪枝(pruning)?
如果分支过多(bushy)，造成over fitting。
a smaller tree with fewer splits might lead to lower variance and better interpretation at the cost of a little bias.
总的来说,a very bushy tree has got high variances,i e ,over fitting the data
cart的原理就是，构造一颗大树$T_0$，然后去剪枝， 这种方法叫做cost complexity pruning/the weakest link pruning, 下面以regression tree 和  classification tree举例说明：

###  2.1 regression tree

$$\min\limits_{T\in T_0} \sum\limits_{m=1}^{|T|}\sum\limits_{x_i\in R_m}(y_i - \hat y_{R_m})^2+\alpha|T|$$

$|T|$是叶子节点的个数。

构造一颗树时，stop rules是叶子节点的node个数少于阈值。

### 2.2 classification tree

regression tree的cost function 是rss，而classification tree的cost是可以是
- **classification error**
$$1-\max\limits_k(\hat p_{mk})$$
也就是每一个region中不属于主要类的样本点的比例，classification error可以看成衡量的是样本的一个众数,也叫做node purity。但是用该策略来决定split方式，会有点jump ，我们需要一个smooth tree-growing process，于是提出了优化的指标gini index：

- **gini index**
相对于classification error,这个是一个关于$k$更加soft的函数
$G = \sum\limits_k \hat p_{mk}(1-\hat p_{mk})$。
他衡量的是K类的一个总的方差,也就是K个变量的协方差矩阵的迹，越小越纯粹。

- **cross-entropy** 
$d = - \sum\limits_k \hat p_{mk}\log\hat p_{mk} $
形状跟gini index差不多，优势在于处理分类变量时不需要转化为dummy variable。


  
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