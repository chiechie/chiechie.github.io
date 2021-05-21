---
title: PC算法
author: chiechie
mathjax: true
date: 2021-03-09 10:06:00
tags:
- 因果推断
- 贝叶斯
categories:
- 技术
---

# 基本概念

- 因果图（cause graph）：因果模型的一种表达方式，因果图是一个DAG， G=（V，E），对于每一个节点v，给定了他的双亲pa（v），v跟所有非后代都是独立的。
- 冲撞点（collider）：一种V型结构的因果图，
- PC算法的两个假设：马尔可夫条件和faithfulness。
- 马尔科夫条件：可用来区分因果关系和相关关系，被用于生成依赖关系集合，构造因果图框架。
- faithfulness假设：可以用来保证，V中的所有变量的独立关系，都可以通过D-seperation表达。

## D-分离
D分离（D-Separation）定义很复杂，拿出来单独说。

定义: 设 X，Y，Z 是 DAG 中不相交的节点集合， $\pi$ 为一条连接 X 中某节点到 Y 中某节点的路径 （不管方向）。 如果路径 $\pi$ 上某节点满足如下的条件：

1. 在路径$\pi$上，w 点处为 V结构 , 即$X \rightarrow w \leftarrow Y$，且 w 及其后代不在 Z中；(因为collider关系，w不可观测时,X,Y 相互独立)
2. 在路径$\pi$上，w 点处为 非V结构 ，且w在Z中 。

则称Z阻断了路径$\pi$。 

如果Z阻断了所有的X到Y的路径，就说Z集合D分离了X和Y，记作$(X \perp Y \mid Z)_{G}$.

D分离这个性质有什么用呢？

D分离是一种用来判断变量是否条件独立的图形化方法，相比于非图形化方法，更加直观，且计算简单。 对于一个DAG(有向无环图)，D-Separation方法可以快速的判断出两个节点之间是否是条件独立的。

# PC算法-概览

PC算法是一种发现因果关系的算法，在满足一定的假设的前提下（基于马尔科夫条件 和 D-separation），使用基于统计的方法（显著性检验-$G^2$），推导出因果关系（代表因果关系的DAG）。实现流程包括三步:

1. 确定因果图的框架，有哪些点，有哪些边
2. 确定边的方向
3. 传播方向

![图1-PC算法](pc-overview.png)

# 确定因果图框架

从全连接图开始，移走条件独立的边

![图2-确定框架](pc1.png)


Z从空集开始，不断增加

具体来说，可以用指标$G^2$（条件交叉熵）来检验给定Z时，X和Y的独立性，这里X，Y，Z是互斥的。

$G^{2}=2 m C E(X, Y \mid Z)$

- m：样本大小
- $ C E(X, Y \mid Z)$:给定Z时，X和Y的条件交叉熵，

> 显著性低于0.05，就说明条件分布和联合分布没有显著性差异，有因果关系
> 显著性超过0.05， 就说明条件分布和联合分布没有显著性差异，没有因果关系，是独立的。

如果$G^2$超过一个预先设定的显著性，比如0.05， 那么X和Y是条件独立的。

PC算法使用一个全连接的无向图，然后使用$G^2$描述条件独立性。


# 确定方向

当DAG的架子搭起来了， 接下来的工作就是决定因果关系的方向了。这里使用到了D-seperation的方法。 

![图3-确定方向](pc2.png)

# 传播方向

![图4-传播方向](pc3.png)




# 参考

1. [2007-The Journal of MachineLearning Research-pc算法](https://www.jmlr.org/papers/volume8/kalisch07a/kalisch07a.pdf)
2. [pc算法-youtube](https://www.youtube.com/watch?v=o2A61bJ0UCw)
3. [因果图的基本概念-知乎](https://zhuanlan.zhihu.com/p/269625734)