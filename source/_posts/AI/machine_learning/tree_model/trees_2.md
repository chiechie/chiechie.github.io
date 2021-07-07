---
title: 树模型2 决策树介绍
author: chiechie
mathjax: true
date: 2021-04-16 20:56:02
tags: 
- 人工智能
- 树模型
- 决策树
categories:
- AI
---


# 总结

## 决策树

1. 决策树是一个预测算法，是使用贪婪的递归的方法来找到最优的预测结构。
1. 构建一个决策树需要确定分支条件以及每个分支的预测值。构建分支条件即把特征空间划分为多个distinct and non-overlapping regions,$R_1,\dots,R_J$；第二步，对每个region定义一个response variable，作为该region的值。
2. 构建好了一个决策树，想要使用决策树做预测，步骤也分为两步：第一步是按照分支条件将新样本路由到指定的叶子结点，第二步将叶子结点对应的responsible varible作为该样本的预测值。
3. 怎么得到分支条件呢？有三类构建决策树的算法：ID3，C4.5和cart。前两者可以构造多叉树，cart只能构造二叉树。 
因为cart效果最好，现在通常就用它（例如sklean）。
4. ID3，C4.5和cart三类算法的大致思路一样，分为两步：第一步是将feature space切成多个boxes。为什么不是切成多个球？因为球没法填充整个feature space.
5. 如何找到最优的切割boxes的方式，如果去遍历每一组partition of feature space，计算量太大了，通常采用greedy的方法。
6. 构建决策树的过程中需要确定的参数：分支的个数，条件分支的条件，终止条件，叶子结点的值。


## 决策树算法-CART
1. cart的特色是构建的一个binary tree，每次分支条件都是将一个空间以分为2，变成2个子空间，每个叶子结点的值，即response variable都是一个常数，是这么的到的：
  - 如果target var是一个连续变量，求落入该region的训练集的response均值，即${y_n}$的均值，其实这个均值对应的是最小化suqared error。
构建一个决策树，主要是要确定partition，或者说分支条件，以及每个落入每个partition（或者说）中对应的预测值。
  - 如果target var是一个离散变量，求众数对应的那个类别。
2. 怎么确定分支条件？找一个decision stump，使用纯度（purify）来衡量分支的质量，如果左边的data set 和右边的dataset 纯度 都很高，（其中的大部分样本的label很接近），就说切分的很好。对应到计算上面，就是找一个让平均不纯度最小的切分方式（decision stump）。
3. 如何确定分支/切割的不纯度？
    - 如果target var是一个连续变量，使用squared loss来描述impurity，（跟样本子集的均值比），
    - 如果target var是一个离散变量，使用不一致样本比例来描述impurity，（跟样本子集的众数币），多分类的时候，不纯度常用giniindex
    - 如果是多分类，经常使用Gini index来刻画不纯度。
4. 什么时候会停下来？当满足下面的条件时，也叫fully grown tree：
    - 落入某个分支的样本的target都一样，不纯度取到最小了，
    - 落入某个分支的样本的x都一样，没有decision stupms了。
5. 为何要剪枝(pruning)? a very bushy tree has got high variances,ie, over-fitting the data


# 附录

## 基本概念

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


## 决策树算法-ID3

ID3,  was the first of three Decision Tree implementations developed by Ross Quinlan

It builds a decision tree for the given data in a top-down fashion. each node of the tree, one feature is tested based on 最大熵降, and the results are used to split the sample set. This process is recursively done until the set in a given sub-tree is homogeneous (i.e. it contains samples belonging to the same category). The ID3 algorithm uses a greedy search. 

Disadvantages:

- Data may be over-fitted or over-classified, if a small sample is tested.
- Only one attribute at a time is tested for making a decision.
- Does not handle numeric attributes and missing values.

## 决策树算法-C4.5

Improved version on ID 3 . The new features (versus ID3) are:

- (i) accepts both continuous and discrete features; 
- (ii) handles incomplete data points;
- (iii) solves over-fitting problem by (very clever) bottom-up technique usually known as "pruning"; and 
- (iv) different weights can be applied the features that comprise the training data.

Disadvantages

- Over fitting happens when model picks up data with uncommon features value, especially when data is noisy.


## 决策树算法-CART

ID3 和 C4.5是使用基于Entropy-最大信息增益的特征作为节点。

CART代表分类树和回归树，使用基于entropy和ginix index计算信息增益。

Disadvantages

- It can split on only one variable
- Trees formed may be unstable


cart的原理就是，构造一颗大树$T_0$，然后去剪枝（也叫做cost complexity pruning/the weakest link pruning）, 下面以regression tree 和 classification tree举例说明

> 如果response var是imbalance, 全部预测为label占比更多的类，怎么办？

### regression tree

regression tree的cost function 是RSS加上正则项

$$\min\limits_{T\in T_0} \sum\limits_{m=1}^{|T|}\sum\limits_{x_i\in R_m}(y_i - \hat y_{R_m})^2+\alpha|T|$$

- $|T|$是叶子节点的个数。
- m表示第m个叶子
- $R_m$表示第m个partition region 
- $y_i$表示第i个样本的真实值
- $y_{R_m}$表示第m个partition region的预测值

可以使用一个递归的方法来构建一个决策树，主要是要确定partition，或者说分支条件，以及每个落入每个partition（或者说）中对应的预测值。

![regression tree 构造流程](./img.png)

### classification tree

classification tree切分节点时，参考信息增益，其他流程和构建回归树是一样的


# 参考
1. [决策树算法-linxuantian](https://www.youtube.com/watch?v=s9Um2O7N7YM)
1. [决策树-linxuantian](https://www.csie.ntu.edu.tw/~htlin/mooc/doc/209_present.pdf)
1. [An Introduction to Statistical Learning](https://static1.squarespace.com/static/5ff2adbe3fe4fe33db902812/t/6062a083acbfe82c7195b27d/1617076404560/ISLR%2BSeventh%2BPrinting.pdf)
2. [Why-is-entropy-used-instead-of-the-Gini-index](https://www.quora.com/Why-is-entropy-used-instead-of-the-Gini-index)
1. [github-id3的实现1](https://github.com/dozercodes/DecisionTree)
2. [github-id3的实现2](https://github.com/SebastianMantey/Decision-Tree-from-Scratch/blob/master/notebooks/decision_tree_functions.py)
3. [wiki-Information_gain_in_decision_trees](https://en.wikipedia.org/wiki/Information_gain_in_decision_trees)
4. [sklearn-decisiontree](https://scikit-learn.org/stable/auto_examples/tree/plot_unveil_tree_structure.html#sphx-glr-auto-examples-tree-plot-unveil-tree-structure-py)
5. [quora-ID3-C4-5-and-CART的区别？](https://www.quora.com/What-are-the-differences-between-ID3-C4-5-and-CART)
