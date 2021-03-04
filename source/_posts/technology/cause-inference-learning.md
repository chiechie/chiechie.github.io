---
title: 因果推断补课
author: chiechie
mathjax: true
date: 2021-03-04 23:23:02
tags:
- 因果推断

categories: 技术类

---


# Prediction 和 causation 的区别



# 因果问题分为因果推断和因果发现
- 一种是因果推断（causal inference），比如给定两个变量 X、Y，希望找到一个衡量它们之间因果关系的参数 theta；
- 另一种是 causal discovery，即给定一组变量，找到他们之间的因果关系。对于后面这种 causal discovery，notes 里面说它在统计上是不可能的。

# 数据有两种产生途径

数据有两种产生途径：

- 一种是通过有意控制、随机化的实验得到的；
- 一种是通过观测数据得到的。
- 控制实验得到的数据（例如拨测）能够直接做因果推断；
- 后一种方式需要另外知道一些先验知识，才能在上面做因果推断。


# 因果关系描述的数学语言

对因果关系描述的数学语言：一种是 counterfactuals，一种是 causal graph；还有一种和 causal graph 相近的 structural equation models。


一个简单的例子『睡眠超过 7 小时的人』（X）『生病少』（Y)，只是代表 X 和 Y 之间有关联性，并不代表如果强制一个人睡眠超过 7 小时，ta 就能够生病少。因为可能『身体好的人』容易『睡眠超过 7 小时』，同时 ta 也『生病少』；但是一个本来身体不好的人，强制 ta 睡眠多，ta 可能也生病不会少。


因果关系可以从随机化的实验中得到；但是很难从观察到的数据中得到。


## Counterfactuals

考虑一个 treatment X，和一个 outcome Y。我们能观察到的是一些数据 [公式] ，但是我们无法知道如果对于某一个数据点 [公式] ，如果改变 X 的值，Y 会怎么变。这件事情就叫做 counterfactual。Notes 里面给了一个图（下图），从数据上看，X 和 Y 是正相关的，但其实对于每一个 样本来说，如果增大 X，会引起 Y 的减小。这一点最开始看的时候并不好理解。举一个例子。研究航空公司票价（X）对销量（Y）的影响，显然，对于某一个客户来说，增加票价（X 变大）会降低客户购买意愿，即使得销量将达（Y 变小）。但是实际中的情况是，在节假日人们出行意愿大导致销量高（Y 大），定价也会相应变高（X 大），从而从数据上看，形成左边图的情形。


## 因果图

因果图是一个DAG，表明各变量之间的联合概率分布。

$$p\left(y_{1}, \ldots, y_{k}\right)=\prod p\left(y_{j} \mid \operatorname{parents}\left(y_{j}\right)\right)$$

下面举例说明，在给定一个因果图之后，如何做因果推断。考虑下面一个因果图，目标是求$p(y \mid \operatorname{set} X=x)$

- 首先，可以看出该 causal graph 提供的信息为 
$p(x, y, z)=p(z) p(x \mid z) p(y \mid x, z)$

- 接下来，由于考虑的是给定 X的影响，因此构建一个新图$G_{*}$,
移除掉所有指向 X 的边，得到新的联合概率分布
$$p_{*}(y, z)=p(z) p(y \mid x, z)$$


- 最后，该概率分布下的数值就是因果推断的结果:
	$p(y \mid \text { set } X=x) \equiv p_{*}(y)=\int p_{*}(y, z) d z=\int p(z) p(y \mid x, z) d z$


## 因果图 和 概率图 的区别

举例说明，比如下雨（Rain，R）和湿草坪（Wet Lawn，W）是不相互独立的
对于下两种 DAG，它们都是合理的 probability graph。即对于任意的联合概率分布 $p(w, r)$，都可以写成 $p(w) p(r \mid w)$ 或者 $p(r) p(w \mid r)$.
但显然下雨是因、草坪湿是果，只有左边的图才是正确的 causal graph。

## Causal Discovery 是不可能的
总存在一个 faithful 的分布使得在样本足够多的时候，产生足够大的 type I error。

## 参考

1. [zhihu-翻译](https://zhuanlan.zhihu.com/p/88173582)
2. [英文原文](http://www.stat.cmu.edu/~larry/=sml/Causation.pdf)