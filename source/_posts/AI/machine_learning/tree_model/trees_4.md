---
title: 树模型4_xgboost介绍
author: chiechie
mathjax: true
date: 2021-05-15 14:59:29
tags:
- 人工智能
- 树模型
categories:
- AI
---

## xgboost

XGB在GBDT的基础上引入了二阶导数，并且在loss中加入了正则项。
boosting tree的另外一个代表是xgboost,下面重点介绍一下xgboost

### 损失函数定义

先看看，xgboost怎么定义ensemble tree的 损失函数：

$$\begin{array}{l}{\mathcal{L}(\phi)=\sum\limits_{i} l\left(\hat{y}_{i}, y_{i}\right)+\sum\limits_{k} \Omega\left(f_{k}\right)}\end{array}$$
$$ \\{\Omega(f)=\gamma T+\frac{1}{2} \lambda\|w\|^{2}} $$

- $\hat{y}_{i}=\phi\left(\mathbf{x}_{i}\right)=\sum^{K} f_{k}\left(\mathbf{x}_{i}\right), \quad f_{k} \in \mathcal{F}$
- l是一个可谓的图函数
- k表示第k棵树
- i表示第i个样本
- T is the number of leaves in the tree.
- $f_k$表示第k棵树的预测函数，
- q代表的书的结构，w代表叶子节点的权重
- $\Omega(f_k)$表示第k棵树的复杂度


### reformulate目标函数

上面的优化问题的解，近似求一系列优化问题的解，如下

$\mathcal{L}^{(t)}=\sum\limits_{i=1}^{n} l\left(y_{i}, \hat{y}_{i}^{(t-1)}+f_{t}\left(\mathbf{x}_{i}\right)\right)+\Omega\left(f_{t}\right)$

- $\mathcal{L}^{(t)}$表示第t棵树的训练目标，也就是通过依次求解最优的树来逼近最开始的损失函数。

### 损失函数-二阶展开

而为了方便计算，上面的损失函数还可以进一步简化（利用二阶展开）

$\mathcal{L}^{(t)} \simeq\sum\limits_{i=1}^{n}\left[l\left(y_{i}, \hat{y}^{(t-1)}\right)+g_{i} f_{t}\left(\mathbf{x}_{i}\right)+\frac{1}{2} h_{i} f_{t}^{2}\left(\mathbf{x}_{i}\right)\right]+\Omega\left(f_{t}\right)$

这里：

$g_{i}=\partial_{\hat{y}^{(t-1) }}l\left(y_{i}, \hat{y}^{(t-1)}\right)$

$h_{i}=\partial_{\hat y^{(t-1)}}^{2} l\left(y_{i}, \hat{y}^{(t-1)}\right)$

进一步，我们remove常数项，得到

$\tilde{\mathcal{L}}^{(t)}=\sum\limits_{i=1}^{n}\left[g_{i} f_{t}\left(\mathbf{x}_{i}\right)+\frac{1}{2} h_{i} f_{t}^{2}\left(\mathbf{x}_{i}\right)\right]+\Omega\left(f_{t}\right)$

### 基于样本集重新定义损失函数

更进一步，我们将$I_{j}=\left\{i | q\left(\mathbf{x}_{i}\right)=j\right\}$定义为落入叶子j的样本集, 那么可以更进一步formulate模型为

$\begin{aligned} \tilde{\mathcal{L}}^{(t)} &=\sum_{i=1}^{n}\left[g_{i} f_{t}\left(\mathbf{x}_{i}\right)+\frac{1}{2} h_{i} f_{t}^{2}\left(\mathbf{x}_{i}\right)\right]+\gamma T+\frac{1}{2} \lambda \sum_{j=1}^{T} w_{j}^{2} \\ &=\sum_{j=1}^{T}\left[\left(\sum_{i \in I_{j}} g_{i}\right) w_{j}+\frac{1}{2}\left(\sum_{i \in I_{j}} h_{i}+\lambda\right) w_{j}^{2}\right]+\gamma T \end{aligned}$

对于一个固定的结构q(x),我们可以算出叶子j的最佳权重为

$w_{j}^{*}=-\frac{\sum\limits_{i \in I_{j}} g_{i}}{\sum\limits_{i \in I_{j}} h_{i}+\lambda}$
以及最优函数值为:

$\tilde{\mathcal{L}}^{(t)}(q)=-\frac{1}{2} \sum_{j=1}^{T} \frac{\left(\sum_{i \in I_{j}} g_{i}\right)^{2}}{\sum_{i \in I_{j}} h_{i}+\lambda}+\gamma T$  （6）

公式（6）可以用来衡量树结构q的质量，类似决策树中的impurity得分。

枚举所有的树结构不现实，best practice是使用一个贪婪算法：

- 从1个叶子结点出发
- 迭代式的添加branch

  令$I=I_{L} \cup I_{R}$, 可以使用下面的公式（7）去评估切分后带来的损失下降

  $\mathcal{L}_{s p l i t}=\frac{1}{2}\left[\frac{\left(\sum_{i \in I_{L}} g_{i}\right)^{2}}{\sum_{i \in I_{L}} h_{i}+\lambda}+\frac{\left(\sum_{i \in I_{R}} g_{i}\right)^{2}}{\sum_{i \in I_{R}} h_{i}+\lambda}-\frac{\left(\sum_{i \in I} g_{i}\right)^{2}}{\sum_{i \in I} h_{i}+\lambda}\right]-\gamma$     (7)

### 切分点查找算法

最朴素的算法，遍历所有特征的所有的切分值，找出最佳的。

目前大部分单机的机器学习库都是采用的这种算法。

![image-20200107161207967](./image-20200107161207967.png)


## 最佳实践

number of trees:太大会造成过拟合，一般用cross validation去挑最好的参数
shrinkage  parameter:迭代步长，通常取0.01或者0.001，较小的lambda会需要很大的B来取得较好的效果。
number  of  split d:通常d=1效果就比较好了。d表示的interation depth，就是交叉项的深度。

## 参考资料
2. [xgboost](https://arxiv.org/pdf/1603.02754.pdf)