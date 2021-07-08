---
title: 强化学习10 使用gym规划出租车行车路线的demo
author: chiechie
mathjax: true
date: 2021-04-28 20:33:14
tags:
- 强化学习
- 人工智能
categories:
- 强化学习
---

> 看完了强化学习的理论，现在来动手试一试

## 场景介绍

以出租车司机路线规划为背景，下面演示一下如何由强化学习解决这个问题

为了将问题进一步简化，基于已有的强化学习框架gym，我就可以更关注在策略实现了

首先强化学习将问题建模为两个部分：环境（environment）和 决策主体（agent）。
环境基于state，action给出反馈（reward），以及下一个时刻的reward。
agent基于state，给出最佳的action。

对于「出租车路线规划问题」，state就是给定区域：出租车坐标，行人坐标，行人目的地，障碍物（墙）的坐标，出租车当前是否已经载人

action就4个方向（东南西北），2个动作（载人，下客）

## 开始编程实现啦

### step1: 加载「出租车路线规划」这个环境

因为gym已经带有这个环境，直接可用

```python
import gym

env = gym.make("Taxi-v3")
env.reset()
# visulize current state
env.render()
```
![environment](img.png)

上图环境渲染的图中

- 黄色方块代表出租车当前的位置
- “|”代表一堵墙，不能穿透
- 蓝色的字母表示载人地方
- 紫色的字母表示下客的地方
- 出租车接到人之后会编程绿色

注意，这些颜色，形状只是方便人理解，算法内部只关心有5个状态

### step2： 看一下action的6个状态

看一下action的6个状态，
down (0), up (1), right (2), left (3), pick-up (4), and drop-off (5).
```python
print(env.action_space)
for i in range(10):
    print(env.action_space.sample())
```
![img_1.png](img_1.png)

### step3: 跟env交互一次

跟env交互一次，拿到反馈和最新的状态


```python
state, reward, done, info = env.step(action=1)
```
![img_3.png](img_3.png)
注意：每个Gym环境的step方法都返回state, reward, done, info这四个变量，输入某个action

### step4： 使用某个简单策略（随机策略）

 使用某个简单策略（随机策略），让出租车跑完一轮

```python
state = env.reset()
env.render()
n = 0
r = 0
while True:
    state, reward, done, info = env.step(action=env.action_space.sample())
    r += reward
    n = n + 1
    if done:
        break
print("totally make %d decisions" % (n+1), "total reward", r)
env.render()
```

![img_4.png](img_4.png)

### step5: 升级策略-Q-table, 跑多轮

Q-table使用bellman方程迭代更新策略，并且存放到一张表中（这个表就是Q-table），即每个状态，每个action对应的Q-value

$$Q_{t+1}\left(s_{t}, a_{t}\right)=\underbrace{Q_{t}\left(s_{t}, a_{t}\right)}_{\text {old value }}+\underbrace{\alpha_{t}\left(s_{t}, a_{t}\right)}_{\text {learning rate }} \times[\overbrace{\underbrace{R_{t+1}}_{\text {reward }}+\underbrace{\gamma}_{\text {discount factor }} \underbrace{\max _{a} Q_{t}\left(s_{t+1}, a_{t}\right)}_{\text {estimate of optimal future value }}}^{\text {learned value }}-\underbrace{Q_{t}\left(s_{t}, a_{t}\right)}_{\text {old value }}]$$

```python
n_state = env.observation_space.n
n_action = env.action_space.n
Q_table = np.zeros((n_state, n_action))
alpha = 0.615
gamma = 0.9

for episode in range(1000):
    state = env.reset()
    #env.render()
    n = 0
    r = 0
    done = False
    while done is False:
        best_action = np.argmax(Q_table[state, :])
        state2, reward, done, info = env.step(action=best_action)
        Q_table[state, best_action] = Q_table[state, best_action] + alpha *(reward + gamma * np.max(Q_table[state2]) - Q_table[state, best_action])
        r += reward
        n = n + 1
        # 别漏了
        state = state2 
    if episode % 50 == 0:
        print('Episode {} Total Reward: {}'.format(episode,r))
```

![img_6.png](img_6.png)


## 遗留问题

1. 策略函数要设置约束，比如不能撞墙


## 参考

1. [强化学习环境框架-gym](https://www.oreilly.com/radar/introduction-to-reinforcement-learning-and-openai-gym/))