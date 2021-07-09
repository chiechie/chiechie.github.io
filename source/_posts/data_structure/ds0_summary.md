---
title: 数据结构0 总览
author: chiechie
mathjax: true
date: 2021-07-08 09:17:29
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

## why 数据结构？

1. 数据结构是对数据的一种组织形式，以方便后续数据被高效地使用。
2. 数据结构是创建快速高效算法的必要ingredeints，数据结构可以帮助管理和组织数据；让代码看起来更干净。
3. 数据结构/抽象数据类/程序的关系就好像，使用乐高搭建一个建筑，层层抽象。

### 抽象数据类型和数据结构

2. 抽象数据类型(Abstract Data Type, ADT)是对数据结构的一个抽象，它只提供接口（interface），数据结构必须遵循该接口（interface），但是这个接口不涉及任何关于实施或者编程语言相关的细节。
3. 抽象数据类型的数据结构的关系，举个例子，从一个地方A到另一个地方B有很多种方法，可以骑自行车或者走路或者坐火车，前者是抽象数据类型，后者是该抽象数据类型对应的具体的数据结构。
5. 一个抽象数据类型之定义了数据结构应该实现什么功能，应该有哪些方法，至于方法的实现细节抽象数据类型是不管的。
4. 举几个抽象数据类型和数据结构的例子
    - 抽象数据类型列表（list）可以使用动态数组或者链表实现，这两个数据结构都能实现add，removing，indexing元素。
    - 抽象数据类型队列（Queue）可以使用Linked list based Queue/Array based Queue/Stack based Queue。
    - 抽象数据类型（Map）可以使用Tree Map/Hash Map/Hash Table实现
1. ADT is a logical description and data structure is concrete. 
2. ADT is the logical picture of the data and the operations to manipulate the component elements of the data. 
3. Data structure is the actual representation of the data during the implementation and the algorithms to manipulate the data elements. 
4. ADT is in the logical level and data structure is in the implementation level.

### 计算复杂度分析

1. 为了分析我们的数据结构的性能：执行该算法需要花费多少时间和占用多少内存？
2. O(*)表示在worst case的情况下，一个算法复杂度上界是多少，当输入数据变得很大的时候，有助于评估性能, 一般来说有这么几种时间复杂度:
    - 常数时间: O(1)
    - 对数时间： O(log(n))
    - 线性时间： O(n)
    - linearithmix时间: O(nlog(n))
    - 二次时间： O(n^2)
    - 三次时间: O(n^3)
    - 指数时间：O（b^n）
    - 因子时间: O(n!)

    n表示输入的大小复杂度从低到高。

3. 因为O（*）只关注当数据变得足够大的时候，算法的表现，所以算法的实际运行时间中的常数项，低阶项都可以省掉，系数都可以扔掉哦。
    $O(n + c) = O(n)$
    $O(cn ) = O(n)$
    $O(n^2 +2*n) = O(n^2)$
4. 二分查找是用在一个有序数组上的查找算法，时间复杂度是对数
5. 找一个集合的所有子集-时间复杂度是指数O(2^n)
6. 找一个字符串的所有排列O(n!)
7. 使用合并排序算法来排序，时间复杂度是 O(nlogn)  


## 数据结构有哪几种？

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


## 参考
1. [algorithm-github](https://github.com/williamfiset/Algorithms)
1. [youtube](https://www.youtube.com/watch?v=gXgEDyodOJU)
2. [Graph-theoretic Models](https://www.youtube.com/watch?v=V_TulH374hw)
3. [the difference-between-ADT和DS](https://stackoverflow.com/questions/13965757/what-is-the-difference-between-an-abstract-data-typeadt-and-a-data-structure)
4. [数据结构和算法](https://www.csie.ntu.edu.tw/~htlin/course/dsa21spring/)
5. [Data Structures Easy to Advanced Course](https://www.youtube.com/watch?v=RBSGKlAvoiM&t=102s)