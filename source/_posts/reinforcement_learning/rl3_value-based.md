---
title: 强化学习3 基于价值的强化学习方法
author: chiechie
mathjax: true
date: 2021-04-21 20:30:11
tags:
- 强化学习
- 人工智能
categories:
- 强化学习
---


## 几个问题：

- 价值 指的是什么？
- 怎么使用深度学习技术来求解这个问题？难点在哪里？
- 跟DQN并列的概念还有哪些？
- DQN网络的输入输出是什么？中间结构是什么？
- DQN怎么更新参数？

## 回答

- 「价值」 指的是 **最优动作价值函数**，即如果后续action都遵循**最优策略**，那么当前的<state，action>的得分。
- **最优动作价值函数** 相当于 一个 先知。
- 回到强化学习要解决的问题本身（MDP）--agent的目标是 打赢游戏（最大化total reward），那么我们将这个目标拆解为两个步骤，
    - 1. 找到最优动作价值函数
    - 2. 在最优价值函数的指引下作出最优决策
- DQN的想法就是，使用一个深度神经网络拟合 **最优动作价值函数**
- 难点在于，谁都不知道 先知（**最优价值函数**）长什么样子（少数情况，如玩游戏可能可以知道）
- DQN的输入是state，输出是所有action对应的分数
    - ![价值网络](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FfsQADMgSRa.png?alt=media&token=18bf844e-aa59-4016-85d6-8cdfc801a9ce)
    - ![策略网络](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FGqFFfS975r.png?alt=media&token=71ba382a-432c-4c00-8759-692d84c03f3d)
- 怎么更新模型参数
    - 可以把一整局游戏玩完，搜集到反馈信号（total reward）再去更新DQN的参数。
    - 也可以玩一步，更新一下参数，就是TD算法

## 时序差分算法（TD算法）

- TD error:  对于上一步<$s_{t-1}, a_{t-1}$>的reward， 模型估值-真实值
    - 模型作出的估计（模型对上一个状态估计的价值 - 模型对当前状态估计的价值） -  真实 reward （来自 environment）
- TD target： 对于上一步<$s_{t-1}, a_{t-1}$>的长期value，
    - 真实reward（来自 environment） + 折扣* 模型对当前状态估计的价值
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FROMmZ-LTGx.png?alt=media&token=dae7cba7-84e1-4e7f-8638-60eaa2b1c166)
- ![时序差分算法](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FEuWKISitjj.png?alt=media&token=0427aa38-bad9-46f3-98bd-a1eb051df5cd)

## 没想清楚

- 更新模型参数时，如果 把一整局游戏玩完，搜集到反馈信号（total reward）再去更新DQN的参数，那么，中间的决策过程怎么放到训练样本里面来？玩多次么？怎么搜集「最优价值」的反馈信号？
- Q函数 和 reward 函数 是不是可以相互转化？--是这样把？
    - Q 函数  ==  reward （来自 environment）+ 策略（来自 agent）
    - Q * 函数 == reward （来自 environment）+ 最优策略（来自 agent）


## 时序差分算法的其他资料
1. Sutton and others: A convergent O(n) algorithm for off-policy temporal-difference learning with linear function approximation. In NIPS, 2008.
2. Sutton and others: Fast gradient-descent methods for temporal-difference learning with linear function approximation. In ICML, 2009.

