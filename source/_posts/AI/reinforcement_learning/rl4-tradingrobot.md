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

> 现在进行强化学习第2个实践项目--构建交易机器人，

## 思路

老路子，先构建environment 再构建agent：

- environment需要定义state，action类型，以及step和render方法。
- agent需要定义策略函数，即每个state下最优action

下面从0到构建一个, 

为了构建environment，先简化问题:

- state: 过去3天close/open/high/low/volume5个指标的数据, shape = <3, 5>
- action: 买/卖/持仓 + 数量(基于目前仓位的百分比), shape = <2, > ,
- step方法返回
    - state2: 第二天的close/open/high/low/volume5个指标的数据, shape = <3, 5>
    - reward：变动仓位 * 变化的股价

为了构建agent:

- 随机策略
- Q-table不行，因为state不连续了
- naive策略：趋势策略，短期有上升趋势就买入，否则卖出。


## 实施

### step 1 自定义一个environment

自定义一个environment，需要继承gym的标准环境类--gym.Env, 并实现类的关键方法，有step和reset

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

#### step1-1: 定义环境类的构造函数

在构造函数中，定义state空间，和 action空间
state是连续的，action因为涉及到具体买卖多少份额，所以也有部分是连续的，
可以用gyn.space.Box来定义，需要指定上下界。 此外，为了简化问题，直接把一个df当作成员给这个子类

```python
class StockEnv(gym.Env):
    def __init__(self, df):
        super(StockEnv, self).__init__()
        self.action_space = spaces.Box(low=-1, high=1,shape=(1, ), dtype=np.float16)
        self.observation_space = spaces.Box(low=0, high=1, shape=(3, 5), dtype=np.float16)
        self.df = df
```


#### step1-2: 定义环境类的reset方法

定义环境类的reset方法，reset用来重置一局游戏。需要返回游戏的初始状态，收益为0，
计算收益的几个变量，账户净值，持股数量，余额，以及游戏开始的时间戳（这个可以随机设置的）



```python
def reset(self):
    self.net_worth = 10000
    self.share_hold = 0
    self.current_step = 2
    self.cash = 10000
    observation = self.df.loc[self.current_step - WINDOW_SIZE + 1: self.current_step,  ["open", "close", "high", "low", "volume"]]

    return observation  # reward, done, info can't be included

```

#### step1-3: 定义环境类的step方法

定义step方法，step是标准环境类的标准方法，输入输出的格式是固定的，输入一个state，输出下一个state, reward, done, info。

这里涉及到交易的常识，补充下几个概念：

- net value：账户净值，也就是说账户当前现金+股票折现价值
- cash：这里叫现金是为了方便理解。专业的叫法是balance。
- action amount：一个百分比，基于当前账户余额（balance）可以买卖的比例。 
- done：什么时候一轮游戏结束？时间到了或者爆仓了，即net value < 0了

```python
    def step(self, action):
        current_price = self.df.loc[self.current_step, "close"]
        # action_type, action_percent = action
        # action = action[0]
        action_amount = abs(action)
        buy = action > 0
        sell = action < 0
        # 0表示买     
        action_amount = self.cash * action_amount / current_price
        reward = 0
        if sell and self.share_hold < action_amount:
            # 不允许做空
            pass
        elif buy:
            self.share_hold += action_amount
            self.cash -= action_amount * current_price
            reward = (self.df.loc[self.current_step + 1, "close"] - current_price) * action_amount
        else:
            self.share_hold -= action_amount
            self.cash += action_amount *  current_price
            reward = - (self.df.loc[self.current_step + 1, "close"] - current_price) * action_amount

        self.current_step += 1
        observation = self._next_observation()
        next_price = self.df.loc[self.current_step, "close"]
                                       
        self.net_worth = self.cash + self.share_hold * next_price
        done = self.net_worth <= 0.01
                                             
                                               
        return observation, reward, done, {}
```

### step 2- 定义策略policy

#### step 2-1 定义一个静态策略

这里先用一个简单的趋势策略作为baseline，如下

```
def pi(obs):
    close_list = obs["close"].values
    ma = close_list[:-1].mean()
    price =  close_list[-1]

    if price / ma > 1.02:
        action = 0.2
    elif price / ma < 0.98:
        action = -0.2
    else:
        action = 0
    return action
    
```

#### step 2-2 使用PPO算法构建策略

PPO（Proximal Policy Optimization）算法结合了A2C和TRPO的idea，主要思路是，
每次更新策略时，新的策略不能离旧的策略太远，具体的做法是对梯度做截断（clipping）。


```python
env_PPO = DummyVecEnv([lambda: StockEnv(df)])
model = PPO2(MlpPolicy, env_PPO, verbose=0)
model.learn(total_timesteps=100)
```
#### step2-3 对比

```python
    df = pd.read_csv('./stockdata/train/sh.600004.白云机场.csv')
    df = df.sort_values('date')
    df["volume"] = df["volume"].map(np.log)

    env_PPO = DummyVecEnv([lambda: StockEnv(df)])
    env_basis = StockEnv(df)
    env_PPO1 = StockEnv(df)

    model = PPO2(MlpPolicy, env_PPO, verbose=0)
    model.learn(total_timesteps=100)

    obs_PPO1 = env_PPO1.reset()
    obs_basis = env_basis.reset()
    env_PPO1.current_step = env_basis.current_step
    obs_PPO1 = obs_basis

    for i in range(200):
        action_PPO1, _ = model.predict(obs_PPO1)
        action_basis = pi(obs_basis)
        # print(env_PPO1.current_step, env_basis.current_step)
        obs_PPO, rewards_PPO, done_PPO, info_PPO = env_PPO1.step(action_PPO1)
        obs_basis, rewards_basis, done_basis, info_basis = env_basis.step(action_basis)
        if i % 20 ==0:
            print("****"*5, "PPO", "***"*5)
            env_PPO1.render()
            print("****"*5,"Basis","***"*5)
            env_basis.render()
            
```

还凑活吧，能用

![img.png](img.png)

## 总结

1. 牛刀初试，强化学习实际效果不算离谱，凑活能用。
2. 现有的强化学习算法框架比较成熟了, like stable-baselines，接口简单易用。
3. 股票交易的Environment，github上还没有找到比较好用的工具，这个只能自己靠自己写了。上面写的一个简单的env，没有考虑交易费用/允许做空等情况，还要进一步细化。


## 参考

1. [quantML-github-chiechie](https://github.com/chiechie/quantML/blob/master/gym_rl.py)
2. [强化学习算法框架-stable-baselines](https://github.com/hill-a/stable-baselines)
3. [强化学习环境框架-gym](https://www.oreilly.com/radar/introduction-to-reinforcement-learning-and-openai-gym/)
4. [构建交易环境-medium](https://towardsdatascience.com/creating-a-custom-openai-gym-environment-for-stock-trading-be532be3910e)
5. [构建交易环境和交易策略-github](https://github.com/wangshub/RL-Stock)
6. [PPO-youtube](https://www.youtube.com/watch?v=5P7I-xPq8u8)
