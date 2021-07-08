---
title: 树模型4 xgboost介绍
author: chiechie
mathjax: true
date: 2021-04-18 14:59:29
tags:
- 人工智能
- 树模型
categories:
- AI
---

## xgboost

除了Adaboost，GradientBoost，Boosting tree的另外一个代表是xgboost,简称为XGB

相对GBDT用err的一阶泰勒展开做近似，XGB做了更逼近的近似--二阶近似，并且在err中加入了正则项，限制了base estimator的复杂度。

### 损失函数

先看看，xgboost怎么定义ensemble tree的损失函数：

$$\begin{array}{l}{\mathcal{L}(\phi)=\sum\limits_{i} l\left(\hat{y}_{i}, y_{i}\right)+\sum\limits_{k} \Omega\left(f_{k}\right)}\end{array}$$
$$ \\{\Omega(f)=\gamma T+\frac{1}{2} \lambda\|w\|^{2}} $$

- $\hat{y}_{i}=\phi\left(\mathbf{x}_{i}\right)=\sum^{K} f_{k}\left(\mathbf{x}_{i}\right), \quad f_{k} \in \mathcal{F}$
- l是一个可微convex函数
- k表示第k棵树
- i表示第i个样本
- T is the number of leaves in the tree.
- $f_k$表示第k棵树的预测函数，
- q代表树的结构，w代表叶子节点的权重
- $\Omega(f_k)$表示第k棵树的复杂度


### reformulate为一系列子优化问题

上面的优化问题的解，近似求一系列优化问题的解，如下

$\min \mathcal{L}^{(t)}=\min\limits_{f_t}\sum\limits_{i=1}^{n} l\left(\hat{y}_{i}^{(t-1)}+f_{t}\left(\mathbf{x}_{i}\right), y_{i}\right)+\Omega\left(f_{t}\right)$

$\mathcal{L}^{(t)}$表示第t棵树的训练目标

### 将子优化问题做二阶近似

跟GBDT一样，将l进一步简化，使用二阶展开

$\mathcal{L}^{(t)} \simeq\sum\limits_{i=1}^{n}\left[l\left(y_{i}, \hat{y}^{(t-1)}\right)+g_{i} f_{t}\left(\mathbf{x}_{i}\right)+\frac{1}{2} h_{i} f_{t}^{2}\left(\mathbf{x}_{i}\right)\right]+\Omega\left(f_{t}\right)$

这里

- $g_{i}=\partial_{\hat{y}^{(t-1) }}l\left(y_{i}, \hat{y}^{(t-1)}\right)$，即l对$y^{(t-1)}$的一阶导数

- $h_{i}=\partial_{\hat y^{(t-1)}}^{2} l\left(y_{i}, \hat{y}^{(t-1)}\right)$，即l对$y^{(t-1)}$的二阶导数

进一步remove常数项，得到优化目标为：

$min \sum\limits_{i=1}^{n}\left[g_{i} f_{t}\left(\mathbf{x}_{i}\right)+\frac{1}{2} h_{i} f_{t}^{2}\left(\mathbf{x}_{i}\right)\right]+\Omega\left(f_{t}\right)$

### 基于样本集重新定义损失函数

更进一步，将$I_{j}=\left\{i | q\left(\mathbf{x}_{i}\right)=j\right\}$定义为叶子j对应的特征子空间 那么优化目标可以更进一步formulate为：

$$\min\limits_{\omega, q} \sum_{i=1}^{n}\left[g_{i} f_{t}\left(\mathbf{x}_{i}\right)+\frac{1}{2} h_{i} f_{t}^{2}\left(\mathbf{x}_{i}\right)\right]+\gamma T+\frac{1}{2} \lambda \sum_{j=1}^{T} w_{j}^{2} $$
$$=\sum_{j=1}^{T}\left[\left(\sum_{i \in I_{j}} g_{i}\right) w_{j}+\frac{1}{2}\left(\sum_{i \in I_{j}} h_{i}+\lambda\right) w_{j}^{2}\right]+\gamma T $$

也就是说，在构造第t个base estimator时，只需满足该estimator是该优化问题的最优解即可。

还是回到cart，确定一棵树，首先要确定分支，其次要确定每个叶子的预测值。
假设已经确定好了第t颗树的分支，即$I_{j}$已经确定，那么最优的叶子预测值$\omega_i$即为
$$w_{j}^{*}=-\frac{\sum\limits_{i \in I_{j}} g_{i}}{\sum\limits_{i \in I_{j}} h_{i}+\lambda}$$

将$w_{j}^{*}$带入上面的目标函数，得到等价的目标函数：

$\min\limits_q  -\frac{1}{2} \sum\limits_{j=1}^{T} \frac{\left(\sum_{i \in I_{j}} g_{i}\right)^{2}}{\sum_{i \in I_{j}} h_{i}+\lambda}+\gamma T$  （6）

公式（6）可以用来衡量树结构q的质量，类似决策树中的impurity得分。根构造决策树算法一样，使用贪婪方法迭代式构造分支，衡量splitting质量好坏用下面的表达式，即计算切分后纯度的提升 或者 不纯度的下降： 
  $$\mathcal{L}_{s p l i t}=\frac{1}{2}\left[\frac{\left(\sum\limits_{i \in I_{L}} g_{i}\right)^{2}}{\sum\limits_{i \in I_{L}} h_{i}+\lambda}+\frac{\left(\sum\limits_{i \in I_{R}} g_{i}\right)^{2}}{\sum\limits_{i \in I_{R}} h_{i}+\lambda}-\frac{\left(\sum_{i \in I} g_{i}\right)^{2}}{\sum_{i \in I} h_{i}+\lambda}\right]-\gamma$$  

其中，$I=I_{L} \cup I_{R}$

### 切分点查找算法

最朴素的算法，遍历所有特征的所有的切分值，找出最佳的。

目前大部分单机的机器学习库都是采用的这种算法。

![image-20200107161207967](./image-20200107161207967.png)


## 最佳实践

- number of trees:太大会造成过拟合，一般用cross validation去挑最好的参数
- shrinkage parameter:迭代步长，通常取0.01或者0.001，较小的lambda会需要很大的B来取得较好的效果。
- number of split d:通常d=1效果就比较好了。d表示的interation depth，就是交叉项的深度。

## 参考资料
1. [xgboost](https://arxiv.org/pdf/1603.02754.pdf)