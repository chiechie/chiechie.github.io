---
title: 图数据基础
author: chiechie
mathjax: true
date: 2021-06-22 11:42:17
tags:
categories:
---



## 图的定义

传统的空间都是定义在欧几里得空间（Euclidean Space）的，在该空间定义的距离被称为欧式距离。
音频，图像，视频等都是定义在欧式空间下的欧几里得结构化数据，然而对于社交网络等数据，使用欧几里得空间进行定义并不合适，所以将这一类的数据所处的空间称为非欧几里得空间。

非欧空间下最有代表的结构就是图（Graph）结构， 其定义形式如下： [公式] , [公式] 表示顶点或节点，其中 [公式] 表示Graph中节点的个数。 [公式] 表示Graph中的边。

下面几个基本概念的定义

### 邻接矩阵（Adjacency matrix）

邻接矩阵是一个NxN的矩阵，对于有权图，其值为权重或0，对于无权图，其值为0和1，该矩阵定义如下：
$$A \in R^{N \times N}, A_{i j}=\left\{\begin{array}{ll}a_{i j} \neq 0 & e_{i j} \in E \\ 0 & \text { othersize }\end{array}\right.$$

### 度矩阵（Degree matrix）

度矩阵D是一个对角矩阵，其定义为：

$$D \in R^{N \times N}, D_{i i}=\sum_{j} A_{i j}$$

### 邻域（Neighborhood）

邻域表示与某个顶点有连接的点集，其定义为：$$N\left(v_{i}\right)=\left\{v_{j} \mid e_{i j} \in E\right\}$$


### 谱（Spectral）

只有方阵才有谱概念，方阵作为线性算子，其所有特征值的集合称为方阵的谱。方阵的谱半径为其最大的特征值，谱分解就是特征分解。

### 拉普拉斯矩阵（Lapalcian matrix）

对于无向图，其拉普拉斯矩阵定义为 $L=D-A$.

而大多数论文中使用的都是正规拉普拉斯矩阵（Symmetric Normalized Laplacian matrix）:
$$L^{s y s}=D^{-1 / 2} L D^{-1 / 2}=I-D^{-1 / 2} A D^{-1 / 2}$$

### 谱分解(Spectral Factorization)

谱分解又叫特征值分解，实际上就是对n维方阵做特征分解. 只有含有n个线性无关的特征向量的n维方阵才可以进行特征分解.

拉普拉斯矩阵是半正定矩阵,有如下的几个性质:

1. 实对称矩阵,有n个线性无关的特征向量;
2. 其特征向量可以进行正交单位化;
3. 所有的特征值非负;

$$L=U\left(\begin{array}{ccc}\lambda_{1} & & \\ & \ddots & \\ & & \lambda_{n}\end{array}\right) U^{-1}=U\left(\begin{array}{ccc}\lambda_{1} & & \\ & \ddots & \\ & & \lambda_{n}\end{array}\right) U^{T}$$



## 图的算法

### 最小割

最小割是指去掉图中的一些边，在使得图从连通图变为不连通图，并且尽可能保证去除的边对应的权重很小。

对于相似性图来说，最小割就是要去除一些很弱的相似性，把数据点从一个连通的大图切割成多个独立的连通小图。


## 参考

1. https://zhuanlan.zhihu.com/p/84271169