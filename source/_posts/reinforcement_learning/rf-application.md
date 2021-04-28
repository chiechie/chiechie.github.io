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
2. 网约车调度：<地址，时间>为state为，每个地址在每个时间的冷门程度是value。
3. 自动生成SQL语句：
4. 神经网络结构搜索(NAS)：
   
    > NAS is closely related to hyperparameter optimization and is a subfield of automated machine learning (AutoML)


### 商品推荐

强化学习可以讲一个很美好的故事：数据驱动的自动化运营。
但是我更好奇的是在实际落地时，强化学习的效果如何？

去问了一位在推荐领域做强化学习的朋友，得到的答案比较让人难过：
强化学习很难调，实际效果没有监督学习好。

系统太复杂又是个黑盒，人hold不住。


> facebook的一个工程师 
> 现在facebook--可能也有很多的其他的公司--在内容推荐技术的很多细分场景上开始应用强化学习，eg消息推送时，以用户的长期活跃为目标，而不是短期打开率为目标。 
> 增强学习的好处是解决了短期反馈和长期活跃之前的换算问题，把两万名算法工程师和他们的实习生们从一部分痛苦的调参工作中解放出来，也让分发策略从长期上达到更高的价值创造。 
> 但一个更加追求用户长期活跃的社交平台就会是环境更健康的么？我对此深表怀疑。（不健康的社交平台，比如很多喷子，反人性的言论）


### 网约车调度

didi的paper是这么写的，不知道生产系统是不是这么用的。

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

- state：截至目前，已经确定的超参。
- action：超参取值
- policy value function：选择某组超参，长期收益，也就是验证集上的准确率。
- 策略网络：一个RNN。

输入向量 $x_t$ 是对上一个超参数 $a_{t−1}$ 做 Embedding 得到的

策略函数为什么不用监督算法拟合（like 贝叶斯优化里面的的高斯过程回归）？
因为超参有可能是离散的，也就是说，损失和奖励对于策略网络参数$\theta$不可微。(btw,贝叶斯优化本身也处理不了离散的超参。)


应用强化学习的代价？需要大量的训练样本，至少上万个奖励，即从初始化开始训练几万个CNN，计算量非常大。

后面学术界提出了更优秀的NAS方法--DARTS 及其变体。


## 参考

1. [DRL-wangshusen中文教材](https://github.com/wangshusen/DRL/tree/master/Notes_CN)
2. [NAS-介绍](https://lilianweng.github.io/lil-log/2020/08/06/neural-architecture-search.html)
3. [NAS-wiki](https://en.wikipedia.org/wiki/Neural_architecture_search)
4. [DARTS: Differentiable Architecture Search-paper](https://arxiv.org/abs/1806.09055)
5. [chiechie: 贝叶斯优化](https://chiechie.github.io/2021/03/24/technology/bayes-optimization//)