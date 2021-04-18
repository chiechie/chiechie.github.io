---
title: zhoubolei-强化学习7-基于模型的强化学习方法
author: chiechie
mathjax: true
date: 2021-04-18 20:18:50
tags:
- 强化学习
- 人工智能
categories:
- 技术
---

## 课前--大大问号

- model-based 是什么？
- 对环境建模是什么意思？
- 提出该方法是为了解决什么问题？有哪些应用场景。
- 类似的流派或者同级别的 不同的流派 还有什么？
- model-based方法是怎么实现的？
- 在多大程度上能解决背景中的问题。有没有引发新的问题？
- 有没有理论支撑？
- 不能做什么？
    - 有理论支撑的话，理论上能证明解决不了什么问题？
    - 工程或者落地时的难点？
- 我们的基于 股票预测结果 构造策略 可以reformulate 成强化学习中的 什么方法？

## 尝试回答v1
  - model-based是强化学习中 一种技术，核心思想就是对环境建模，使用计算机构建仿真环境。
  - model-based是为了解决 样本获取成本较高的问题。例如，无人驾驶 /无人机，如果让机器在真实交通环境中行驶，并且通过于真实环境互动来获取数据，那必然以发生多次严重车祸作为获取样本的巨大代价。在机器人场景中应用较多。
  - 跟model-based相对的概念是model-free，它直接通过跟环境交互来获取样本，假设现实中获取样本的成本几乎为0。现在的强化学习论文中，大部分是采用的这个方法。
  - 对环境建模：输入state action， 输出 下一个 state reward ？从这个角度来说 。我们的基于 股票预测结果 构造策略 可以reformulate 成强化学习中的 什么方法？ 就是一个model-based强化学习框架。

## 参考资料

1. [bilibili：周博磊课程](https://www.bilibili.com/video/BV1hV411d7Sg)
2. [知乎：model-based和model-free模型优缺点](https://www.zhihu.com/question/318703290/answer/751123263)
3. [知乎：model-based方法介绍](https://zhuanlan.zhihu.com/p/72642285)
4. [知乎：model-based和model-free模型优缺点](https://www.zhihu.com/question/318703290/answer/751123263)
