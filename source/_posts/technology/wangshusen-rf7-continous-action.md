---
title: wangshusen-rf7-连续控制
author: chiechie
mathjax: true
date: 2021-04-18 20:46:57
tags:
- 强化学习
- 人工智能
categories:
- 技术
---


## 离散控制与随机控制介绍

- 这节课讨论离散控制与连续控制的区别、连续控制的难点、以及连续动作空间的离散化。
- 之前讲的强化学习方法 都是来 处理离散控制变量的场景的：策略梯度 和 DQN
- 那么对连续控制变量这种场景如何处理呢？
    - 1. 将连续空间网格化，套上离散控制变量的方法。但是输出向量的长度随着控制变量的个数呈指数增加。
    - 2. 使用更适合连续控制变量的方法：
        - 确定的策略网络
        - 随机的策略网络


## 确定策略梯度DPG
- DPG是一种actor cirtic方法
- DPG的策略网络输出1个实数（机械手臂只有1个关节）或者1个向量（机械手臂又多个关节），但是输出是确定的没有随机性的
- 流程：
    - 结构：
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FFFnC_ZZDfS.png?alt=media&token=b76bce5a-44cb-4967-8831-9ad7e3bac627)
    - 更新价值网络
    - 更新策略网络
        - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2F1lO0oHsg0f.png?alt=media&token=4624994e-3a30-4a5d-9ea3-7ecc419516e1)
- 缺点，bootstrap带来的估计偏差（一开始TD target估高了，就会持续高估）
    - bootstrap的解释：使用自己的更新至来更新自己
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FckUhWapL92.png?alt=media&token=56166270-8cad-42fc-a94e-f88f329c6c41)
- 改进方法-target networks：基本思路，使用别的网络来计算TD target
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2F6lBqZiXi9f.png?alt=media&token=17ff418c-0a8d-437f-8b82-7848a8cfa243)
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FkSbwop1R3_.png?alt=media&token=9dd29838-a62a-4a89-8717-7e1972f3da74)
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FKrdc65AdRE.png?alt=media&token=3d72f27f-a8be-468c-8910-df0ad025f5da)
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FnLzNJ8bG14.png?alt=media&token=fe8f8570-8304-4c5e-bd16-0649684ecaf7)
- 还有一些其他的DPG的改进方法：
    - target network
    - 经验回放（experience replace）
    - multi-step TD target
- 随机策略网络和确定策略网络对比
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FRDz29hZbP4.png?alt=media&token=52f95ad5-5f13-4974-87f8-c52d8c734836)


## 用随机策略做连续控制

- 基本概念
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2Fza5rJJXnoe.png?alt=media&token=48526a35-cc19-486c-b91b-3a0ef9df1edc)
    - 策略梯度：--谁对谁的梯度？（状态价值函数 关于策略网络参数的梯度）
        - $\frac{\partial V_{\pi}(s)}{\partial \boldsymbol{\theta}}=\mathbb{E}_{A \sim \pi}\left[\frac{\partial \ln \pi(A \mid s ; \boldsymbol{\theta})}{\partial \boldsymbol{\theta}} \cdot Q_{\pi}(s, A)\right]$
    - 随机策略梯度
        - $\mathbf{g}(a)=\frac{\partial \ln \pi(a \mid s ; \boldsymbol{\theta})}{\partial \boldsymbol{\theta}} \cdot Q_{\pi}(s, a), \text { where } a \sim \pi(\cdot \mid s ; \boldsymbol{\theta})$
- 连续控制问题 怎么设计方案？
    - chiechie的思索：第一反应是 使用回归模型，即策略网络是将state 映射到一个确定的action（比如机械臂转动多少度），但是从贝叶斯角度来理解现实问题的话，刚刚的处理方式是明显不合理的， 因为永远 不存在唯一的正确答案。（贝叶斯认为正确答案 是一个随机变量，只不过在不同区间的概率分布不一样）。
    - 接下来看看wang给的方案：
        - 将连续的决策空间离散化为网格空间，但是会带来维树灾难
        - 使用确定性的梯度算法：上一节讲的内容，对于给定的状态，输出的action是唯一确定的。（这不就是我上面的想法吗？似乎不是，这个随机策略梯度更像是一种数值算法，类似sgd）
        - 使用随机策略网络，也就是本节要讲的内容
    - 【核心】构建一个输出pi的概率分布的策略网络：
        - 如何训练？ --使用随机策略梯度
            - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2F-HeYISazuj.png?alt=media&token=0d3ebe9c-6f04-4867-b2cb-553503e9a523)
        - 策略梯度的第一个部分使用 辅助神经网络，第二部分使用 reinforce 或者 actor2critic来计算
            - step1- 辅助神经网络：就是算实际action的似然概率，以此作为反馈信号，往后传播策略网络参数的梯度。
                - 往前传播
                    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FF3rR0gy8Su.png?alt=media&token=32c8deac-e6ed-4c36-8490-682457bd96cf)
                - 往后传播
                    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FqKuN1X6D_b.png?alt=media&token=aa379ddd-eb64-4674-a1c9-136f77996250)
            - step2-计算Q value：然是在计算策略梯度的时候，又遇到了一个问题，就是如何计算Qvalue？又两种方法：
                - reinforce： 玩完一局，然后那观测到的ut去近似
                - actior-critic：使用价值网络来近似Q，
                    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2F1OdWlyBP_B.png?alt=media&token=4e37f7bb-e85d-4e0b-afb3-bbf99f3bb628)
                - 还有更高级的方法--策略梯度with baseline: 
                    - reinforce==> [[策略梯度中的Baseline 2/4-REINFORCE with Baseline]]
                    - actor-critic==> [[策略梯度中的Baseline 3/4-A2C 方法]]

    
## 参考

1. https://www.youtube.com/watch?v=rRIjgdxSvg8
2. https://www.youtube.com/watch?v=cmWejKRWLA8
3. https://www.youtube.com/watch?v=McqFyl_W5Wc
