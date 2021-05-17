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


# 总结

1. 决策树通过将特征空间划分为多个distinct and non-overlapping regions,$R_1,\dots,R_J$，并且对每个region定义一个response variable，作为该region的值。
2. 决策树的 实现算法有三类：id3，c4.5和cart。前两者可以构造多叉树，cart只能构造二叉树。 因为cart效果最好，现在通常就用它。例如sklean的决策树默认用cart
   

# 基本概念

- 信息增益: 衡量切分前后，样本的秩序的提升or混乱程度的下降。

```python
IG = information before splitting (parent) — information after splitting (children)
```
- 具体的，有两个衡量混乱程度的指标：Entropy 和 Gini Impurity
    - **gini index**: $$I_{G}=1-\sum_{j=1}^{c} p_{j}^{2}$$
        - $p_j$: 落入该节点的样本中，第j类样本的占比
        - 如果所有样本都属于某一类c，gini系数最小，为0。
    - entropy（熵）：$$I_{H}=-\sum_{j=1}^{c} p_{j} \log _{2}\left(p_{j}\right)$$
        - $p_j$: 落入该节点的样本中，第j类样本的占比
        - 如果所有样本都属于某一类c，熵最小，为0。
    

# 决策树算法-id3

ID3,  was the first of three Decision Tree implementations developed by Ross Quinlan

It builds a decision tree for the given data in a top-down fashion. each node of the tree, one feature is tested based on 最大熵降, and the results are used to split the sample set. This process is recursively done until the set in a given sub-tree is homogeneous (i.e. it contains samples belonging to the same category). The ID3 algorithm uses a greedy search. 

Disadvantages:

- Data may be over-fitted or over-classified, if a small sample is tested.
- Only one attribute at a time is tested for making a decision.
- Does not handle numeric attributes and missing values.

# 决策树算法-C4.5

Improved version on ID 3 . The new features (versus ID3) are:

- (i) accepts both continuous and discrete features; 
- (ii) handles incomplete data points;
- (iii) solves over-fitting problem by (very clever) bottom-up technique usually known as "pruning"; and 
- (iv) different weights can be applied the features that comprise the training data.

Disadvantages

- Over fitting happens when model picks up data with uncommon features value, especially when data is noisy.


# 决策树算法-cart

ID3 和 C4.5是使用基于Entropy-最大信息增益的特征作为节点。

CART代表分类树和回归树。

Disadvantages

- It can split on only one variable
- Trees formed may be unstable


## 原理


## 每个region的response variable是什么？

落入每个region的response variable是什么？

  - 如果是连续变量，求该region上训练集的response均值，以后赋值给新落入该区间的predictor的response，
  - 如果是离散变量，求众数。

## 怎么找最佳的region partition ？

怎样构造这些region呢？ 分两步

1. 对feature space进行一个切割，切成boxes
   
   > 思考？为什么不是切成一个球？没法填充整个predictor space
2. 找到最优的一系列的boxes，即最优化问题：the goal is to find boxes$R_1,\dots,R_j$ that minimize the SSE，即为了获得最小组内方差(within variance)。

$$\min\ SSE = \sum\limits_{j=1}^J \sum\limits_{i\in R_j}(y_i - \hat y_{R_j})^2$$

- i是第i个样本
- R_j表示第j个partition

怎么找到最佳的一个partition？如果考虑每一个partition of predictor space，那计算量太大了。

自然，用greedy的方法。

  > 如果response var是imbalance, 全部预测为label占比更多的类，怎么办？

cart的原理就是，构造一颗大树$T_0$，然后去剪枝，这种方法叫做cost complexity pruning/the weakest link pruning, 下面以regression tree 和 classification tree举例说明：


## regression tree

regression tree的cost function 是RSS

$$\min\limits_{T\in T_0} \sum\limits_{m=1}^{|T|}\sum\limits_{x_i\in R_m}(y_i - \hat y_{R_m})^2+\alpha|T|$$

- $|T|$是叶子节点的个数。
- m表示第m个叶子


构造一颗树时，stop rules是叶子节点的node个数少于阈值。

## classification tree

classification tree的cost是可以是

- **classification error**
    $$1-\max\limits_k(\hat p_{mk})$$
    也就是每一个region中不属于主要类的样本点的比例，classification error可以看成衡量的是样本的一个众数,也叫做node purity。但是用该策略来决定split方式，会有点jump ，我们需要一个smooth tree-growing process，于是提出了优化的指标gini index：

- **cross-entropy** 
    $$d = - \sum\limits_k \hat p_{mk}\log\hat p_{mk} $$
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
6. [Why-is-entropy-used-instead-of-the-Gini-index](https://www.quora.com/Why-is-entropy-used-instead-of-the-Gini-index)