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

step 1: 自定义一个environment，需要继承gym的标准环境类--gym.Env, 并实现类的关键方法，有step和reset

```python
class CustomEnv(gym.Env):
    metadata = {'render.modes': ['human']}

    def __init__(self, arg1, arg2, ...):
        super(CustomEnv, self).__init__()
        self.action_space = None
        self.observation_space = None
                                       

    def step(self, action):                                               
        return observation, reward, done, info
    
    def reset(self):
        return observation  # reward, done, info can't be included
    
    def render(self, mode='human'):
                                       
        return
    
    def close (self):
        return 


```
step1-1: 在构造函数中，定义state空间，和 action空间
state是连续的，action因为涉及到具体买卖多少份额，所以也有部分是连续的，
可以用gyn.space.Box来定义，需要指定上下界。 此外，为了简化问题，直接把一个df当作成员给这个子类

```python
class StockEnv(gym.Env):
    def __init__(self, df):
        super(StockEnv, self).__init__()
        self.action_space = spaces.Box(low=np.array([0,0]), high=np.array([3, 1]), dtype=np.float16)
        self.observation_space = spaces.Box(low=0, high=1, shape=(3, 5), dtype=np.float16)
        self.df = df

```
step1-2: 定义step方法，step是标准环境类的标准方法，输入输出的格式是固定的，输入一个state，输出下一个state, reward, done, info。

这里涉及到交易的常识，补充下几个概念：

- net value：账户净值，也就是说账户当前现金+股票折现价值
- cash：这里叫现金是为了方便理解。专业的叫法是balance。
- action amount：一个百分比，基于当前账户余额（balance）可以买卖的比例。 

```python
def step(self, action):
    current_price = self.df.loc[self.current_step, "cLose"]
    action_type, action_percent = action
    
    # [0, 1)表示买     
    action_amount = self.cash * action_percent / current_price
    if action_type < 1:
        self.share_hold += action_amount
        self.cash -= action_amount * current_price
        reward = (self.df.loc[self.current_step + 1, "cLose"] - current_price) * (action_amount + self.share_hold)
    # [1,2) 表示卖
    elif action_type < 2:
        self.share_hold -= action_amount * current_price
        self.cash = action_amount 
        reward = (self.df.loc[self.current_step + 1, "cLose"] - current_price) * ( - action_amount + self.share_hold)
    # [2, 3]表示不动
    self.current_step += 1
    observation = self.df
    next_price = self.df.loc[self.current_step, "cLose"]
                                   
    self.net_worth = self.cash + self.share_hold * self.df.loc[self.current_step + 1, "cLose"]
                                          
    return observation, reward, done, info

```

## 参考

1. [quantML-github-chiechie](https://github.com/chiechie/quantML/blob/master/gym_rl.py)
2. [强化学习算法框架--stable-baselines](https://github.com/hill-a/stable-baselines)
3. [强化学习环境框架-gym](https://www.oreilly.com/radar/introduction-to-reinforcement-learning-and-openai-gym/)
4. [构建交易环境-medium](https://towardsdatascience.com/creating-a-custom-openai-gym-environment-for-stock-trading-be532be3910e)
5. [构建交易环境和交易策略-github](https://github.com/wangshub/RL-Stock)
