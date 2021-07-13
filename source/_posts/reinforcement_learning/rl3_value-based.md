---
title: 强化学习3 基于价值的强化学习方法
author: chiechie
mathjax: true
date: 2021-04-21 20:30:11
tags:
- 强化学习
- 人工智能
categories:
- 强化学习
---



# 总结

1. 强化学习方法之一有价值学习方法，在价值学习中，如果我们能得到最优动作函数，就能知道agent在不同的state下该有如何的表现了。

2. 如何学习最优动作价值函数呢？目前最有效的办法就是DQN，Deep Q Network。DQN通过构造一个神经网路去拟合最优行动价值函数，然后通过与环境互动获得训练数据，从而训练出一个神经网络版的先知，他知道在每一个state下，每个action能带来的长期的最大价值。
   >  在离散的动作空间中，该价值网络输入的是state，输出每个action对应的Q值, 
   >
   >  那么如果是连续的决策呢？

3. 如何训练价值网络？最根本的问题在于如何获得训练样本，这个样本的feature为state，label为每个action对应的Qvalue。这个label怎么获取呢？
   1. 通过玩很多局游戏，然后记录每个state下面的最优的
   
4. Q-learning算法是一种时序差分算法（TD），是一种训练DQN的方法：

   - TD error:  未来的我回到现在来做决策，优于现在的我做决策的程度。
   - TD target：未来的我回到现在的世界来做决策，可以得多少分。
   
5. 轨迹 **(Trajectory)** 是指一回合 (Episode) 游戏中，智能体观测到的所有的状态、动作、奖励: s1,a1,r1, s2,a2,r2, s3,a3,r3, ···

6. DQN 的训练可以分割成两个独立的部分:收集训练数据、更新参数 w。

   1. 收集训练数据：可以使用行为策略(Behaviour Policy)让agent根环境互动，将一条轨迹的数据分成多个$(s_t, a_t, r_t, s_{t+1})$四元组，将四元组统一存放到经验回放数组(Replay Buffer)

   2. 更新DQN参数:从经验回放数组中取出一个四元组，使用时序差分（TD）算法更新参数

      1. 使用DQN计算当前时刻价值网络的预测值$\hat q_j$ ，下一个时刻价值网络的预测值  $\hat q_{j+1}$。

      2. 更新DQN参数的目标是为了让TD误差变小，所以可以将当前的学习目标设置为，最小化TD误差平方和，即：
         $$
         \min L(\boldsymbol{w})=\frac{1}{2}\left[Q\left(s_{t}, a_{t} ; \boldsymbol{w}\right)-\widehat{y}_{t}\right]^{2}
         $$
      
      3. 更新DQN的参数：根据样本中的$r_t$, 计算出TD误差$\delta_{j}$，对 DQN 做反向传播，得到梯度$\boldsymbol{g}_{j}=\nabla_{\boldsymbol{w}} Q\left(s_{j}, a_{j} ; \boldsymbol{w}_{\mathrm{now}}\right)$
         
         $\boldsymbol{w}_{\mathrm{new}} \leftarrow \boldsymbol{w}_{\mathrm{now}}-\alpha \cdot \delta_{j} \cdot \boldsymbol{g}_{j}$
         
         

# 附录 （自问自答）

## 几个问题：

1. 价值 指的是什么？

   「价值」 指的是 **最优动作价值函数**，即如果后续action都遵循**最优策略**，那么当前的<state，action>的得分。**最优动作价值函数** 相当于 一个 先知。

2. 怎么使用深度学习技术来求解这个问题？

   回到强化学习要解决的问题本身（MDP）--agent的目标是 打赢游戏（最大化total reward），那么我们将这个目标拆解为两个步骤，

   1. 找到最优动作价值函数
   2. 在最优价值函数的指引下作出最优决策

3. 使用深度学习技术来学习价值函数，难点在哪里？

   难点在于，谁都不知道先知（**最优价值函数**）长什么样子（股票的话，如果认为只有一条价格路径，可以算出来，如果只是众多路径中的一条，算不出来）, 不过只要有足够多的“经验”，就能训练出“先知”。

4. DQN的主要思路是什么？

   DQN的想法就是，使用一个深度神经网络拟合 **最优动作价值函数，**

3. 跟DQN并列的概念还有哪些？

4. DQN网络的输入输出是什么？中间结构是什么？

   DQN的输入是state，输出是所有action对应的分数。

5. DQN怎么更新参数？

   - 可以把一整局（episode）游戏玩完，搜集到反馈信号（total reward）再去更新DQN的参数。

   - 也可以玩一步，更新一下参数----就是TD算法

8. Q函数 和 reward 函数 是不是可以相互转化？--是这样吗？

   - Q 函数  ==  reward （来自 environment）+ 策略（来自 agent）
   - Q * 函数 == reward （来自 environment）+ 最优策略（来自 agent）

   不是啊，策略输出的是行为概率，Q是价值。量纲都不一样。

## 一图胜千言


- 价值网络
  ![价值网络](./imgs%2Fapp%2Frf_learning%2FfsQADMgSRa.png?alt=media&token=18bf844e-aa59-4016-85d6-8cdfc801a9ce.png)

- 策略网络

  ![策略网络](./imgs%2Fapp%2Frf_learning%2FGqFFfS975r.png?alt=media&token=71ba382a-432c-4c00-8759-692d84c03f3d.png)



## 时序差分算法（TD算法）

- 优化目标--最小化TD误差：
由最优贝尔曼方程（Optimal Bellman Equations）推导出理想情况下，Qvalue应该满足的表达式：

![image-20210713200249948](/Users/stellazhao/research_space/chiechie.github.io/source/_posts/reinforcement_learning/rl3_value-based/image-20210713200249948.png)

  并且将最小化左边和右边的误差平方 作为优化目标，来求Q，


- TD error:  对于上一步<$s_{t-1}, a_{t-1}$>的reward， 模型估值-真实值，有点像未来的我回到现在来做决策，优于现在的我做决策的程度。
    - 模型作出的估计（模型对上一个状态估计的价值 - 模型对当前状态估计的价值） -  真实 reward （来自 environment）
- TD target： 对于上一步<$s_{t-1}, a_{t-1}$>的长期value，未来的我回到现在的世界来做决策可以打多少分。
    - 真实reward（来自 environment） + 折扣* 模型对当前状态估计的价值
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FROMmZ-LTGx.png?alt=media&token=dae7cba7-84e1-4e7f-8638-60eaa2b1c166)
- ![时序差分算法](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FEuWKISitjj.png?alt=media&token=0427aa38-bad9-46f3-98bd-a1eb051df5cd)




## 时序差分算法的其他资料
1. Sutton and others: A convergent O(n) algorithm for off-policy temporal-difference learning with linear function approximation. In NIPS, 2008.
2. Sutton and others: Fast gradient-descent methods for temporal-difference learning with linear function approximation. In ICML, 2009.

