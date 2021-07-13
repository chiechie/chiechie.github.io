---
title: 强化学习1 基本概念
author: chiechie
mathjax: true
date: 2021-04-19 20:27:02
tags:
- 强化学习
- 人工智能
categories:
- 强化学习
---



# 总结

1. 强化学习是一个序列决策过程，它试图找到一个策略，使得agent在跟环境互动的过程中，获得最大的价值。

2. agent和环境：agent就是一个机器人，是我们希望去设计出来的一个AI，环境负责跟AI进行互动，对agent的动作评价，以及生成新的状态。agent接受当前时刻的状态，返回动作agent，环境接受行动，返回下个时刻的的状态，以及奖励。

3. 状态(state)表示当前环境，在围棋例子中，表示当前的棋局。

4. 状态转移：状态转移可以是确定的也可以是随机的（随机性来源于环境）

5. 行动(action)表示当前agent的行动空间，在围棋例子中，棋盘上有361个落子位置，对应有361个action

6. 动作空间**(ActionSpace)**是指所有可能动作的集合，在围棋例子中，动作空间是 A = {1, 2, 3, · · · , 361}。

7. 策略函数(policy)表示agent根据观测到的状态做出决策. 策略函数常常被定义为一个条件概率密度函数，输入当前state，输出动作空间每个动作的概率得分。强化学习的目的，就是希望让agent学习到一个聪明的策略函数。

8. 强化学习中的两个收益：即时收益（reward)和长期收益（discounted return, aka，cumulative discounted future reward）
   1. Reward对于某个<s，a>，是在agent执行一个动作之后，环境返回的一个激励信号，是一个数值记为$R_t$，取值越大说明越赞赏当前的行动.
   2. Return对于某个<s，a>，即时收益的折扣累积就是长期收益, 记为$U_t$$$U_{t}=R_{t}+ \gamma R_{t+1}+\gamma ^{2} R_{t+2}  \cdots$$
   3. reward和return都是随机变量：reward的随机性来自当前状态和当前行动。return的随机性来自状态转移和 policy。
   
9. 如何引导agent做出好的策略？定义合适的收益函数，并且对收益求期望--即价值，然后以最大化价值作为目标来训练agent。

10. 强化学习中的价值函数主要有两个作用：一是用来判断agent的策略的好坏，一是用来判断当前局势（state）的好坏。强化学习中有3个价值函数: 行动价值函数，最优行动价值函数，状态价值函数
   1. 行动价值函数（action-value function）：表示在当前<s, a>下，采取某个策略$\pi$之后的价值，$Q_{\pi}\left(s_{t}, a_{t}\right)=E_{\pi}\left(U_{t} \mid S_{t}=s_{t}, A_{t}=a_{t}\right)$

   2. 最优动作价值函数(optimal  action-value function):表示采取最优策略$\pi^{\star}$时，给当前的<s, a>打分

      $Q^{*}\left(s_{t}, a_t\right)=\max\limits_{\pi} Q_{\pi}\left(s_{t}, a_t\right)$

   3. 状态价值函数(state value function):在后续采取某个策略$\pi$的情况下，评判当前局势的好坏（是快赢了还是快输了）。用于给当前state打分。如果对s求期望$\mathbb{E}_{S}\left[V_{\pi}(S)\right]$，就是对策略$\pi$打分。$V_{\pi}\left(s_{t}\right)=\operatorname{E_A}\left(Q_{\pi}\left(s_{t}, A\right)\right), A \sim \pi(.\mid s_t)$
     - 如果动作（action）是离散变量：$V_{\pi}\left(s_{t}\right) = \sum\limits_{a}Q_{\pi}(s_t,a) \pi\left(a \mid s_{t}\right)$
     - 如果动作（action）是连续变量： $V_{\pi}\left(s_{t}\right) = \int_{a} Q_{\pi} \left(s_{t}, a\right) \pi\left(a \mid s_{t} \right) da$
     - 总结一下，动作价值函数和最优动作价值函数是来给<state,action>打分，状态价值函数是在给当前<state，>打分，不涉评价动作。

11. 策略网络：使用一个神经网络来近似策略函数

   


## 强化学习的两个方向: 价值学习和策略学习



强化学习的方法通常分为两类，基于模型的方法（model-based）和 无模型的方法（model-free）。

无模型的方法又分为价值学习和策略学习。



价值学习:

1. 价值学习 (Value-Based Learning) ，是指 以最大化价值函数（3个价值函数都可）为目标去训练agent。

2. 价值学习的大致思路是这样的，每次agent观测到一个$s_t$,就把它输入价值函数，让价值函数来对所有动作做评价（分数越高越好）

3. 那么问题来了，如何去学习价值函数呢？可以使用一个神经网络来近似价值函数。
4. 最有名的价值学习方法是DQN, 还有Q-learning



策略学习

1. 策略学习指的是学习策略函数$\pi$, 然后agent可利用策略函数计算不同state下行动的得分，然后随机选一个执行。
2. 那么问题来了，如何去学习策略函数呢？可以使用一个神经网络来近似策略函数，然后使用策略梯度来更新网络参数。



#  附录

## 价值学习

价值学习： value-based，目的是学习最优行动价值函数。

- Deep Q network: 近似最优行动价值函数
- 时间差分（TD）算法:TD（Temporal Difference）算法：
    - SARSA算法：基于表格的方法和基于神经网络的方法
    - Q-learning算法
    - Multi-step TD target

- 策略学习：policy-based，目的是让agent直接学会最优策略。 actor critic
    - 使用策略网络来近似策略函数， 使用策略梯度更新网络参数
        -  ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2Frx6kfw7dc6.png?alt=media&token=22e1d520-3194-42b5-b624-e52034b62b4d)
    -  ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FNC2bv9ZwlF.png?alt=media&token=8fb33ced-8383-42fe-8fee-4742d9abadc4)
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FV4DSavxJZ8.png?alt=media&token=408e1eb5-24f9-4fd9-bbf5-a7d20a53f7fb)
    - 计算策略梯度和行动价值函数
        - 方法1-跟环境互动获取长期的收益![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2F8F3Gq0YTEB.png?alt=media&token=d6cd369f-ea76-41b0-84e1-66953c0d4e56)
        - 方法2-构造价值网络来计算action-value
- 价值学习和策略学习结合： actor critic ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FQX2HjRc5gn.png?alt=media&token=18a877c8-f337-4c15-a9f5-7b3f410c8475)

## 怎么理解两个价值函数？

![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FDl41z9c-9y.png?alt=media&token=d5e65193-4372-4d43-b8c4-85237c20b61d)

## 控制agent的两种方法-基于策略和基于最优动作价值函数

![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FtnK44wspcQ.png?alt=media&token=259a4682-aa14-4b7d-8f55-e88d29cdb319)

![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FJf9FJZ0nSH.png?alt=media&token=8fb09202-0693-4658-9c45-d2bec3f8642c)

## 参考资料 

1. [wangshusen-slide](https://github.com/wangshusen/DRL/blob/master/Slides/1_Basics_1.pdf)
