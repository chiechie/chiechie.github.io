---
title: 随机游走
author: chiechie
mathjax: true
date: 2021-05-23 12:52:46
tags:
- 因果推断
- 因果分析
- 贝叶斯
- 根因分析
- AIOps
categories: 
- 图
---

## 介绍

已知因果图时，随机游走可以用来做因果推断。

原理是，基于因果关系，构建概率转移矩阵，通过模拟故障传播路径，得到故障根因。


## 随机游走算法-详细

- Step 1. 生成一个关系图G. $e_{ij} = 1$ 表示节点i是节点j的原因（之一）
- Step 2. 计算转移矩阵Q:
    1. 向前游走：从result节点到 the cause节点.理论上，跟异常节点越相关的节点，就越有可能是根因. 也就是说 $Q_{ij} = R(v_{abnormal}, v_j)$, $R(v_{abnormal}, v_j)$表示异常节点$v_{abnormal}$ 和 $ v_j$之间的相关系数, and $e_{ji} = 1$ 
    2. 向后游走：从cause节点到result节点. 为了避免算法陷入跟异常不相关或者低相关的节点，随机游走允许从cause节点跳出到result节点。
如果 $e_{j i} \in E$且 $e_{ij} \notin E$, 那么 $$Q_{ji} =\rho R\left(v_{abnormal}, v_{i}\right),\rho \in[0,1]$$.
    3. 维持原状：如果一个节点，它的邻居们都跟异常节点的相关性很低，这个节点很有可能就是根因了，所以游走者应该停留在这里，
       $$Q_{i i}=\max \left[ 0, R\left(v_{abnormal}, v_{i}\right)- \max _{k: e_{k i} \in E} R\left(v_{abnormal}, v_{k}\right) \right]$$	
- Step 3. 对行做归一化，得到转移概率矩阵
  $$\bar{Q}_{i j}=\frac{Q_{i j}}{\sum_{j} Q_{i j}}$$
- Step 4. 在G上面随机游走，使转移概率$\bar{Q}$

采用类似pagerank的方法，得到每个节点的得分。

## 参考
4. [Page3,4 -paper](https://netman.aiops.org/wp-content/uploads/2020/06/%E5%AD%9F%E5%AA%9B.pdf)