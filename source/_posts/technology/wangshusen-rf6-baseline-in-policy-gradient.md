---
title: wangshusen-rf6-策略梯度中的baseline
author: chiechie
mathjax: true
date: 2021-04-18 20:41:33
tags:
- 强化学习
- 人工智能
categories:
- 技术
---

## 策略梯度中的Baseline 1/4-数学推导
- https://www.youtube.com/watch?v=yNEqbptitZs
- Policy gradient (策略梯度) 方法中常用 baseline 来降低方差，加速收敛。Baseline 可以是任何不依赖于动作的函数。这节课我们做数学推导，得出带有 baseline 的策略梯度，并对它做蒙特卡洛近似。在后面的两节课中，我们把它用在 REINFORCE 和 A2C 方法中。
- 策略梯度中的数学推导
- 将baseline 应用到reinforce算法中
- 将baseline 应用到actor2critic方法中：
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FVfCPRKT8Wg.png?alt=media&token=aa73c63c-f5f4-4b48-b626-a5c5262b71ca)
    - 直观理解：
        - 放到两军交战的场景，对于某一方来说
            - 策略网络就像一个军师，对于每一种战况（state）给出行动建议（action）；
            - 价值网络就像一个先知，预测当前战况（state）的好坏（state value）。
            - 战斗是一个持续的过程，每次交锋（at）一次，根据即时的收益（rt）, 军队首领（agent）要反省得失，更新军师的策略：
                -  根据先知对交锋前后的战况预测差异 ， 来评价刚刚行为（at）好坏（好坏用 td error来衡量，越大说明at越垃圾）， 并且用来指引接下来策略（梯度更新公式中出现的td error一项）
                - 此外，策略调整的方向 还有一个因子，就是 策略梯度，也就是 在st时，选择at的概率下降 的方向。是想让军师探索其他方向？？
            - 同时，也要更新先知的认知
                - 梯度表示想要td error减少

## 策略梯度中的Baseline 2/4-REINFORCE with Baseline

- Policy gradient (策略梯度) 方法中常用 baseline 来降低方差，加速收敛。这节课我们学习 REINFORCE with Baseline，它是一种策略梯度方法。它用价值网络近似 baseline 函数。它用带 baseline 的策略梯度学习策略网络，用价值网络去拟合观测到的回报。
- ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FMRYKuGAY2M.png?alt=media&token=120372a3-7c92-4b48-be89-9bc99679e69c)
- ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2F2KjX--n5BX.png?alt=media&token=5edb9a0b-aaec-4d99-addf-ff2898eccd99)
- 
- ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FWR1r0uRGvm.png?alt=media&token=3bb38d00-ec27-4f8d-acf9-8aa0208cb791)
- 接下来用reinforce训练策略网络，用回归算法训练价值网络
- 更新策略网络：
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2Fi95LEON8Wl.png?alt=media&token=626bdff1-fccd-4c9f-b1cc-c288c2ee0977)
- 更新价值网络
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2F55QYQnU8wS.png?alt=media&token=b116aa54-22c2-49ae-9fe5-a8b124f78d56)
- 整体：
- ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2Foc2FFolpEM.png?alt=media&token=9f2955f5-98a1-4752-b475-094e41be8e6f)

## 策略梯度中的Baseline 3/4-A2C 方法

https://www.youtube.com/watch?v=mtT4TSGSon8


## 策略梯度中的Baseline 4/4-REINFORCE与A2C的异同

- 先看A2C： 单步td算法和 多步td 算法（多步用的多一些）
    - one -step ：
        - step1 观察1个transition数据
            - $\left(s_{t}, a_{t}, r_{t}, s_{t+1}\right)$
        - step2 计算TD Target
            - $y_{t}=r_{t}+\gamma \cdot v\left(s_{t+1} ; \mathbf{w}\right)$
        - step 3 计算TD eror
            - $\delta_{t}=v\left(s_{t} ; \mathbf{w}\right)-y_{t}$
        - step 4 更新策略网络
            - $\boldsymbol{\theta} \leftarrow \boldsymbol{\theta}-\beta \cdot \delta_{t} \cdot \frac{\partial \ln \pi\left(a_{t} \mid s_{t} ; \boldsymbol{\theta}\right)}{\partial \boldsymbol{\theta}}$
        - step5 更新价值网络
            - $\mathbf{w} \leftarrow \mathbf{w}-\alpha \cdot \delta_{t} \cdot \frac{\partial v\left(s_{t} ; \mathbf{w}\right)}{\partial \mathbf{w}}$
    - multi-step 
        - step1: 观察m个transitions数据$\left\{\left(S_{t+i}, a_{t+i}, r_{t+i}, S_{t+i+1}\right)\right\}_{i=0}^{m-1}$
        - step2: 计算TD target：
            - $y_{t}=\sum\limits_{i=0}^{m-1} \gamma^{i} \cdot r_{t+i}+\gamma^{m} \cdot v\left(s_{t+m} ; \mathbf{w}\right)$
        - step3: 计算 TD error：
            - $\delta_t = v(s_t;w) - y_t$
        - step4: 更新策略网络（actor）
            - $\boldsymbol{\theta} \leftarrow \boldsymbol{\theta}-\beta \cdot \delta_{t} \cdot \frac{\partial \ln \pi\left(a_{t} \mid s_{t} ; \boldsymbol{\theta}\right)}{\partial \boldsymbol{\theta}}$
        - step5: 更新价值网络（critic）
            - $\mathbf{w} \leftarrow \mathbf{w}-\alpha \cdot \delta_{t} \cdot \frac{\partial v\left(s_{t} ; \mathbf{w}\right)}{\partial \mathbf{w}}$
- 再看看reinforce with baseline
    - step1 获取一整局游戏的transition 数据
    - step2 使用观测到的奖励来计算回报
        - $u_{t}=\sum_{i=t}^{n} \gamma^{i-t} \cdot r_{i}$
    - step3 计算error：价值网络和回报的差
        - $\delta_{t}=v\left(s_{t} ; \mathbf{w}\right)-u_{t}$
    - step4 更新策略网络
        - $\boldsymbol{\theta} \leftarrow \boldsymbol{\theta}-\beta \cdot \delta_{t} \cdot \frac{\partial \ln \pi\left(a_{t} \mid s_{t} ; \boldsymbol{\theta}\right)}{\partial \boldsymbol{\theta}}$
    - step5 更新价值网络
        - $\mathbf{w} \leftarrow \mathbf{w}-\alpha \cdot \delta_{t} \cdot \frac{\partial v\left(s_{t} ; \mathbf{w}\right)}{\partial \mathbf{w}}$
- 两者的关系：A2C 和 Reinforce
    - Reinforce 是A2C的一种特例，用到了更多的观测数据，没有使用bootsrap（a2c的第二项）
        - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2F4cy4PbmERs.png?alt=media&token=07f13dfc-b03b-43f5-a99e-82690b8b6a79)
    - one -step A2C 是 multi-step A2C 的一个特例，使用了最少的观测数据。multi-step的效果更好
        - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FNV4Q90x0xU.png?alt=media&token=eed6b59f-01e5-4803-9475-27a7f594bab9)

## 参考
1. https://www.youtube.com/watch?v=hN9WMIMMeAI&t=64s
