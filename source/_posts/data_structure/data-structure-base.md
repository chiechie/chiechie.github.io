---
title: 数据结构基础
author: chiechie
mathjax: true
date: 2021-06-25 09:17:29
tags:
- 编程
- 数据结构
- 图数据
categories: 
- 数据结构
---

> 数据结构是在计算机中存储数据的方法
> 
> 一般情况下，精心挑选的数据结构能产生有效的算法


## 总结 


1. 数据结构分为线性数据结构和非线性数据结构
2. 线性数据结构包括数组（array），链表（linked list），栈（stack），队列（queue）
   
    ![线性数据结构](b3728c27302a8548fe9e8a87e619ca83.png)
3. 非线性数据结构包括树和图,树可以认为是有向图的special case
    ![非线性数据结构](e6d5a8d9a75587abe612dfef9abffc01.png)
   
4. 图分有向图和无向图
   
    ![有向图vs无向图](18c651092d22c7204021d10a5a79b0ff.png)
5. 无向图的一个实例是fb的社交网络，边表示好友关系。
   
    ![社交网络](f3fc896014d62fb1ec1c96c93210f7ff.png)
6. 基于社交网络这个数据结构有什么应用呢？好友推荐, 推荐朋友的朋友,网络社会科学的小世界

    > 小世界网络的重要性质：“流行病学”、“合作”、“知识”
   
7. 有向图的一个实例是万维网：
    ![www](b9b97250ce6e998045dcbb0d5b379724.png)

8. 图还可以分有权图和无权图。无权图可认为是权图的special case，权重都为1。
9. 有权图的一个实例是高速公路网,边代表距离
    ![公路网](5b81b50b2d2b048ed3188b71af85a02f.png)
10. 树的一个实例是家谱，树种，任意一对节点，有且只有一条通路（不存在loop嘛）
11. 因果图是一个有向有权图。


## 附录

### 抽象数据类型和数据结构

1. ADT is a logical description and data structure is concrete. 
2. ADT is the logical picture of the data and the operations to manipulate the component elements of the data. 
3. Data structure is the actual representation of the data during the implementation and the algorithms to manipulate the data elements. 
4. ADT is in the logical level and data structure is in the implementation level.



### 列表和链表

1. 列表（list）或序列（sequence）是一种抽象数据类型(ADT), 
2. 链表是一种数据结构。

## 参考

1. [youtube](https://www.youtube.com/watch?v=gXgEDyodOJU)
2. [Graph-theoretic Models](https://www.youtube.com/watch?v=V_TulH374hw)
3. [the difference-between-ADT和DS](https://stackoverflow.com/questions/13965757/what-is-the-difference-between-an-abstract-data-typeadt-and-a-data-structure)