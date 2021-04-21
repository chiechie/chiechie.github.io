---
title: wangshusen-rf5-价值学习高级技巧1/3-经验回放
author: chiechie
mathjax: true
date: 2021-04-18 20:40:17
tags:
- 强化学习
- 人工智能
categories:
- 技术
---

- 经验回放(experience replay）： 就是把n条transition记录存下来(bufffer)，更新价值的时候，从里面抽样。好处是可以重复利用经验。
- 技术发展路线
    - 93年提出「经验回放」的想法
    - 15年的DQN使用到了「经验回放」
    - 之后各种模型训练都会用到该技巧
    - 「经验回放」演化成了强化学习里面的标准方法
- 「经验回放」的变体
    - 最优「经验回放」： 给予极端场景更高的权重
        - 增加transition采样的权重
        - 减少使用transition更新价值网络时的 learning rate
