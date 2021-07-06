---
title: 树模型3 RandomForest，Adaboost和GBDT
author: chiechie
mathjax: true
date: 2021-05-15 10:39:45
tags:
- 机器学习
- 树模型
- 人工智能
categories:
- AI
---

> boosting tree的代表是gbdt，以及一系列变形xgboost,catboost,下面重点介绍一下xgboost


# 总结

1. Bagging是一种数据采样的方法，Blending是一种模型混合的方法.
2. Blending的方法有几个：投票/线性blending/stacking。
    - 如果每个base estimator一样重要，就对他们采用uniform的aggregation方式。
    - 如果有的base estimator特别重要，可以将estimator当成特征转换器使用，然后将不同estimator的输出做一个线性映射，得到最终的结果，这个也叫线性blending。
    - 如果想让这个aggregation更加复杂，也就是说，想要这个模型实现，在不同的condition下，不同的estimator发挥的重要性不一样，这个时候就要基于不同estimator的输出构建一个非线性映射。这个也叫stacking
3. 投票追求的是公平稳定，其他两种追求高效。
4. 也可以一边构建base estimator，一边决定模型blending的方法。
    - uniform aggregation的代表是Bagging Tree/随机森林
    - 通过改变样本权重的方式得到不一样的base estimator，一边根据他们的表现决定给每个estimator多少票。
    - 在不同条件下找到最优的base estimator的代表算法是决策树，回想一下，在决策树里面，不同的路使用的是不同的叶子（在决策树的场景中，一个叶子就是一个base estimator）。
    - gradient boost是一种线性blending对模型汇总的方法，只不过，构建新的base estimator时，不是像adboost那样重新对样本赋予权重，而是，直接去最小化残差，然后根据学的结果，赋予当前base estimator一个权重。
    - boosting-like的算法最受到欢迎，即ada boosting和gradient boost
    
   ![img_2.png](img_2.png)
5. 为什么对模型做aggregation之后，效果变好了？相当于对特征做了非线性变换，整体表达能力更强；多个estimator求共识，相当于做了正则化，模型效果更稳定。
6. 不同的blending方法中，有的方法解决overfitting，有的适合解决underfitting。

# 随机森林，Adaboost和 GBDT

## 随机森林

1. Bagging是什么？通过bootstrap的方式抽样得到多分样本。Bagging的特点是，可以降低整体的模型预测的不稳定性，通过让多个base estimator投票或者取均值的方式。
2. decision tree非常不稳定，训练数据稍微有一点变化，分支条件就会改变，从而整个树都长得不一样了。
3. Bagging + decision tree综合起来构建随机森林。
    ![img_1.png](img_1.png)
4. Bagging的部分可以并行化，从而可以更加有效地处理更大的训练数据。
5. 构建base estimator时，除了对样本采样，还可以对特征采样，随机森林的原始论文作者建议，在构建cart的每个分支条件时，都随机采样一个特征子空间，在这个子空间中找一个最优的分割点。
6. rf的作者还建议，可个对特征作随机组合，即构建投影矩阵，将原始的训练数据映射到一个低维的特征空间。



## Adaboost

![adaboost](./img.png)

boosting中有三个重要的参数：
- 树的棵树
- $\lambda$: 学习速率
- 每一棵树的最大splits个数


## GBDT 

1. GBDT是将一个优化问题，拆解成一系列子优化问题，每个子优化问题要解决的问题是，在当前学习成果的基础上查漏补缺，从而更加逼近target variable。即当前的base estimator的优化目标是最小化target和截止当前的预测值之间的差，也叫residual。
   ![img_4.png](img_4.png)
   h是当前的base estimator，$\eta$表示权重
2. 怎么求这个优化问题？先求里层的优化问题，再求外层的优化问题。里面的优化问题是将err对当前prediction做一阶泰勒展开，其中一阶项（h，待学习的目标）的系数就是error相对当前prediction的一偏导数，也就是梯度。这里有一个问题，如果我已经知道要做regression，也就是当前的目标函数是关于h的二次函数，我直接求最优解不就好了，即令损失函数关于h的导数为0？其实是可以的，但是GBDT的野心很大，想推导出，任意形式的err对应的最优解h，随意下面的一阶近似方法是更具备通用性的。（想象一下如果err是absolute loss或者像交叉熵这种非convex，就不适用了。btw，xgboost是使用err的二阶泰勒展开去作为err的近似）
   
   > 回忆一下一阶泰勒展开：
   > $$f(x + \Delta x) = f(x) + \Delta x \partial f_x $$
   
   ![img_5.png](img_5.png)
3. 里层的优化问题求解方法类似拉格朗日乘子法，先将h的大小限制到一定的范围内。然后经过各种变形之后，发现直接拟合residual就好。
   ![img_7.png](img_7.png)
4. 如何使用GBDT做回归？将上面的error 设置为squared loss。
6. adaboost是要根据不同样本的权重来找一一个拟合的最好的小g; GBDT是找一个能拟合当前残差最好的小g。



# 参考资料
1. [Random Forest Algorithm-Hsuan-Tien Lin](https://www.youtube.com/watch?v=ATM3sH0D45s&list=RDCMUC9Wi1Ias8t4u1OosYnHhi0Q&index=9)
2. [Adaptive Boosting linxuantian](https://www.csie.ntu.edu.tw/~htlin/mooc/doc/208_present.pdf)
3. [youtube-gbdt](https://www.youtube.com/watch?v=2xudPOBz-vs)
4. [xgboost](https://arxiv.org/pdf/1603.02754.pdf)
5. [An Introduction to Statistical Learning](https://static1.squarespace.com/static/5ff2adbe3fe4fe33db902812/t/6062a083acbfe82c7195b27d/1617076404560/ISLR%2BSeventh%2BPrinting.pdf)
7. [Gradient Boosted Decision Tree :: Gradient Boosting](https://www.youtube.com/watch?v=F_EuNXhS9js&list=RDCMUC9Wi1Ias8t4u1OosYnHhi0Q&index=4)
5. [Blending and Bagging :: Bagging (Bootstrap Aggregation)- Tien Lin](https://www.youtube.com/watch?v=3T1mdvzRAF0&list=RDCMUC9Wi1Ias8t4u1OosYnHhi0Q&index=6)
