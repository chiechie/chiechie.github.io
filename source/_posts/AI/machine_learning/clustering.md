---
title: 一些聚类算法
author: chiechie
mathjax: true
date: 2021-06-23 11:20:42
tags:
- 图数据
- 聚类
- 日志聚类
categories:
- AI
---


> 除了kmeans，聚类算法中还有一类更简单直观的方法，就是层次聚类。

## 总结

1. 我们常常对表数据使用kmeans来实现聚类，内部流程分为两步，第一步是是将多维的表数据（维度代表特征），转为2D的相似图（一个节点代表一个样本）。第二步，是对这个图进行聚类。
2. kmeans在实际中应用较多，但是并没有理论保证其收敛。
   > ...I don't think there is a nice theory of convergence or rate of convergence, But is  a very popular algorimth,..., But that's one sort of hack that works quite well--Gilbert Strang
3. 对一个图，找到其中的一些cluster，kmeans是其中一种方法，还有谱聚类（spectral clustring）。
4. 除此之外，还可以使用终极大招--使用数值方法去求解优化问题（比如BP，就是alternative methods的一个特例）。
5. 按照理论严密从高到低，实现聚类的算法可以分为三类：谱聚类，基于运筹优化的聚类，一些直觉类的方法。直觉类的方法包括kmeans，层次聚类


## 聚类算法summary

目前常用的聚类算法大概分为三类：

- 层次聚类： nearest neigbour
- 基于分区的聚类（partitional）：Kmeans系列/FCM/图理论
- 基于密度：DBSCAN
- 其他：ACODF

![img.png](./img.png)



## 层次聚类

层次聚类算法,又称为树聚类算法,它使用数据的联接规则,透过一种层次架构方式,反复将数据进行分裂或聚合,以形成一个层次序列的聚类问题解.

本文仅以层次聚类算法中的层次聚合算法为例进行介绍.

层次聚合算法的时间复杂度为$O(n^2)$,适合于小数据集的分类. 

该算法由树状结构的底部开始逐层向上进行聚合,假定样本集S={o1,o2,...,on}共有n个样本.

1. 初始化：每个样本 $o_i$ 为一个类; 共形成 n 个类:$o_1,o_2,...,o_n$
2. 找最近的**两个类**：从现有所有类中找出相似度最大的**两个**类$o_r$ 和 $o_k$
   $$distance(o_r,o_k) = min_{\forall{o_u,o_v \in S,o_u \neq o_v}}distance(o_u,o_v)$$
3. 将类$o_r$和$o_k$合并成一个新类$o_{rk}$，现有cluster个数减1
4. 若所有的样本都属于同一个类,则终止;否则,返回步骤2.


层次聚类最大的优点，它一次性地得到了整个聚类的过程。如果想要改变cluster个数，不需要重新训练聚类模型，取聚类树的中间某一层就好了。相比之下kmeans是要重新弄训练的。

层次聚类的缺点是计算量比较大，因为要每次都要计算多个cluster内所有数据点的两两距离，复杂度是O(n^2*d)，d表示聚类树的深度。 

还有一个缺点，超参数比较难设置，跟kmeans一样。最终到底聚成多少类，需人工给定一个distance threshold，这个阈值跟sample有关，跟距离函数的定义也有关，并且聚类结果对这个参数比较敏感。

btw，如何判断两个cluster之间的距离？

- 一开始每个数据点独自作为一个类，它们的距离就是这两个点之间的距离。
- 对于包含不止一个数据点的cluster，最常用的方法是average-linkage，即计算两个cluster各自数据点的两两距离的平均值。
  其他方法还有single-linkage/complete-linkage，选择两个cluster中距离最短/最长的一对数据点的距离作为类的距离。
  个人经验complete-linkage基本没用，single-linkage通过关注局域连接，可以得到一些形状奇特的cluster，但是因为太过极端，所以效果也不是太好。


## 基于Graph的聚类

基于Graph的聚类算法，大致思路是，计算样本两两之间的相似度，并设定阈值，高于阈值就右边，低于阈值就无边，以此构造有权无向图，权重即为相似度。



### 谱聚类

什么是spectrum？

矩阵的spectrum就是矩阵的特征根。
spectral clustring就是使用矩阵的特征根聚类


回忆一下，其拉普拉斯矩阵定义为 $L=D-A$.

怎么根据L来找到目标的clusters（or centriod）？
对L进行谱分解（特征根分解），并且找到L的fieder向量。

fieder向量中的元素有正的，也有负的，
正的位置对应的样本分配到cluster1，负元素对应的样本分配到cluster2

谱聚类的流程是：

输入：n个样本, 类别k

1. 根据样本两两之间的相似度，构建有权无向图，以及邻接矩阵W。
2. 计算出拉普拉斯矩阵L，对L做谱分解（相当于降维）：计算拉普拉斯矩阵L的最小的k个特征值对应的特征向量u1, u2,...,uk,将这些向量组成n*k维的矩阵U,
3. 将U中的每一行作为一个样本，共n个样本，使用k-means对这n个样本进行聚类

得到簇划分C(c1,c2,...ck).



## 聚类应用



### 日志分析


### 图像分割

图像的分割是图聚类的其中一个应用场景。

层次聚类可看成框架，而基于图的图像分割是在层次聚类上加了骨头，使他更适用于图像分割的领域，而使用者可以在骨头上继续加肉来达到不同的分割效果。

图像中的每一个像素点是一个item，图像分割的任务即对所有items聚类，属于一个cluster的所有像素就构成了一个区域，
最终的分割结构就是若干个区域（即clusters）组成的。由此我们过渡到本文的第二部分，


基于图的图像分割算法，主要流程如下

输入一个图$G=(V,E)$，有n个点和m个边。输出是一个分割V，分割成$S=(C_1,...,C_2).$

1. 对边E进行排序，生成非递减的序列$\pi = (o_1,...,o_m)$
2. 从初始分割$S^0$开始，每一个点$v_i$自己就是一个区域
3. 对于每一个边$q = 1,...,m$重复步骤3，通过$S^{q-1}$构建$S^q$，使用如下的方式
   
    - 令$v_i$和$v_j$表示按顺序排列的第q条边的两个点，比如$o_q = (v_i,v_j)$。
    - 如果$v_i$和$v_j$在$S^{q-1}$中连个不同的区域下，并且$w(o_q)$比两个区域的内部差异都小，那么合并这连个区域，否则什么也不做。
    - 用公式来表达就是：
      - 令$C_{i}^{q-1}\)是\(S^{q-1}$的一个区域，它包含点$v_i$；令$C_{j}^{q-1}\)是\(S^{q-1}$的一个区域，它包含点$v_j$。
      - 如果$C_{i}^{q-1} \neq C_{j}^{q-1}$ 并且$w(o_q) \leq MInt(C_i^{q-1},C_j^{q-1})$，那么通过合并$C_{i}^{q-1}$和$C_{j}^{q-1}$得到了$S^q$；
      - 否则的话$S^q = S^{q-1}$，返回$S = S^m$

图像分割结果如下

![图像分割](img1.png)



## 参考

1. [一种基于图结构的日志聚类算法](https://patentimages.storage.googleapis.com/34/c0/df/3417293b1602a5/CN105468677A.pdf)
2. [常用聚类算法综述](https://zhuanlan.zhihu.com/p/78382376)
3. [从层次聚类到Graph-based图像分割](https://buptjz.github.io/2014/04/21/cluster)
4. [聚类算法](http://www.jos.org.cn/1000-9825/19/48.pdf)
5. [35. Finding Clusters in Graphs-mit-youtube](https://www.youtube.com/watch?v=cxTmmasBiC8)