---
title: 数据结构1 线性数据结构
author: chiechie
mathjax: true
date: 2021-07-09 12:13:11
tags:
- 编程
- 数据结构
- 图数据
categories: 
- 数据结构
---


# 总结

1. 数组（array），链表（linked list），栈（stack），队列（queue）是最常用的线性的数据结构。
2. 数组是一片连续的存储空间中，链表在物理空间中不是连续地存在一起的。

#  通用的数据结构

## 数组(array) 

1. 数组是一个很基本的数据结构，使用数组和pointer可以构建出所有的数据结构
1. 数组分为静态数组（static array）和动态数组（dynamic array）两种.静态数组大小固定，动态数组可以扩展。
  
   > wangshusen的视频里面分别叫array和向量vector，下面还是以动/静为标准。

2. 静态数组(static array)是一个固定长度的容器（container），包含了n个元素，每个元素有一个索引，索引值从[0,n-1]
3. 动态数组（dynamic array）的大小可以增加或者减少。
3. indexable是什么意思？数组中的每一个槽位(slot)/索引(index)，都能关联到（be referenced with）一个数字。索引就好像给每个元素取的名字，是一个阿拉伯数字。
4. 静态数组是存储在内存中的一片连续的区域。
5. 什么时候使用静态数组：
   - 依次（sequentially）存储元素/读取（accessing）元素
   - 在读入/输出流式数据的时候，一次性读入数据量太大，所以分chunk读入数据，并且用buffer去读取每个chunk的数据，buffer就是用array实现的
   - 在lookuptable中，也会用到数组，因为可以按照index读取数据
    - 如果某个函数只允许返回1个，但是有多个信息需要返回，可以使用array构造一个workaround，将多个信息打包成一个array，然后返回这个array的指针或者reference
    - 动态规划：用于缓存子问题的answer，例如背包问题（knapsack problem）和 应变变换问题。
6. 静态数组和动态数组的时间复杂度
    - 读取的时间复杂度: 都是O(1)，因为数组是indexable的
    - 查找的时间复杂度：都是O（n），有可能查找的值找不到
    - 插入某个元素的时间复杂度：静态数组不允许插入（记住，他是一个长度固定的容器），动态数组插入的时间复杂度是O（n）- 添加的时间复杂度：
    - 删除：
7. 获取静态数组中元素的唯一方式，就是通过index去reference（数组的成员只对外暴露他们的index编号）。
8. operations on 动态数组：静态数组上能做的操作，动态数组都能做，还能做的更多，比如增，删。
9. 怎么实现一个动态数组？
    - 方法1--使用静态数组实现：
      - 初始化：创建一个静态数组，有一个初始容量，此后一遍增加元素一遍跟踪容器中元素的个数。
      - 添加新元素时，如果容器没满，直接加入； 如果容器满了，自动扩容--创建一个新的静态数组，大小为当前容量的2倍，将老的静态数组的元素复制到新的静态数组中，然后加入该元素。


## 链表(linked list)

1. 链表上相邻的元素在物理存储上并不是相邻的，每个元素的物理未知只存在于他的上下游节点中。
2. 链表分为单链表（singly linked lists）和双链表(doubly linked lists）。
3. 按顺序查找链表也很快。
4. 链表是一种数组的组织形式，将数据组织成一个有序的一系列节点（nodes）的形式，每个节点（node）代表一个数据，每个节点都指向其他代表数据的节点。
5. 下面是一个单链表（singly linked），每个节点都包含一个数据，同时包含一个指针，指向他的邻居节点。最后一个节点的指针是空的。

![img.png](img.png)
6. 在哪里会用到链表？ 
   - 需要实现某个抽象数据类型时会用到链表，比如要实现列表（lists），队列（Queue）和 栈（Stack），因为这些抽象数据类型需要频繁操作adding和remocving，而这两个操作对于链表来说是相当拿手的。
   - 链表很容易对现实事物建模，如火车
   - 创建循环列表时很有用，可以让链表的最后一个节点的指针指向第一个节点，训练列表对于建模重复事件循环很有用，
   - 实现哈希表通常使用链表处理冲突的问题。
7. 链表的几个关键元素：
   - head：指向链表的第一个节点的pointer，
   - tail：指向表示链表的最后一个节点的pointer，
   - pointer表示邻居节点的指针，
   - node：表示一个包含数据和指针的对象。在实现时，每个节点也可以被表示为structures，或者classes
8. 单链表和双链表的区别，单链表的每个节点只存储下一个邻居的pointer，双向链表的每个节点存储两个指针，分别是上一个邻居节点和下一个邻居节点。前者占用的内存是后者的1/2，缺点是不知道当前节点的前一个节点是什么，要找的话，只能从head开始遍历。如果要删除某个元素，双链表的时间复杂度是常数，单链表是线性的，因为他要重新遍历以找到上家，然后修改它的指针。
   ![单链表vs双链表-优缺点对比](img_1.png)
9. 链表中，同时存链表的第一个节点和最后一个节点，是为了更快速的添加和删除元素。
10. 链表中的复杂度分析：
   - 搜索某个值在链表中的位置：单链表和双链表的复杂度是O(n)
   - 在链表头部/尾部插入某个元素：单链表和双链表的复杂度是O(1)
   - 删除头部的元素：单链表和双链表的复杂度是O(1)
   - 删除尾部的元素：单链表的复杂度O(n)，双链表的复杂度是O(1)
   - 删除中间元素：单链表的时间复杂度为复杂度O(n)，双链表的复杂度是O(n)



## 栈（stack）

1. 栈是一种线性的数据结构，栈的一端是固定的，跟现实世界中的stack一样，stack有两个主要的操作：入栈（push）和出栈（pop）。
2. 栈中有一个top指针指向栈的顶端。以为对栈的操作主要是集中在顶端。
3. 数据出栈（pop）和入栈（push）符合后进先出的顺序，也叫LIFO/

   ![image-20210711073942878](/Users/shihuanzhao/research_space/chiechie.github.io/source/_posts/data_structure/image-20210711073942878.png)

1. 可以使用数组或者链表来实现一个栈。
5. 什么时候用到栈：括号匹配/撤销操作/汉诺塔/图遍历中的深度优先搜索。（DFS）

##  队列

1. 队列是一个线性的数据结构，有两个主要的操作，入队（enqueue/adding/offering）和出队（dequeue/polling）。入队就是添加数据到队尾添，出队就是删除队头的数据。

2. 数据入队和出队，符合后进后出的顺序。（LILO）

   ![image-20210711085139955](/Users/shihuanzhao/research_space/chiechie.github.io/source/_posts/data_structure/image-20210711085139955.png)

2. queue可以用于对排队场景建模；跟踪最新添加的k个数据；web server请求管理--谁先来就服务谁/图遍历中的广度优先搜索（BFS）。
3. 可以用链表来实现队列。
3. 使用队列实现BFS。
   ![image-20210711091627179](./image-20210711091627179.png)


# python中的数据结构

1. 在python中，list就是一个动态的数组，append有时候效率很低。
2. 在python中，collections.deque是一个double-ended queue，两端固定的队列，基于双向链表实现的插入效率更高一些。但是按照序号查找某个元素，效率不高。
3. 在python中怎么实现一个栈？

   1. 使用python内置的对象list，其自带的append和pop方法可以实现push和pop
   2. 使用collections.deque，其自带的append和pop方法可以实现push和pop



# 参考

1. [algorithm-python-github](https://github.com/akzare/Algorithms)
1. [youtube](https://www.youtube.com/watch?v=gXgEDyodOJU)
2. [Graph-theoretic Models](https://www.youtube.com/watch?v=V_TulH374hw)
3. [the difference-between-ADT和DS](https://stackoverflow.com/questions/13965757/what-is-the-difference-between-an-abstract-data-typeadt-and-a-data-structure)
4. [数据结构和算法](https://www.csie.ntu.edu.tw/~htlin/course/dsa21spring/)
5. [Data Structures Easy to Advanced Course](https://www.youtube.com/watch?v=RBSGKlAvoiM&t=102s)
7. https://realpython.com/how-to-implement-python-stack/#implementing-a-python-stack