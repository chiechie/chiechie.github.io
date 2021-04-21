---
title: 强化学习的应用
author: chiechie
mathjax: true
date: 2021-04-21 08:22:04
tags:
- 强化学习
- 人工智能
- auto ml
- NAS
categories:
- 技术
---

## 应用场景
1. 商品推荐: 每个用户的信息作为state，推荐的商品是action，长期看用户的点击率/浏览时长/消费额reward
2. 网约车调度：定义state为 <地址，时间>，价值函数value为每个地址在每个时间的冷门程度
3. 自动生成SQL语句：
4. 神经网络结构搜索(NAS)：
   
    > NAS is closely related to hyperparameter optimization and is a subfield of automated machine learning (AutoML)



### 网约车调度



### 神经网络结构搜索（NAS）

就好像下一盘棋一样，一局玩完可能要很久，所以每走一步，都可以，判断当前这步棋走的水平如何。

但是只有当一盘都下完了，我们才真的知道具体的reward。

NAS是auto-ml的子领域，为了让机器去设计神经网络。
2017年谷歌发了一篇paper，使用基于强化学习的思路，去设计网络结构。
具体思路是这样的，假设要设计一个20层的CNN，超参个数为20*3，即每一层卷积，需要设置：
- 卷积核数量
- 卷积核大小
- 步长大小


怎么定义为强化学习的问题呢？

- state为：截至目前，已经确定的超参。
- action：60个超参数中的一个的取值。
- policy value function为：选择某组超参，长期收益，也就是验证集上的准确率。
- 策略网络，也就是用来逼近策略价值函数，是一个RNN。

输入向量 $x_t$ 是对上一个超参数 $a_{t−1}$ 做 Embedding 得到的/
高斯过程回归算法--代理函数，超参和效果的映射函数，实际上就等价于，策略价值函数的特殊情况。（只对所有的超参都给定时，才能给出价值）

这个本质上就是一个有监督的方法，

为什么强化学习NAS不采用这个简单的有监督方法，而要把这个问题拆解得更稀碎，不等所有超参确定完，而是确定一个超参，就要评估一下可行性呢？


应用强化学习的代价是需要大量的训练样本，至少上万个奖励，即从初始化开始训练几万个 CNN。 这种强化学习 NAS 方法的计算量非常大。在这种强化学习 NAS 方法提出之后，很快就 有更好的 NAS 方法出现，无需使用强化学习。有兴趣的读者可以了解一下 DARTS 方法 [68];DARTS 及其变体是比较实用的 NAS 方法。

这里有一个很有意思的问题，为什么不用监督算法拟合策略函数呢？
因为超参有可能是离散的，也就是说，损失和奖励对于策略网络参数$theta$不可微，这个时候，怎么拟合呢？想想贝叶斯优化里面，对正整数类型的超参，别扭的处理方法。


## 参考
1. [DRL-wangshusen中文教材](https://github.com/wangshusen/DRL/tree/master/Notes_CN)
2. [NAS-介绍](https://lilianweng.github.io/lil-log/2020/08/06/neural-architecture-search.html)
3. [NAS-wiki](https://en.wikipedia.org/wiki/Neural_architecture_search)
4. [DARTS: Differentiable Architecture Search-paper](https://arxiv.org/abs/1806.09055)
5. [chiechie: 贝叶斯优化](https://chiechie.github.io/2021/03/24/technology/bayes-optimization//)