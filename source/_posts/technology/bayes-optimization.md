---
title: 自动调参之贝叶斯优化
author: chiechie
mathjax: true
date: 2021-03-24 10:30:05
tags:
- auto-ml
- 贝叶斯
categories:
- 技术
---

> 「贝叶斯优化」技术是求解黑盒最优化问题的一种方法，该技术可拓展到自动调参的场景中。
> 
> 接下来从直觉的角度，描述「贝叶斯优化」的思路
>
> 效用函数是我对acquisition function的理解，得「意」可忘形，没必要扣字眼。


## 背景

什么是「黑盒优化」？

求解 
$$ \max _{x \in \mathcal{X}} f(x)$$
但是$f(x)$的内部运行机制，我们一无所知，即$f(x)$对我们而言就是一个黑盒


## 技术方案概述
「自动调参」（或者「超参转化」）本质上就是在求解一组最优的超参数，使得这组超参在样本集上的效果最好。
这个需求可以抽象为一个典型的黑盒优化问题，即
$$ \max _{x \in \mathcal{X}} f(x)$$

- x代表一组超参数的取值，是一个向量，每个元素代表一个超参数，一个算法可能包含成千上万个超参数。
- f(x)代表一组超参数取某个值时，模型的准确率）。

模型越复杂，超参数就越多（即x里面的元素越多），调参的工作就越依赖经验，并且也非常耗时。


调参的过程中存在的痛点：极其依赖算法理论和工程经验，调试过程繁琐。那么有没有什么技术能将人力解放出来呢？
有的，目前比较常用的思路就是贝叶斯优化算法（Bayes Optimization），它的大概思路是这样的：

1. 收集样本<x,f(x)>，x是一组超参，f(x)表示这组超参下，训练出来的模型的准确率。
2. 使用样本<x,f(x)> 拟合一个高斯过程回归器（GaussianProgressRegressor）
3. 基于高斯过程回归器构造一个acquisition函数,用来衡量超参数空间中，当前每个区域的潜在收益。一般来说acquisition函数就是均值加上n倍方差（Upper condence bound算法）。
4. 搜索acquisition函数的最大值，即最值得探索的点（Next Best Guess）。
5. 将找到Best Guess（一组超参）带入原来算法，重新训练一遍，并记录下此时模型的准确率。
6. 将5得到的新样本数据加入样本集中，回到第二步，进行下一个迭代，循环往复，直到模型的准确率没有提升或者达到最大的迭代次数。


## 技术方案概述-可视化版

![贝叶斯优化](img.png)

https://distill.pub/2020/bayesian-optimization/

上图中，横轴是超参，
上面图的纵轴：超参数取不同值时，模型的准确率。
	黑色的线代表拟合出来的均值线，灰色区域代表拟合出来的置信区间
	红色的线代表超参和准确率的真实关系
                    红点代表下一次要探索的点
下面图的纵轴：超参数取不同值时，采集函数的值，取值越大，表示该区域越有可能存在最优超参。


## 技术方案细节-专业版


调参的过程中存在的痛点：极其依赖算法理论和工程经验，调试过程繁琐。那么有没有什么技术能将人力解放出来呢？
有的，目前比较常用的思路就是贝叶斯优化算法（Bayes Optimization），它的大概思路是这样的：

为了求解该问题：
$x^{*}=\arg \max _{x \in \mathcal{X}} f(x)$

1. 收集样本$<x_i,f(x_i)>$，x是一组超参，f(x)表示这组超参下，训练出来的模型的准确率。
2. 使用样本$<x_i,f(x_i)>$拟合一个高斯过程回归器（GaussianProgressRegressor）$\hat G$
3. 基于$\hat G$构造一个效用函数。效用函数是用来衡量当前的超参数组成的空间中，探索每个区域的潜在收益或者效用（注意我们的目标是，找到$x_\star$，使得f(x)取到最大值）。 最简单的acquisition function就是均值加上n倍方差（Upper condence bound算法）。
4. 利用效用函数,我们去找最值得探索的点（Next Best Guess）
5. explore，将找到的最佳的超参组合带入模型重新训练一遍，并记录下此时模型的准确率。$<x_j,f(x_j)>$
6. 将5得到的新样本数据加入样本集中，回到第二步，进行下一个迭代，循环往复，直到模型的准确率没有提升。


## 参考资料

1. [贝叶斯优化-通俗版-tobe-知乎](https://zhuanlan.zhihu.com/p/29779000)
2. [贝叶斯优化-技术版—Dai Zhongxiang-知乎](https://zhuanlan.zhihu.com/p/76269142)
3. [贝叶斯优化-可视化版-distill](https://distill.pub/2020/bayesian-optimization/)
4. [开源工具-Aavisor-github](https://github.com/tobegit3hub/advisor)