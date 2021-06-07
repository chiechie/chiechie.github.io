---
title: PageRank算法
author: chiechie
mathjax: true
date: 2021-05-22 18:07:11
tags: 
- 图算法
- 拓扑数据
categories:
- 实践
---

> 最近需要对图数据进行分析，了解下PageRank的原理
> 
> BTW, PageRank还可以做社区划分？

## 总结一下

pagerank:
假设，网上冲浪的人都是漫无目的的人，他们在网页一个链接接一个链接点下去，但是，整个互联网的节点，最终流量可能的分布能达到一个稳态。
也就是说，随机分布于各网页的流量经过次数足够多的转移之后，会达到一个稳定的状态，这个也代表每个网页的信息度。

基于这个假设，可以构建一个模型，
$$V_n = M ^n * V_0$$

一般来说，可以拿到网页之间调用拓扑，经过转换，可以将这个拓扑变为概率转移矩阵M，最终算出稳定态的V_n

给定点之间的连接关系，输出每个节点的分数，

补充下，最naive的方法存在终止点问题，也就是说，一个网站它不链接任何别的网站，或者一个微服务，它不调用其他任何微服务
按照上面的思路很可能出现一种情况，最终的流量全到这个自大狂网页那里去了，所以有一个改进的思路，假设冲浪着有一点点聪明，他并不是一味的接受灌输的链接，而是有一定概率，主动跳出去，到达一个新的站点，这个方式可以用下面的式子建模：

$$V_n = d * M * V_{n-1} + (1-d) * e $$

e = [1/n,...1/n]

d表示按照当前页面推荐的链接继续点下去的概率
1-d表示从当前网页跳出来，主动输入一个新网页重新开始的概率

## 代码

用代码验证一下，理解没有问题

https://github.com/chiechie/BasicAlgo/blob/main/pagerank.py

```shell
ground truth
 [[0.25419178], [0.13803151], [0.13803151], [0.20599017], [0.26375504]]
result
 [0.25419178 0.13803151 0.13803151 0.20599017 0.26375504]
```



## 参考
1. [pagerank-csdn](https://blog.csdn.net/gamer_gyt/article/details/47443877)
2. [scikit-network-pagerank](https://scikit-network.readthedocs.io/en/latest/tutorials/ranking/pagerank.html)
3. [wiki-PageRank](https://zh.wikipedia.org/wiki/PageRank)
4. https://blog.csdn.net/google19890102/article/details/48660239
