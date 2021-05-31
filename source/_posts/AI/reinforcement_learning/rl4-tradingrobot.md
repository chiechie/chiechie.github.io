---
title: 强化学习4 使用gym构建股票交易机器人
author: chiechie
mathjax: true
date: 2021-05-31 15:01:49
tags:
- 强化学习
- 人工智能
- 量化交易
categories:
- AI
---

## 思路

现在进行强化学习第2个实践项目--构建交易机器人，

老路子，先构建environment 再构建agent

environment需要定义state，action，以及下个时刻的reward，state
agent需要定义每个state，action的value或者直接给每个state下最优的action

作为练手项目，就从0到构建一个吧

为了构建environment，先简化问题:

- state: 过去3天close/open/high/low/volume5个指标的数据, shape = <3, 5>
- action: 买/卖/持仓 + 数量(基于目前仓位的百分比), shape = <2, > ,
- step方法:
    - state2: 第二天的close/open/high/low/volume5个指标的数据, shape = <3, 5>
    - reward：第二天的net value - 操作之前的net value

为了构建agent:

- 随机策略
- Q-table好像不行，因为state不连续了。
- naive策略：均线策略，or韭菜策略，短期，低买高卖。


## 开始实施




## 参考
5. [构建交易环境和交易策略-github](https://github.com/wangshub/RL-Stock)
4. [构建交易环境-medium](https://towardsdatascience.com/creating-a-custom-openai-gym-environment-for-stock-trading-be532be3910e)
1. [quantML-github-chiechie](https://github.com/chiechie/quantML/blob/master/gym_rl.py)
2. [强化学习算法框架--stable-baselines-](https://github.com/hill-a/stable-baselines)
3. [强化学习环境框架-gym](https://www.oreilly.com/radar/introduction-to-reinforcement-learning-and-openai-gym/))