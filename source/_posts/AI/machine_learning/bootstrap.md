---
title: 一种重采样方法-resampling
author: chiechie
mathjax: true
date: 2021-05-15 16:29:28
tags:
- 统计学
categories: 
- AI
---

## bootstrap的假设

- 当前的数据资料是最能反映population信息的一批资料
- 绝大部分样本跟population中的个体很相似

![img.png](./img.png)

## 为什么bootstrap比一般的参数估计方法更有效？
![bootstrap比一般的参数估计方法更有效](./img_1.png)

bootstrap本质上是一种外推法（extrapolating to the population），更有想象力，当让也更有出错的可能

![img_2.png](./img_2.png)

## 参考
1. [Resampling methods-bootstrapping--youtube](https://www.youtube.com/watch?v=gcPIyeqymOU)