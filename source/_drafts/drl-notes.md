---
title: 强化学习-笔记
author: chiechie
mathjax: true
date: 2021-03-08 13:18:33
tags:
- AI技术
- 强化学习
categories: 技术类
---

## 基本概念

- 术语：
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FiWn6HNTIWS.png?alt=media&token=5dba0429-7a1c-4c50-b6ad-817dac166b86)
- Returns的随机性来源：action和状态转移
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FrPEtw3BcPJ.png?alt=media&token=33034dcc-3fc1-456e-8c76-171e67ad7cfd)
- 怎么理解两个价值函数？
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FDl41z9c-9y.png?alt=media&token=d5e65193-4372-4d43-b8c4-85237c20b61d)
- 控制agent的两种方法-基于策略和基于最优动作价值函数
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FtnK44wspcQ.png?alt=media&token=259a4682-aa14-4b7d-8f55-e88d29cdb319)
- ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FJf9FJZ0nSH.png?alt=media&token=8fb09202-0693-4658-9c45-d2bec3f8642c)
- 参考资料
    - https://github.com/wangshusen/DRL/blob/master/Slides/1_Basics_1.pdf


## 基于策略的强化学习

- 问题：
    - 基于策略学习 的大致思路是怎么样的？
        - 目的还是要让agent 实现最大的value，拆解为2步
            - 1. 学习最优的策略（策略函数）
            - 2. 根据最优策略作出决策
    -  难点在哪里？
        - state 太多，action 太多，
    - 深度学习技术是怎么帮助 实现策略学习的？
        - 构造一个神经网络来拟合策略函数，这个神经网络 叫做策略网络
- 策略网络：输入：state，输出：多个action 下的概率，最后一层的激活函数用softmax。
    - （input 和 output 的shape 和 DQ网络 一样）
    - ![策略网络](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FGqFFfS975r.png?alt=media&token=71ba382a-432c-4c00-8759-692d84c03f3d)
    - ![价值网络](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FfsQADMgSRa.png?alt=media&token=18bf844e-aa59-4016-85d6-8cdfc801a9ce)
- 状态价值函数 ：基于某个策略，评价当下处境的好坏。state value function，对所有action求期望
    - action-value function ： reward（env） + policy（agent）
    - policy （agent）
    - 动作的随机性来自策略
    - 状态的随机性来自环境的状态转移 
- 策略梯度是啥：
    - ![策略梯度公式](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2F-uknITMKCq.png?alt=media&token=e1a97c61-4fef-4837-8983-f74ec86f3e5f)
- 策略网络的是怎么更新参数的？
    - policy-gradient算法，目标是 求一个 最优策略，使得在这个策略下，几乎所有的状态价值 都是最大的（就是对状态 求期望）
    - 
    - ![Update policy network using policy gradient](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FGvhdfoS3jM.png?alt=media&token=52ed6dde-1bd6-4428-856b-e319d58800d1)



## 参考
1. [强化学习中文版教材-wangshusen-github](https://github.com/wangshusen/DRL/blob/master/Notes_CN/DRL.pdf)
2. [强化学习怎么入门？wangshusen的回答-知乎](https://www.zhihu.com/question/277325426/answer/1753868459)
3. [chiechie的手写笔记](notability)