---
title: 强化学习4 基于策略的强化学习方法
author: chiechie
mathjax: true
date: 2021-04-22 20:33:30
tags:
- 强化学习
- 人工智能
categories:
- 强化学习
---



# 总结

1. 强化学习的方向之一是基于策略学习的方法，即通过求解优化问题，得到一个最优的策略函数。
2. 可以使用一个神经网络来拟合这个策略函数，这个神经网络也叫策略网络。策略网络的输入是一个state，输出是每个action对应的概率向量。策略网络参数的初始值是随机的，利用agent跟环境之间的互动，收集state，action，reward，然后去更新策略网络参数。

3. 策略学习的目标函数是状态价值函数关于所有状态的期望: 如果策略越好，目标函数就越大
   $$
   \max\limits_{\theta} J(\boldsymbol{\theta})=\mathbb{E}_{S}\left[V_{\pi}(S)\right]
   $$

4. 目标函数关于策略的梯度记为「策略梯度」，可以使用梯度上升的策略去更新策略网络的参数
   $$
   \boldsymbol{\theta}_{\text {new }} \leftarrow \boldsymbol{\theta}_{\text {now }}+\beta \cdot \nabla_{\boldsymbol{\theta}} J\left(\boldsymbol{\theta}_{\text {now }}\right)
   $$

5. 又根据策略梯度证明定理，策略梯度等价于
   $$
   \nabla_{\boldsymbol{\theta}} J(\boldsymbol{\theta})=\mathbb{E}_{S}\left[\mathbb{E}_{A \sim \pi(\cdot \mid S ; \boldsymbol{\theta})}\left[Q_{\pi}(S, A) \cdot \nabla_{\boldsymbol{\theta}} \ln \pi(A \mid S ; \boldsymbol{\theta})\right]\right]
   $$

6. 策略梯度中包含两部分，动作价值函数和策略函数的梯度，策略函数的梯度好算，动作价值函数不好算，有两种方法可以对动作价值函数做近似，分别是reinforce和actor-critic。

   1. reinforce：用实际观测的回报u去近似状态价值函数
   2. actor-critic方法：用神经网络去近似动作价值函数

7. reinforce计算Qvalue的方法

   1. 从t时刻开始，agent每次完成一局（episode）游戏，观测并且记录所有reward，计算return，将其作为Q的一个观测值。
   2. 将Q和对策略网络使用BP算出来的梯度相乘，得到策略梯度，更新策略网络参数。

8. reinforce 是一种在线策略 (On-Policy) 方法，要求行为策略 (Behavior Policy) 与目标策略 (Target Policy) 相同，两者都必须是策略网络 $\pi( a| s; \theta_{now} )$，其中$ \theta_{now} $是策略网络当 前的参数。所以经验回放不适用于 REINFORCE。

9. Actor-Critic 方法用神经网络近似Q

10. A2C方法 的价值网络与DQN结构相同，但是意义不同，训练的算法也不同，

    1. A2C的价值网络是对动作价值函数的近似DQN的价值网络是对最优动作价值函数$Q^{\star}$的近似
    2. 对A2C的价值网络训练使用的是SARSA算法，它是一种在线策略（on-policy），不能用经验回放（replay buffer）。DQN的训练使用的是Q-learning算法，它属于离线策略（off-policy），可以使用replay buffer

11. 训练策略网络：策略网络接受价值网络的评价，调整自己的权重，

12. 训练价值网络：价值网络也需要持续提升自己，不然评委固步自封，这样策略网络只会投其所好，不会学到真正的好的策略。怎么训练价值网络呢？使用SARSA算法更新价值网络参数，即用观测到的奖励$r_t$来校准q，

13. Actor-Critic 具体的训练流程如下：

    1. 观测到当前状态 $s_t$，根据策略网络预测:$a_t ∼ \pi( · | s_t; \theta_{now})$，并让agent执行动作$a_t$。

    2. 从环境中观测到奖励 $r+t$ 和新状态 $s_{t+1}$。

    3. 让策略网络预测：$\tilde a_{t+1} ∼ π( · |s_{t+1}; \theta_{now})$，但不执行

    4. 让价值网络预测: t和t+1时刻

    5. 计算TD误差$\delta_{t} $

    6. 更新价值网络，损失函数为TD误差平方：

       $$\boldsymbol{w}_{\mathrm{new}} \leftarrow \boldsymbol{w}_{\mathrm{now}}-\alpha \cdot \delta_{t} \cdot \nabla_{\boldsymbol{w}} q\left(s_{t}, a_{t} ; \boldsymbol{w}_{\mathrm{now}}\right)$$

    7. 更新策略网络:

       $$\boldsymbol{\theta}_{\mathrm{new}} \leftarrow \boldsymbol{\theta}_{\mathrm{now}}+\beta \cdot \widehat{q}_{t} \cdot \nabla_{\boldsymbol{\theta}} \ln \pi\left(a_{t} \mid s_{t} ; \boldsymbol{\theta}_{\mathrm{now}}\right)$$

       

    

# 附录

## 问题

- 基于策略学习 的大致思路是怎么样的？
    - 目的还是要让agent 实现最大的value，拆解为2步
        1. 学习最优的策略（策略函数）
        2. 根据最优策略作出决策
-  难点在哪里？
    - state 太多，action 太多，
- 深度学习技术是怎么帮助 实现策略学习的？
    - 构造一个神经网络来拟合策略函数，这个神经网络 叫做策略网络



## 价值网络是什么？

![image-20210713213113364](./image-20210713213113364.png)

## 策略网络是什么？



![image-20210713204844974](./image-20210713204844974.png)



输入：state，输出：多个action 下的概率，最后一层的激活函数用softmax。

- （input 和 output 的shape 和 DQ网络 一样）
- ![策略网络](./imgs%2Fapp%2Frf_learning%2FGqFFfS975r.png?alt=media&token=71ba382a-432c-4c00-8759-692d84c03f3d.png)

## A2C中策略网络，价值网络和环境的关系

![image-20210713213534951](./image-20210713213534951.png)

## 为什么不把reward直接给策略网络，而是给了价值网络呢？



为什么不直接把奖励 R 反馈给策略网络(演员)，而 要用价值网络(评委)这样一个中介呢?

原因是这样的。

策略学习的目标函数 J(θ) 是回 报 U 的期望，而不是奖励 R 的期望;注意回报 U 和奖励 R 的区别。

虽然能观测到当前的奖励R，但是它对策略网络是毫无意义的; 训练策略网络(演员)需要的是回报 U ，也就是未来所有奖励的加权和。

价值网络(评委)能够估算出回报 U 的期望，因此能帮助 训练策略网络(演员)。



## 状态价值函数是啥？

状态价值函数 ：基于某个策略，评价当下处境的好坏。state value function，对所有action求期望

- action-value function ： reward（env） + policy（agent）
- policy （agent）
- 动作的随机性来自策略
- 状态的随机性来自环境的状态转移 

## 怎么计算策略梯度？

利用策略梯度定理 简化策略梯度的计算

![image-20210713205711384](./image-20210713205711384.png)

## 策略梯度是啥？

- ![策略梯度公式](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2F-uknITMKCq.png?alt=media&token=e1a97c61-4fef-4837-8983-f74ec86f3e5f)

## 策略网络的是怎么更新参数的？

- policy-gradient算法，目标是 求一个 最优策略，使得在这个策略下，几乎所有的状态价值 都是最大的（就是对状态 求期望）
- ![Update policy network using policy gradient](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FGvhdfoS3jM.png?alt=media&token=52ed6dde-1bd6-4428-856b-e319d58800d1)
