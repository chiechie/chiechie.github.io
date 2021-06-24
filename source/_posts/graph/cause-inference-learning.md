---
title: 因果发现和因果推断
author: chiechie
mathjax: true
date: 2021-03-04 23:23:02
tags:
- 因果推断
- 因果分析
- 贝叶斯
- 复杂网络
categories: 
- 人工智能
---


# 因果推断和因果发现

因果问题分为两种：一种是因果发现；一种是因果推断

- 因果发现（causal discovery）：给定一组变量，找到他们之间的因果关系，这个在统计上是不可能的。
- 因果推断（causal inference）：给定两个变量，找到一个衡量它们之间因果关系的参数；

# 两种数据产生途径

数据有两种产生途径：

- 一种是通过有意控制、随机化的实验得到的:例如拨测,能够直接做因果推断
- 一种是通过观测数据得到的: 需要另外知道一些先验知识，才能做因果推断。


# 用数学语言表达因果模型

因果模型有三种表达方式：

- 反事实（counterfactuals）
- 因果图（causal graph）
- 结构方程模型（structural equation models）


两个变量的因果关系可以从随机化的实验中得到；但是很难从观察到的数据中得到。


## 反事实（Counterfactuals）

反事实（counterfactual）：我们可以观察 $\left\{\left(X_{i}, Y_{i}\right)\right\}$ ，但是我们不知道如果对于某一个数据点 $\left(X_{i}, Y_{i}\right)$ ，如果改变 X 的值，Y 会怎么变。

下图，从数据上看，X 和 Y 是正相关的，但其实对于每一个 样本来说，如果增加X，会引起 Y 的减小。

举一个例子。研究航空公司票价（X）对销量（Y）的影响，显然，对于某一个客户来说，增加票价（X 变大）会降低客户购买意愿，即使得销量将达（Y 变小）。但是实际中的情况是，在节假日人们出行意愿大导致销量高（Y 大），定价也会相应变高（X 大）。

![反事实举例](./img.png)

## 因果图（causal graph）

因果图是一个DAG，表明各变量之间的联合概率分布。

$p\left(y_{1}, \ldots, y_{k}\right)=\prod p\left(y_{j} \mid \operatorname{parents}\left(y_{j}\right)\right)$

下面举例说明，在给定一个因果图之后，如何做因果推断。
考虑下面一个因果图，目标是求$p(y \mid \operatorname{set} X=x)$

- 首先，从因果图中得到信息： 
$$p(x, y, z)=p(z) p(x \mid z) p(y \mid x, z)$$

- 接下来，构建一个新图$G_{*}$, 移除掉所有指向 X 的边，得到新的联合概率分布:

$$p_{*}(y, z)=p(z) p(y \mid x, z)$$


- 最后，该概率分布下的数值就是因果推断的结果:
  
  $$p(y \mid \text { set } X=x) \equiv p_{*}(y)=\int p_{*}(y, z) d z=\int p(z) p(y \mid x, z) d z$$


## 因果图 和 概率图 的区别

因果图的箭头代表了因果关系方向，但是概率图（贝叶斯网络）没有这个要求，他不要求箭头表达因果关系。。

举例说明，比如下雨（Rain）和湿草坪（Wet Lawn）这两个事件经常一起出现。
下面两个DAG都是概率图，但是只有左边才是因果图。

![right_causal_graph](right_causal_graph.png)



# 因果发现是不可能的

总存在一个 faithful 的分布使得在样本足够多的时候，产生足够大的 type I error。

# 参考

1. [zhihu-关于因果推断公开课翻译](https://zhuanlan.zhihu.com/p/88173582)
2. [英文原文](http://www.stat.cmu.edu/~larry/=sml/Causation.pdf)