---
title: 强化学习1 资料汇总
author: chiechie
mathjax: true
date: 2021-04-18 20:16:31
tags:
- 强化学习
- 人工智能
categories:
- AI
---


## why rl?

可能可以超越人类。监督学习 只能模仿，不能超越

## 公开课

### zhoubolei：rf-intro

- zhoubolei: ：rf-intro
  
    - [zhoubolei-github-课程资料](https://github.com/zhoubolei/introRL) ｜[slide](https://github.com/zhoubolei/introRL/blob/master/lecture1.pdf)
    - [zhoubolei视频](https://www.bilibili.com/video/BV1LE411G7Xj)
- wangshusen
    - [baidu网盘](https://pan.baidu.com/s/1XpTgny_Vr0LobBsuYF4KkA):密码:x0wb
    - [强化学习中文教材](https://github.com/wangshusen/DRL/blob/master/Notes_CN/DRL.pdf)
    - [视频](https://youtu.be/vmkRMvhCW5c)|[深度学习课件](https://github.com/wangshusen/DeepLearning)|[深度强化学习课件](https://github.com/wangshusen/DRL)
- chiechie的笔记
    - 【基础】[1. 强化学习的基本概念](https://chiechie.github.io/2021/04/18/technology/wangshusen-rf1-basic-concepts/)
    - 【基础】[2. 基于价值的强化学习-DQN 和 TD 算法](https://chiechie.github.io/2021/04/18/technology/wangshusen-rf2-value-based/)
        - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FSSywH-1RRf.png?alt=media&token=6b811aba-b275-47f0-a6d3-2945b6fff817)
    - 【强化学习基础】[3. 基于策略的强化学习](https://chiechie.github.io/2021/04/18/technology/wangshusen-rf3-policy-based/)
        - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FVhwFEvq-CN.png?alt=media&token=de826d99-eaa2-4c92-8bec-520c7a04ee0b)
    - 【强化学习基础】[4. actor-critic 方法](https://chiechie.github.io/2021/04/18/technology/wangshusen-rf4-actor-critic/)
    - 【专题】价值学习高级技巧
        - [价值学习高级技巧1/3-经验回放](https://chiechie.github.io/2021/04/18/technology/wangshusen-rf5-advanced-value-based/)
            - 对TD算法的改进
            - 对神经网络结构的改进 
    - 【专题】[策略梯度中的baseline](https://chiechie.github.io/2021/04/18/technology/wangshusen-rf6-baseline-in-policy-gradient/)
        - Policy gradient (策略梯度) 方法中常用 baseline 来降低方差，加速收敛。Baseline 可以是任何不依赖于动作的函数。这节课我们做数学推导，得出带有 baseline 的策略梯度，并对它做蒙特卡洛近似。在后面的两节课中，我们把它用在 REINFORCE 和 A2C 方法中。
        - 策略梯度中的Baseline 1/4-数学推导
        - 策略梯度中的Baseline 2/4-REINFORCE with Baseline
            - Policy gradient (策略梯度) 方法中常用 baseline 来降低方差，加速收敛。这节课我们学习 REINFORCE with Baseline，它是一种策略梯度方法。它用价值网络近似 baseline 函数。它用带 baseline 的策略梯度学习策略网络，用价值网络去拟合观测到的回报。
        - 策略梯度中的Baseline 3/4-A2C 方法
        - 策略梯度中的Baseline 4/4-REINFORCE与A2C的异同
    - 【专题】[连续控制](https://chiechie.github.io/2021/04/18/technology/wangshusen-rf7-continous-action/)
        - 连续控制1/3-离散控制与连续控制
        - 连续控制2/3-确定策略梯度DPG
        - 连续控制3/3-用随机策略做连续控制

## 练手项目

- [使用强化学习炒股](https://github.com/wangshub/RL-Stock)
- [强化学习应用于金融问题的文章](https://zhuanlan.zhihu.com/p/267998242)
- [Gym-强化学习开发框架](https://gym.openai.com/)：开发强化学习算法的工具箱，有很多第三方环境


## 强化学习的应用

[强化学习的应用--chiechie](https://chiechie.github.io/2021/04/21/reinforcement_learning/rf-application/)

- 神经网络结构搜索
- 自动生成SQL语句
- 推荐系统
- 网约车调度

## 一些思考

### 价值学习，策略学习，动作裁判学习 这三种方法 存在的必要？

- 既然有了policy Function 或者 Q, 都可以告诉agent怎么操作。那有什么必要去做 actor-critic 算法呢？单独学两个神经网络不就可以了？
- 问题就是不容易学呀！所以才会有Actor-Critic这种复杂的方法。

### 强化学习 适合哪些应用场景？图像分类 适合用吗？为什么 auto-ml适合用？

- 适合 序列决策 问题， 图像分类没必要 用
- 训练一个大的神经网络类似玩一局游戏 耗时很久，每一个迭代 类似 游戏里面的 一小步，可以拿到即时reward，这样就可以使用TD算法来学习 value-function。

### 在线学习 和 强化学习的 区别？

- online learning 不假定模型输出会影响未来输入，只看单步损失
- reinforcement learning 中模型输出会影响未来输入，必须考虑输出的后果

### on-policy 和 off-policy 的 区别？

- 相同点：都是用来更新价值网络的学习算法
- 不同点是：on-policy是按照当前的policy来估计q-value；off-policy是按照最优的policy来估计q-value


  
### 监督学习 和 强化学习的区别

- 监督学习
    - 要求数据 是独立同分布的
    - 学习过程有老师手把手教，标准答案（label）是什么
- 强化学习
    - 数据不用iid
    - 没有导师（supervisor），不会被告知正确的action是什么，只有一个奖励信号，并且有延迟的。需要自己去试错，找到具长期reward最大的action
    
### 强化学习的特点

- 强化学习中随机性的两个来源：action可以是随机的，状态转移可以是随机的
- 试错机制
- 延迟reward
- 时间会产生影响
- agent的行为会影响后续收到的数据（agent的action改变了环境） 
  
  > 类似我们的模型自动更新，担心时间会对模型造成影响。比如模型推荐了商品a，我们也只能收到关于商品a的反馈。
    

## 参考
1. [on-policy和off-policy的区别-stackoverflow](https://stats.stackexchange.com/questions/184657/what-is-the-difference-between-off-policy-and-on-policy-learning)
2. [深度强化学习 在实际应用中少吗？难点在那里？](https://www.zhihu.com/question/290530992)
3. [宋一松SYS](https://weibo.com/titaniumviii?refer_flag=0000015010_&from=feed&loc=nickname)
4. [强化学习和在线学习的区别-zhihu](https://www.zhihu.com/question/64526936)