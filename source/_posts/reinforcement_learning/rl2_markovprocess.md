---
title: 强化学习2 马尔可夫过程
author: chiechie
mathjax: true
date: 2021-04-20 20:18:20
tags:
- 强化学习
- 人工智能
categories:
- 强化学习
---



>  强化学习就是一系列的决策过程，他试图找到一个长期收益最大的策略，马尔可夫决策过程可以看成是一个浓缩版的强化学习



# 马尔可夫过程/马尔可夫奖励过程/马尔可夫决策过程

## 基本概念

- 状态转移矩阵(P)：$p\left(s_{t+1}=s^{\prime} \mid s_{t}=s\right)$
    - $P=\left[\begin{array}{cccc}P\left(s_{1} \mid s_{1}\right) & P\left(s_{2} \mid s_{1}\right) & \ldots & P\left(s_{N} \mid s_{1}\right) \\ P\left(s_{1} \mid s_{2}\right) & P\left(s_{2} \mid s_{2}\right) & \ldots & P\left(s_{N} \mid s_{2}\right) \\ \vdots & \vdots & \ddots & \vdots \\ P\left(s_{1} \mid s_{N}\right) & P\left(s_{2} \mid s_{N}\right) & \ldots & P\left(s_{N} \mid s_{N}\right)\end{array}\right]$
    
- horizon：在每个episode中，最大时间步数

- 轨迹(episode)：从这个马尔可夫链中采样得到的，每采样一轮就得到叫一个轨迹，例如一局游戏

- 马尔可夫性质： future is independent of the past given the present， $s_t$是符合马尔可夫性质的： 
    - 给定历史状态： $h_{t}=\left\{s_{1}, s_{2}, s_{3}, \ldots, s_{t}\right\}$
    
      $$\begin{aligned} p\left(s_{t+1} \mid s_{t}\right) &=p\left(s_{t+1} \mid h_{t}\right) \\ p\left(s_{t+1} \mid s_{t}, a_{t}\right) &=p\left(s_{t+1} \mid h_{t}, a_{t}\right) \end{aligned}$$


## 马尔可夫过程（MP）

马尔可夫链 （MP， Markov Process）：$\left(S, P^{\pi}\right)$

- 一个状态转移如果符合马尔可夫，即一个状态的下一个状态只取决于当前状态，而跟他当前状态之前的状态都没有关系
- $P=\left[\begin{array}{cccc}P\left(s_{1} \mid s_{1}\right) & P\left(s_{2} \mid s_{1}\right) & \ldots & P\left(s_{N} \mid s_{1}\right) \\ P\left(s_{1} \mid s_{2}\right) & P\left(s_{2} \mid s_{2}\right) & \ldots & P\left(s_{N} \mid s_{2}\right) \\ \vdots & \vdots & \ddots & \vdots \\ P\left(s_{1} \mid s_{N}\right) & P\left(s_{2} \mid s_{N}\right) & \ldots & P\left(s_{N} \mid s_{N}\right)\end{array}\right]$
- ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2Fsby0HtEUFa.png?alt=media&token=1f6bf364-4cca-4535-9191-4b77bda51131)
- 从这个马尔可夫链 中采样，得到轨迹，每一轮叫一个episode
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FvZ2dIl6a_W.png?alt=media&token=03dda99b-181a-4b48-aec2-03f0679ab1de)

## 马尔可夫奖励过程（MRP）

马尔可夫奖励过程（MRP， Markov Reward Process）：$\left(S, P, R, \gamma\right)$

- MRP = MP + reward
- R是奖励函数：$R\left(s_{t}=s\right)=\mathbb{E}\left[r_{t} \mid s_{t}=s\right]$
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2Fo3iwF_-fJr.png?alt=media&token=0fd15a10-a6f0-4588-a019-1a258b22dc8d)horizon：在每个episode中，最大时间步数
    - Horion可以是无穷大，此外MRP又叫有限马尔可夫奖励过程
- 状态价值函数（state value function）：代表当前state下，未来所有rewards的折现
    - $\begin{aligned} V_{t}(s) &=\mathbb{E}\left[G_{t} \mid s_{t}=s\right] \\ &=\mathbb{E}\left[R_{t+1}+\gamma R_{t+2}+\gamma^{2} R_{t+3}+\ldots+\gamma^{T-t-1} R_{T} \mid s_{t}=s\right] \end{aligned}$
    - 折现因子是强化学习的一个超参，用来约束模型对长期收益的看重程度 
- 马尔可夫奖励过程的 bellman等式:$V(s)=\underbrace{R(s)}_{\text {Immediate reward }}+\underbrace{\gamma \sum_{s^{\prime} \in S} P\left(s^{\prime} \mid s\right) V\left(s^{\prime}\right)}_{\text {Discounted sum of future reward }}$
- 马尔可夫奖励过程的bellman等式的矩阵形式：
    - $V=R+\gamma P V$
        - $\left[\begin{array}{c}V\left(s_{1}\right) \\ V\left(s_{2}\right) \\ \vdots \\ V\left(s_{N}\right)\end{array}\right]=\left[\begin{array}{c}R\left(s_{1}\right) \\ R\left(s_{2}\right) \\ \vdots \\ R\left(s_{N}\right)\end{array}\right]+\gamma\left[\begin{array}{cccc}P\left(s_{1} \mid s_{1}\right) & P\left(s_{2} \mid s_{1}\right) & \ldots & P\left(s_{N} \mid s_{1}\right) \\ P\left(s_{1} \mid s_{2}\right) & P\left(s_{2} \mid s_{2}\right) & \ldots & P\left(s_{N} \mid s_{2}\right) \\ \vdots & \vdots & \ddots & \vdots \\ P\left(s_{1} \mid s_{N}\right) & P\left(s_{2} \mid s_{N}\right) & \ldots & P\left(s_{N} \mid s_{N}\right)\end{array}\right]\left[\begin{array}{c}V\left(s_{1}\right) \\ V\left(s_{2}\right) \\ \vdots \\ V\left(s_{N}\right)\end{array}\right]$
- 马尔可夫奖励过程的 状态价值函数（上面的矩阵V） 求解可采用2个方法
    1. 解Bellman方程，根据算出来的解析解，硬算逆矩阵，和矩阵乘法
        - $V=(I-\gamma P)^{-1} R$
        - 求解的复杂度n3，n表示state个数
    2. 使用迭代的算法：对state个数较多的情况可行
        - 动态规划![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FMSB3EJXzsN.png?alt=media&token=19f9a683-eec0-48f5-8296-2ac88a4ea3c1)
        - 蒙特卡洛估计（采样）：需要完成一次![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2F6s5BEi2xsO.png?alt=media&token=1bc3ed8d-81f3-4439-b895-845770f1f651)
        - 时间序列差分学习时间差分（TD）算法:TD（Temporal Difference）算法：
- bellman期望方程：
    - $v^{\pi}(s)=E_{\pi}\left[R_{t+1}+\gamma v^{\pi}\left(s_{t+1}\right) \mid s_{t}=s\right]$
    - 【参考】$V(s)=\underbrace{R(s)}_{\text {Immediate reward }}+\underbrace{\gamma \sum_{s^{\prime} \in S} P\left(s^{\prime} \mid s\right) V\left(s^{\prime}\right)}_{\text {Discounted sum of future reward }}$
    - $q^{\pi}(s, a)=E_{\pi}\left[R_{t+1}+\gamma q^{\pi}\left(s_{t+1}, A_{t+1}\right) \mid s_{t}=s, A_{t}=a\right]$
- 推导过程如下：
    - $\begin{aligned} v^{\pi}(s) &=\sum_{a \in A} \pi(a \mid s) q^{\pi}(s, a) \\ q^{\pi}(s, a) &=R_{s}^{a}+\gamma \sum_{s^{\prime} \in S} P\left(s^{\prime} \mid s, a\right) v^{\pi}\left(s^{\prime}\right) \end{aligned}$
    - 相互替换得到
        - $v^{\pi}(s)=\sum_{a \in A} \pi(a \mid s)\left(R(s, a)+\gamma \sum_{s^{\prime} \in S} P\left(s^{\prime} \mid s, a\right) v^{\pi}\left(s^{\prime}\right)\right)$
        - $q^{\pi}(s, a)=R(s, a)+\gamma \sum_{s^{\prime} \in S} P\left(s^{\prime} \mid s, a\right) \sum_{a^{\prime} \in A} \pi\left(a^{\prime} \mid s^{\prime}\right) q^{\pi}\left(s^{\prime}, a^{\prime}\right)$
- 计算图
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FDt61DqVl6N.png?alt=media&token=9736af0f-e3ef-457b-86d6-abae268af1a3)![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FB2u5I82qnF.png?alt=media&token=02a1de1d-4a08-4f43-983b-65eb815df328)
##  马尔可夫决策过程（ MDP）

马尔可夫决策过程（MDP， Markov Decision Process）： $\operatorname{MDP}(S, A, P, R, \gamma)$ 

- MDP能够对现实世界中很多问题建模，是研究强化学习的基础框架
- 一个马尔可夫决策过程由状态空间，决策空间，状态转移，奖励函数组成。
- MDP要求环境变量是完全可观测，但是现实中部分可观测问题也可转化为MDP问题
- 「马尔可夫决策过程」相对于「马尔可夫奖励过程」多了决策，所以需要建模的对象多了两个要素：action 和 概率转移$P\left(s_{t+1}=s^{\prime} \mid s_{t}=s, a_{t}=a\right)$。
    ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FbPx8H2l13W.png?alt=media&token=6e62ffdf-8434-40b4-b312-5037d805f911)
- 马尔可夫决策过程中的策略函数（policy evaluation）
- 马尔可夫决策过程中的控制：策略迭代和价值迭代

什么是MDP中的预测和控制？

- 预测问题：给定策略时，求某个状态的价值函数。（先知）
- 控制问题：最优的策略，和对应的状态价值函数。（军师）

## 马尔可夫过程/马尔可夫奖励过程/马尔可夫决策过程三者的关系

- 马尔可夫过程和马尔可夫奖励过程是马尔可夫决策过程的简化版
- 马尔可夫过程(MP)只有一个概率转移
- 马尔可夫奖励过程(MRP) = 马尔可夫过程(MP) + Reward，但是没有自由意志，还是随波逐流
- 马尔可夫决策过程(MDP) = 马尔可夫奖励过程 （MRP）+ action，多了自由意志
- ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FIuFRm5JvB6.png?alt=media&token=1ad94ce8-0886-4329-8902-dbf859b8ef22)


# 附录（自问自答）

1. model-based 是什么？对环境建模是什么意思？

   

   model-based是强化学习中一种技术，核心思想就是对环境建模，使用计算机构建仿真环境。比如构建一个模拟环境，更新状态和reward。

2. 提出model-based是为了解决什么问题？有哪些应用场景？

   

   model-based是为了解决样本获取成本高的问题。例如，无人驾驶/无人机，如果让机器在真实交通环境中行驶，并且通过于真实环境互动来获取数据，那必然以发生多次严重车祸作为获取样本的巨大代价。在机器人场景中应用较多。

   

3. 类似model-based的流派或者同级别的不同的流派 还有什么？

   

   跟model-based相对的概念是无模型（model-free）方法，它直接通过跟环境交互来获取样本，假设现实中获取样本的成本几乎为0。但是现在的强化学习论文大部分都采用model-free的方法，可见离落地还有一段距离。

4. model-based方法是怎么实现的？

   

   对环境建模：输入<state,action>，输出下一个<state,reward> 。

5. 在多大程度上能解决背景中的问题。有没有引发新的问题？

6. Model-based不能做什么？

7. 有没有相关理论支撑？理论上能证明解决不了什么问题？

8. 工程或者落地时的难点？

9. 基于股票预测结果构造策略 可以reformulate 成强化学习中的 什么方法？

   model-free的方法就可以，但是样本很少，因为历史不会重演。

   理论上是可以使用model-based的方法，从观测数据中估计生成模型的参数，然后从该分布中采样路径，构造多份合成数据，然后在这个模拟环境中，学习策略vs回测效果。

   


# 参考资料

1. [周博磊课程-bilibili](https://www.bilibili.com/video/BV1hV411d7Sg)
2. [知乎：model-based和model-free模型优缺点](https://www.zhihu.com/question/318703290/answer/751123263)
3. [知乎：model-based方法介绍](https://zhuanlan.zhihu.com/p/72642285)
4. [知乎：model-based和model-free模型优缺点](https://www.zhihu.com/question/318703290/answer/751123263)
