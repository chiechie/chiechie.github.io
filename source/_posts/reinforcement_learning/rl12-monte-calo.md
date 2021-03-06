---
title: 强化学习12 蒙特卡洛树搜索
author: chiechie
mathjax: true
date: 2021-04-30 16:58:05
tags:
- 强化学习
- 人工智能
categories:
- 强化学习
---

# 总结

1. 蒙特卡洛树搜索（MCTS）是一种基于模型的强化学习方法。
2. 蒙特卡洛树搜索比价值学习和策略学习更难。
3. AlphaGp是一个围棋AI，依靠MCTS做决策，决策过程中需要策略网络和价值网络辅助。
4. AlphaGo真正跟人下棋的时候，做决策的不是策略网络或者价值网络，而是MCTS。
5. MCTS不需要训练，可以直接做决策，训练策略网络和价值网络的目的是辅助MCTS。
   
## MCTS的基本思想

1. 高手是怎么下棋的？高手一般都会往前多看几步。
2. 高手下棋，一般是动态看待局势，确保自己几步之后占据优势。如果只根据当前格局做判断，不考虑长远，肯定赢不了高手。
3. 同理，AI下棋也应该跟高手一样，多往前看几步，枚举所有可能发生的情况，从而判断当前执行什么动作的胜算最大，这样远浩宇用策略网络输出一个动作。
4. AlphaGo每走一步，就要用MCTS做成千上万次模拟，基本思想如下：假设当前有3种可选的动作，每次模拟从三种动作中选出一种，然后将游戏进行到底，从而拿到最终结果。
重复成千上万次模拟，统计每种动作的胜负概率，发现胜率分别是48%、56%、52%。那么 AlphaGo 应当执行第二种动作，因为 它的胜算最大。然而实际做起来还有很多难点需要解决。

## MCTS的四个步骤

MCTS每次模拟都要经历四个步骤：选择（selection），扩展（expansion），估值（evaluation），回溯（backup）

### 选择

1. 观察当前局势，找出符合规则的所有走位中最有胜算的那部分走位。
如何评估走位优劣？综合两方面：
   
1. 看走位的胜率，就是动作价值
2. 看策略网络给a的评分

### 扩展

接下来，ALphaGo需要考虑：对方会怎么走？只能靠猜，使用"推己及人"的方式。使用策略网络模拟对手的策略，
策略网络站在对手角度，根据观测到的棋局，给出多个走位的概率，ALphaGo采样一个动作。
这个时候对手落下的子就是ALphaGo的新状态了。

上面的所有的走位都不是真实发生的，而是在AlphaGo的脑袋中推演的，

ALphaGo专门执行推演的模块就叫模拟器（就是环境，牛逼的很，自己构建出与现实平行的虚拟世界），

在模拟器中，ALphaGo每执行一个动作，模拟器就会返回一个新的状态。

如何搭建一个好的模拟器？关键在于构建的正确的状态转移函数，如果其跟事实偏离太远，那么模拟器就是一个废物。

AlphaGo如何构建好的模拟器呢？他利用了围棋的对称性：即我方视角的状态转移，等价于对方视角的真实策略--可以复用已经训练好的策略网络。

> 有点类似理性市场假设，每个人都是同样聪明的人，市场不存在套利机会。

在围棋场景，搭建模拟器很简单，但是其他场景就没有这么简单了。

比如机器人、无人车等应用，状态转移的构造需要物理模型，要考虑到力、运动、以及外部世界的干扰。
如果物理模型不够准确，导致状态转移函数偏离事实太远，那么 MCTS的模拟结果就不可靠。


### 求值

AlphaGo在模拟器中交替落子，直到分出胜负，把结果记为r.但是r只是众多平行世界中的一个，把r就作为评判第一个走位的依据，未免过于随机。

有没有更加稳定的方法？AlphaGo的解决方案，把r与价值网络对下个状态的评价进行求和，得到了下一个状态的价值。

在实际实现的时候，AlphaGo在这一步训练了一个更小的策略网络。

为什么只在求值这一步使用小网络呢？因为这一步是连哥哥策略网络交替落子，并且通常要走一两百步，导致这步成为瓶颈。

用小的策略网络代替大的策略网络，可以大幅加速MCTS


### 回溯

对于下一步的每个状态，通过模拟都能得到多条记录--代表该状态的价值。

把假想走位的所有状态的所有价值记录，求平均，就能得到假想走位的价值。

将这个候选走位带回第一步，更新每个走位的分数，选出最高的，进行下一轮模拟

$$ \operatorname{score}(a) \triangleq Q(a)+\frac{\eta}{1+N(a)} \cdot \pi(a \mid s ; \boldsymbol{\theta}), \quad \forall a $$


## 决策

一轮 选择->扩展->扩展->回溯只是执行了单次模拟。

实际上做一次决策之前，需要做成千上万次模拟。

每次模拟时，更新每个候选走位的score，选择最高分score，在模拟器执行。

经过成千上万次模拟后，最经常被选中的走位，就作为最终的决策依据。

注意，每次在真实世界走完一步，上一步存的分数都要清零。

总结下，AlphaGo下棋非常暴力：每走一步，都要在虚拟世界模拟几万局。

计算量是机器战胜柯洁的秘密武器。



# 参考
1. [HotSpot-英文paper](https://netman.aiops.org/wp-content/uploads/2018/12/sunyq_IEEEAccess2018_HotSpot.pdf)
2. [HotSpot-中文](https://mp.weixin.qq.com/s/Kj309bzifIv4j80nZbGVZw)