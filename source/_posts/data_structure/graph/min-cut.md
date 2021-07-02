---
title: 图的最大流和最小割算法
author: chiechie
mathjax: true
date: 2021-07-02 08:48:14
tags: 
- 图算法
- 图
- 最小割
- 数据结构
categories: 
- 数据结构
---

## 最大流

### 最大流问题

### 最大流算法


## 最小割

### 最小割问题


最小割要解决的问题和最大流是一样的


输入：方向有权图
目标：割的容量最小
输出：某个S-T cut，


> 最大流最小割定理（Max-Flow Min-Cut Theorem）
> 
> 在一个网络流量中，从s到t的最大流量等于，最小s-t cut的容量。
> 
> --L. R. Ford and D. R. Fulkerson. Flows in Networks. Princeton University Press, \(1962 .\)


![](img_1.png)



### 寻找最小割的方法

1. 使用最大流算法获得residual graph， 移走其中反向的边
2. 在residual graph中，从起点s出发，找到所有能达到的节点，并记为集合S，把其他所有节点记做T（s到不了的节点）。
3. 将{S, T}记做最小割。


## 参考

1. [图的最小割算法-youtube](https://www.youtube.com/watch?v=Ev_lFSIzNh4&t=128s)
2. [图的最小割算法-slide](https://github.com/wangshusen/AdvancedAlgorithms)