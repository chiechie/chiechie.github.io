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

1. 决策树做预测的流程可以分为两步：第一步，将特征空间划分为多个distinct and non-overlapping regions,$R_1,\dots,R_J$；第二步，对每个region定义一个response variable，作为该region的值。
2. 怎么得到这些region呢？决策树的实现算法有三类：ID3，C4.5和cart。前两者可以构造多叉树，cart只能构造二叉树。 
因为cart效果最好，现在通常就用它（例如sklean）。
3. 三类算法的大致思路一样，分为两步：第一步是将feature space切成多个boxes。为什么不是切成多个球？因为球没法填充整个feature space.
第二步是找到最优的切割boxes的方式，如果去遍历每一组partition of feature space，计算量太大了，通常采用greedy的方法。
4. 落入每个region的response variable是什么？

  - 如果是连续变量，求该region上训练集的response均值，以后赋值给新落入该区间的predictor的response，
  - 如果是离散变量，求众数。

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


# 决策树算法-ID3

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


# 决策树算法-CART

ID3 和 C4.5是使用基于Entropy-最大信息增益的特征作为节点。

CART代表分类树和回归树，使用基于entropy和ginix index计算信息增益。

Disadvantages

- It can split on only one variable
- Trees formed may be unstable


cart的原理就是，构造一颗大树$T_0$，然后去剪枝（也叫做cost complexity pruning/the weakest link pruning）, 下面以regression tree 和 classification tree举例说明

> 如果response var是imbalance, 全部预测为label占比更多的类，怎么办？

## regression tree

regression tree的cost function 是RSS加上正则项

$$\min\limits_{T\in T_0} \sum\limits_{m=1}^{|T|}\sum\limits_{x_i\in R_m}(y_i - \hat y_{R_m})^2+\alpha|T|$$

- $|T|$是叶子节点的个数。
- m表示第m个叶子
- $R_m$表示第m个partition region 
- $y_i$表示第i个样本的真实值
- $y_{R_m}$表示第m个partition region的预测值

构造一颗树的流程如下：

![regression tree 构造流程](./img.png)

## classification tree

classification tree切分节点时，参考信息增益，其他流程和构建回归树是一样的


## 为何要剪枝?

为何要剪枝(pruning)?
a very bushy tree has got high variances,ie, over-fitting the data



## 参考
1. [github-id3的实现](https://github.com/dozercodes/DecisionTree)
2. [github-id3的实现](https://github.com/SebastianMantey/Decision-Tree-from-Scratch/blob/master/notebooks/decision_tree_functions.py)
3. [wiki-Information_gain_in_decision_trees](https://en.wikipedia.org/wiki/Information_gain_in_decision_trees)
4. [sklearn-decisiontree](https://scikit-learn.org/stable/auto_examples/tree/plot_unveil_tree_structure.html#sphx-glr-auto-examples-tree-plot-unveil-tree-structure-py)
5. [quora-ID3-C4-5-and-CART的区别？](https://www.quora.com/What-are-the-differences-between-ID3-C4-5-and-CART)
6. [Why-is-entropy-used-instead-of-the-Gini-index](https://www.quora.com/Why-is-entropy-used-instead-of-the-Gini-index)
7. [An Introduction to Statistical Learning](https://static1.squarespace.com/static/5ff2adbe3fe4fe33db902812/t/6062a083acbfe82c7195b27d/1617076404560/ISLR%2BSeventh%2BPrinting.pdf)