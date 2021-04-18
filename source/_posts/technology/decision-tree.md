---
title: 决策树是二叉树还是多叉树？
author: chiechie
mathjax: true
date: 2021-04-07 20:56:02
tags: 
- 人工智能
- 决策树
categories:
- 技术
---

> 今天被问到决策树是不是都是二叉树，我给问懵逼了。这触及到了我的知识盲区。
> 
> 仔细一调研，发现决策树算法早先的版本，是可以构造多叉树的。

# 总结

决策树的 实现算法有三类：id3，c4.5和cart。前两者可以构造多叉树，cart只能构造二叉树。
因为cart效果最好，现在通常就用它。例如sklean的决策树默认用cart

> 都说到这里了，就复习下决策树的基础知识吧

# 基本概念

- 信息增益: 衡量的是给定了一个属性之后，类别列上获得的秩序的提升（也叫熵减）。

${\displaystyle IG(T,a)=\mathrm {H} {(T)}-\mathrm {H} {(T|a)},}$


# 决策树算法-id3
Python Implementation of ID3


https://github.com/dozercodes/DecisionTree
https://github.com/SebastianMantey/Decision-Tree-from-Scratch/blob/master/notebooks/decision_tree_functions.py

# 决策树算法-c4.5

# 决策树算法-cart




# 参考
1. [wiki-Information_gain_in_decision_trees](https://en.wikipedia.org/wiki/Information_gain_in_decision_trees)
1. [sklearn-decisiontree](https://scikit-learn.org/stable/auto_examples/tree/plot_unveil_tree_structure.html#sphx-glr-auto-examples-tree-plot-unveil-tree-structure-py)
2. [quora-ID3-C4-5-and-CART的区别？](https://www.quora.com/What-are-the-differences-between-ID3-C4-5-and-CART)