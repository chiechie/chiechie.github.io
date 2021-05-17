---
title: 树模型2_决策树介绍
author: chiechie
mathjax: true
date: 2021-04-07 20:56:02
tags: 
- 人工智能
- 树模型
- 决策树
categories:
- AI
---

> 决策树不一定是二叉树，早先的版本，是可以构造多叉树的。


# 总结

决策树的 实现算法有三类：id3，c4.5和cart。前两者可以构造多叉树，cart只能构造二叉树。
因为cart效果最好，现在通常就用它。例如sklean的决策树默认用cart

> 都说到这里了，就复习下决策树的基础知识吧

# 基本概念

- 信息增益: 衡量的是给定了一个属性之后，类别列上获得的秩序的提升（也叫熵减）。

${\displaystyle IG(T,a)=\mathrm {H} {(T)}-\mathrm {H} {(T|a)},}$


# 决策树算法-id3
Python Implementation of ID3


# 决策树算法-c4.5

# 决策树算法-cart

## 原理

将predictor space划分为J个distinct and non-overlapping regions,$R_1,\dots,R_J$


## 每个region的response variable是什么？

落入每个region的response variable是什么？

  - 如果是连续变量，求该region上训练集的response均值，以后赋值给新落入该区间的predictor的response，
  - 如果是离散变量，求众数。

## 怎么找最佳的region partition ？

怎样构造这些region呢？ 对predictor space进行一个切割，切成boxes
   
   > 思考？为什么不是切成一个球？没法填充整个redictor space


定义为最优化问题：the goal is to find boxes$R_1,\dots,R_j$ that minimize the SSE，即为了获得最小组内方差(within variance)。

$$\min\ SSE = \sum\limits_{j=1}^J \sum\limits_{i\in R_j}(y_i - \hat y_{R_j})^2$$

- i是第i个样本
- R_j表示第j个partition

怎么找到最佳的一个partition？如果考虑每一个partition of predictor space，那计算量太大了。

自然，用greedy的方法。

  > 如果response var是imbalance, 全部预测为label占比更多的类，怎么办？

cart的原理就是，构造一颗大树$T_0$，然后去剪枝，这种方法叫做cost complexity pruning/the weakest link pruning, 下面以regression tree 和 classification tree举例说明：


## regression tree

$$\min\limits_{T\in T_0} \sum\limits_{m=1}^{|T|}\sum\limits_{x_i\in R_m}(y_i - \hat y_{R_m})^2+\alpha|T|$$

- $|T|$是叶子节点的个数。


构造一颗树时，stop rules是叶子节点的node个数少于阈值。

## classification tree

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

## 为何要剪枝?

为何要剪枝(pruning)?
如果分支过多(bushy)，造成over fitting。

a smaller tree with fewer splits might lead to lower variance and better interpretation at the cost of a little bias.

总的来说,a very bushy tree has got high variances,i e ,over fitting the data


## 参考
1. [github-id3的实现](https://github.com/dozercodes/DecisionTree)
2. [github-id3的实现](https://github.com/SebastianMantey/Decision-Tree-from-Scratch/blob/master/notebooks/decision_tree_functions.py)
3. [wiki-Information_gain_in_decision_trees](https://en.wikipedia.org/wiki/Information_gain_in_decision_trees)
4. [sklearn-decisiontree](https://scikit-learn.org/stable/auto_examples/tree/plot_unveil_tree_structure.html#sphx-glr-auto-examples-tree-plot-unveil-tree-structure-py)
5. [quora-ID3-C4-5-and-CART的区别？](https://www.quora.com/What-are-the-differences-between-ID3-C4-5-and-CART)