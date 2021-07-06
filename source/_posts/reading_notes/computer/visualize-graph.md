---
title: 风控领域的图挖掘场景
author: chiechie
mathjax: true
date: 2021-05-24 09:08:21
tags:
- 图数据
- 图挖掘
- 连通子图算法
- 层次社区划分算法
- 社区发现算法
categories: 
- 阅读
---

# 几个场景

## 攻防对抗

一般都会有下述风控规则：

- 7天内在注册账户数超过**的设备号
- 1天内某IP关联超过xx个账户

虽然规则简单，但由于实时性和贴近业务的特点，可以拦截不少黑产, 但也会造成一定误杀，特别是IP类的规则。

同时因为风控是攻防对抗的过程，黑产也会升级，比如通过伪造设备、LBS、频繁更换IP等手段尽可能伪装成一个真人，隐蔽聚集性等显性特征，绕过风控规则。

相应风控技术也在进化，如使用设备指纹、IP画像、验证码对抗等。

## 基于关联关系识别团伙

可以定义设备-账户的Graph如下：

- 节点：账户 + 设备
- 边：近xx天账户与设备出现在同一事件中，则它们之间有一条边。
  
识别方法：

- 无监督方法：通过「连通子图算法」识别出一个个连通的社区，如果社区规模较大，可能背后业务含义是黑产控制一批账户。定义社区规模为score，通过调节阈值来控制误杀、召回。
- 有监督方法-传统：等价为节点分类问题，通过提取节点业务特征、拓扑特征、所属社区特征，训练一个分类器去预测。
- 有监督方法-图神经网络：将节点业务特征X与网络拓扑结构A作为输入学习函数，用于对未知数据的预测.相比规则来说，此类方法不仅用到了更复杂的关系，同时也考虑了节点业务和拓扑特征。故防控能力会更强一些。


## 基于相似度识别团伙

如果黑产成功避开设备指纹、聚集性规则等风控措施，把自己伪装成一个真人，如何检测？——只要作案了，总会留下蛛丝马迹。
   
- 垃圾文本：比如留下了垃圾文本，那么利用文本之间相似度（Jaccard、semi-hash）构建账户之间相似关系，然后使用图分割+连通子图查找技术识别，具体可[参考这里](https://zhuanlan.zhihu.com/p/23385044)
- 行为相似度：facebook针对刷量的行为，通过计算账户之间行为度来构建graph，[详见这里](https://zhuanlan.zhihu.com/p/58334765).此类方法最大问题是, 两两节点相似度计算性能问题。一般会做下约束，如限制在某个事件场景下、限制在某段事件内以及如何分段计算+合并，并且往往是通过spark分布式计算。


## 基于共享特征识别团伙

因为黑产控制大量的账户通过软件进行攻击，而不是手工操作，故这些账户某些共同属性上会有Pattern。

大概的解题思路是“搜集证据”，将共享的某特征/字段作为，判断证据强弱，如共享设备是强证据，可以仅凭此条直接断案。
但是像共用某品牌手机是弱证据，需要多个弱证据结合一起断案。

再抽象一点是有点像「层次社区划分算法」：

1. 定义证据：通过业务经验定义共享特征集，不通场景特征集应该是不同的。
2. 找证据：分别对每个/每组特征进行聚类/社区划分算法，得到一个个簇.
3. 组合证据：将2得到的簇作为节点，簇之间共有账户数作为连边，构建graph，再进一步划分得到最终的社区，每个社区中的节点共享多个特征。


# 总结

1. 上述三种类型方法的主要区别在于graph的不同，在整个图挖掘体系中最核心也最难一点在于，如何将业务经验抽象成Graph。
2. 社区发现算法以图分割+连通子图查找为主，一方面可解释强，另外可以通过调节阈值权衡召回和误差，以满足运营人员需要。这也是 "拿算法适配业务，而不是业务适配算法"的工作准则。

# 参考
1. [scikit-network的doc](https://scikit-network.readthedocs.io/en/latest/)
2. [zhihu-风控](https://www.zhihu.com/search?type=content&q=%E5%9B%BE%E6%8C%96%E6%8E%98)
3. [GCN](https://zhuanlan.zhihu.com/p/62750137)