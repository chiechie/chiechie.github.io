---
title: wangshusen-rf1-强化学习基本概念
author: chiechie
mathjax: true
date: 2021-04-18 20:27:02
tags:
- 强化学习
- 人工智能
categories:
- 技术
---

## 术语

![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FiWn6HNTIWS.png?alt=media&token=5dba0429-7a1c-4c50-b6ad-817dac166b86)

## Returns的随机性来源：action和状态转移

![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FrPEtw3BcPJ.png?alt=media&token=33034dcc-3fc1-456e-8c76-171e67ad7cfd)


## 强化学习的三个方向

- 价值学习： value-based，目的是学习最优行动价值函数。
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
 
## 基本概念

- 状态
- 行动
- 状态转移：状态转移可以是确定的也可以是随机的（随机性来源于环境）![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2F0gnPu0zeUj.png?alt=media&token=d6bcd51c-418f-4c1c-800b-e82842f20c27)
- 2个收益: 长期和即时收益。这两个收益都是随机变量，怎么利用收益函数来 引导agent做出好的策略？对收益求期望，也就是价值函数
    - 强化学习中的两个收益：即时收益（reward)：对于某个s，a，env立刻返回的激励信号。长期收益（discounted return, aka，cumulative discounted future reward）：对于某个s，a，长期来看收益。
    - 这两个收益都是随机变量, $R_t$的折扣累积就是长期收益
    - $U_{t}=R_{t}+ \gamma R_{t+1}+\gamma ^{2} R_{t+2}  \cdots$
    - $U_t$的随机性来自2个部分: 
        - 环境：状态转移概率, $p\left(s^{\prime} \mid s, a\right)$
        - agent: policy函数
    - $R_t$的随机性来自2个部分：$s_t$和$a_t$
- 策略网络：使用一个神经网络来近似策略函数
- 3个价值函数: 行动价值函数，最优行动价值函数，状态价值函数
    - 强化学习中的价值函数主要有两个作用：
    - 一是用来判断agent的策略的好坏，一是用来判断当前局势（state）的好坏。
    - 具体来说主要有3个：
    - 动作价值函数（action-value function）：表示在当前s和a下，采取某个策略$\pi$之后的价值，$Q_{\pi}\left(s_{t}, a_{t}\right)=E_{\pi}\left(U_{t} \mid S_{t}=s_{t}, A_{t}=a_{t}\right)$
        - 可以给当前的a打分
    - 最优动作价值函数（optimal  action-value function），当前s和a下，采取最优策略$\pi^{\star}$的分数，$Q^{*}\left(s_{t}, a_t\right)=\max\limits_{\pi} Q_{\pi}\left(s_{t}, a_t\right)$
    - 状态价值函数：state value function，在后续采取某个策略$\pi$的情况下，评判当前局势的好坏（是快赢了还是快输了）。用于给当前state打分。如果对s求期望$\mathbb{E}_{S}\left[V_{\pi}(S)\right]$，就是对策略$\pi$打分。
        - $V_{\pi}\left(s_{t}\right)=\operatorname{E_A}\left(Q_{\pi}\left(s_{t}, A\right)\right), A \sim \pi(.\mid s_t)$
        - 如果动作（action）是离散变量：$V_{\pi}\left(s_{t}\right) = \sum\limits_{a}Q_{\pi}(s_t,a) \pi\left(a \mid s_{t}\right)$
        - 如果动作（action）是连续变量： $V_{\pi}\left(s_{t}\right) = \int_{a} Q_{\pi} \left(s_{t}, a\right) \pi\left(a \mid s_{t} \right) d a$
    - 前面两个（action-value function 和 optimal action-value function）都是评判state + action +policy。
    - 第三个（state value fuction） 单纯地评判当前局势（ state）+ policy，。

## 怎么理解两个价值函数？

![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FDl41z9c-9y.png?alt=media&token=d5e65193-4372-4d43-b8c4-85237c20b61d)

## 控制agent的两种方法-基于策略和基于最优动作价值函数

![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FtnK44wspcQ.png?alt=media&token=259a4682-aa14-4b7d-8f55-e88d29cdb319)

![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FJf9FJZ0nSH.png?alt=media&token=8fb09202-0693-4658-9c45-d2bec3f8642c)

## 参考资料 
1. [wangshusen-slide](https://github.com/wangshusen/DRL/blob/master/Slides/1_Basics_1.pdf)
