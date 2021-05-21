---
title: chapter2.2 多维下钻根因定位 
author: chiechie
mathjax: true
date: 2021-05-21 16:05:42
tags:
- AIOps
- 根因分析
categories: 
- AIOps
---


# 数据分析

比赛提供了两份数据集
Anomalytime_data_test1.csv: 异常时刻

2019AIOps_data_test1/*.csv：多维度指标

- 异常时刻：

  有200个异常时刻，即16个小时的异常。

- 多维指标数据：

测试数据集从2018-09-15 00:00:00,到 2018-09-28 23:55:00，频率为5分钟一条数据，供14天
多维时序：有5个维度
dim1 147
dim2 13
dim3 9
dim4 35
dim5 5

LEAF的个数，即layer 5的elements个数：

- 理论上：按照所有维度值的遍历，理论上应该有212 5760条曲线
- 实际上：每天最细粒度有3 0011条曲线

（paper中做实验的数据LEAF有21600个）

layer1($C^1_5 = 5​$)

layer2($C^2_5 = 10$)

layer3($C^3_5 = 10$)

layer4(($C^4_5 =5$个cubics) $B_{H,I,J,K}$, ...

layer5 $B_{H,I,J,K,L}$








处理非均衡样本

1. 分类问题

2. 回归:

   1. 直接使用z_score（异常分值）当成目标

   2. 使用z_score * 贡献度当目标

      2优于1，why？因为我的z_Score已经除了贡献度的导数

   ![企业微信截图_15560959535368](../_image/多维度分析.png)





1. 异常注入，需满足条件： 异常值引起了整体kpi的异常，
2. 什么指标可以刻画某个维度为根因的概率？



## 百度的方案：

参考人工定位过程中的分析思路，提取了两个特征，用来描述某维度是否为根因：

1. **贡献度，**即该维度PVLost与总PVLost的比例。（这个我们也有）

2. **一致度，**即构成该维度的子维度的异常程度的相似度。子维度的异常程度的一致度可通过各子维度异常程度间的变异系数衡量，变异系数越小，则异常程度越一致。(如何衡量小？绝对值还是相对值?)

   <https://zhuanlan.zhihu.com/p/48926992>







[TOC]


# 问题描述

<http://iops.ai/competition_detail/?competition_id=8&flag=1>


在运维场景中对监控指标的多个维度进行根因定位。具体来说，当某个总指标（如总流量）发生异常时，需要快速准确地定位到是哪个交叉维度的细粒度指标（如“省份=北京 & 运营商=联通”的流量）的异常导致的，以便尽快做进一步的修复止损操作。图示如下：

# 算法框架

调研hotspot，整体算法框架如下：

![image-20190322210515877](../_image/image-20190322210515877.png)

- 对KPI使用N-sigma进行实时异常检测，如果异常进入2
- 开始定位：
  for  layer=1到layer= L:
	- for  cuboid in  layer=l 的cuboids:
  	1. 剪枝：cuboid l中如果的element的属性值，没有出现在l-1(上一层的Bset中)，那就把elements去掉，剪掉之后生成备选的elements给2用。
  	2. mcts：使用蒙特卡洛搜索树寻找最有可能是根因的维度组合$B_{set}$，这里的reward function = potential score（potential score表示该维度为故障根因的概率)：
  		for 迭代次数(即蒙特卡洛数的深度) 从1 到 100：
  			1.select;
  			....
  		返回ps最大的维度
  	3. 如果potential score超过指定PS或者迭代时长超过给定值或者所有维度都遍历过了，迭代终止，返回ps最大的维度值。




# 符号说明

![image-20190322211056826](../_image/image-20190322211056826.png)



$e_i$: $e=(p, i, d, c) $，表示多个属性值的组合，是有具体的取值的.![image-20190322212044105](/Users/stellazhao/Library/Application Support/typora-user-images/image-20190322212044105.png)

$v(e_i)$:某个维度上指标的实际值。

$f(e_i)$:某个维度上指标的预测值。

cuboids:粒度可以任意细，任意粗  $ \left\{B_{P}, B_{P, I}, B_{P, I, D}, \ldots\right\}$
layer1，layer2，。。。layer3的解释如下
![image-20190322220603157](../_image/image-20190322220603157.png)



data cube: 多维的时序数据

ps: 衡量一组$S=\{e_i\}$是根因的概率。

e的后继结点：$\operatorname{Desc}(e)=\left\{e^{\prime} | e^{\prime} \text { is a descendant of } e\right\}$

e的后继LEAF结点：$\operatorname{Desc}^{\prime} (e)=\left\{e^{\prime} | e^{\prime}  \left(p^{\prime},i^{\prime}, d^{\prime}, c^{\prime}\right) \in L E A F, e^{\prime} \in \operatorname{Desc}(e) \right\}$

为什么要单独定义LEAF维度？因为它具有可加性，加起来等于它的父节点,如下

$\begin{array}{c}{v(e)=\sum\limits_{e^{\prime} \in D e s c^{\prime}(e)} v\left(e^{\prime}\right)} \\ {\text { e.g. }} \\ {v(\text { Bei jing }, *, *, *)=\sum\limits_{j, k, h} v\left(\text {Beijing}, i_{j}, d_{k}, c_{h}\right),} \\ {\text { Total } P V=v(*, *, *, *)=\sum\limits_{i, j, k, h} v\left(p_{i}, i_{j}, d_{k}, c_{h}\right)}\end{array}$



# ripple effect

# potential score 

##  potential score的计算公式

- ps的计算公式如下

  $\text { Potential Score }=\max \left(1-\frac{d(\vec{v}, \vec{a})}{d(\vec{v}, \vec{f})}, 0\right)$

  - a表示某维度确为根因的假设下，LEAF的预期值
  - v表示LEAF的真实值
  - f表示LEAF的预期值，跟疑似维度无关

  原理就是：比较在疑似故障这个假设下，LEAF相对真实值的区别，跟没有这个假设的前提下，LEAF相对真实值的区别。这两个区别越大，越说明这个假设是显著的。

  强行令他大于0，是因为想取到0~1 之间的值，当成概率。

- potential score指标计算公式基于的假设是：如果某个维度是故障的根因，那么在这个维度上的kpi的变化率 跟 它 的所有后代维度（descendant）的实际变化率应该是一致的。所以可以使用某个维度的kpi变化率减去后代的kpi变化率，基于这个difference的值算根因概率potential score，这个差值越大，根因概率越小，反之亦然。

- 这个绝对变化是 预测值（历史均值f）和实际值(v）的差(delta)。

## potential score计算步骤示例

假设现在最细粒度的LEAF维度有如下6个取值

$\vec{y}$=[(Beijing;Mobile);(Shanghai;Mobile);

(Guangdong;Mobile); (Bei jing;Unicom);

 (Shanghai;Unicom); (Guangdong;Unicom)].

那么

 $\vec f = (20;15;10;10;25;20)$, 

$\vec v= (14;9;10;7;)$

对于cuboid--$B_p$,它包含3个属性值（elements）:

[(Beijing;\*);(Shanghai; \*);(Guangdong;*)]

所以$B_p$的所有子集有7个，上面3个ele的排列组合：

$S_{p 1}=\{(B e i j i n g, *)\}$,

$ S_{p 2}=\{(\text {Shanghai}, *)\}$

....

$S_{p 7}=\{(\text {Beijing}, *), \quad \text { (Shanghai,*) },(\text {Guangdong,*}) \}$

![image-20190322212356628](../_image/image-20190322212356628.png)

- 注：上图中箭头左边是预测值($\vec{f}$)，右边的是真实值($\vec v$)

  现在以$S_{p1}=\{(B e i j i n g, *)\}$,$为例，算它的根因概率（ps），步骤如下

（1）如果$S_{p1}$是故障根因，受其影响，最细维度（LEAF）的预测值为：

- (a)$S$不是LEAF

  - $y_i$ 不属于$D e s c^{\prime}(S)$（非leaf后继）

    $a\left(y_{i}\right)=f\left(y_{i}\right)$

  - $y_i$ 属于$D e s c^{\prime}(S)$(leaf后继)

    $a\left(y_{i}^{\prime}\right)=f\left(y_{i}^{\prime}\right)-h(x) \times \frac{f\left(y_{i}^{\prime}\right)}{f(x)},(f(x) \neq 0)$

- (b)$S$是LEAF:

  - $y_i$ 不属于S， $a(y_i)= f(y_i)$

  - $y_i$ 属于S，$ a(y_i)= v(y_i)$

$\vec{a}\left(S_{p 1}\right)=(14,15,10,7,25,20)$

   (2) 最细维度（LEAF）对应的真实值为:

$\vec{v}=\left[v\left(y_{1}\right), v\left(y_{2}\right), v\left(y_{3}\right), \ldots, v\left(y_{n}\right)\right]$

  (3) 不管$S_{p1}$是不是故障根因，最细维度（LEAF）上面的预测值为:

$\vec{f}=\left[f\left(y_{1}\right), f\left(y_{2}\right), f\left(y_{3}\right), \ldots, f\left(y_{n}\right)\right]$



# 蒙特卡洛树搜索

## 符号说明

s： 状态，表示每个cubic中的elemments的集合,如$s\{e_1, e_2, ...\}$

$A(s)$: 表示状态=s时的所有action集合，值就是其中1个element。表示在s选定之后，还可以往里面加入的elements，这个直接用该cuboid的所有terminal-node\s中的elements。

$N(s)$: 表示s被访问的次数，s就是维度集合

$N(s;a)$: 表示边(s;a) 被访问的次数

$Q(s, a)$: 表示action function，取$s^{\prime}$的ps和$s^{\prime}$后继节点的ps的较大值。

$Q(s, a)=\max _{u \in\left\{s^{\prime}\right\} \cup \operatorname{descendent}\left(s^{\prime}\right)} p s(S(u))$

蒙特卡洛搜索树是强化学习框架下的一种数值算法。



##执行步骤

1）选择（selection）：根据当前的s和Q选取最优的action

$a=\underset{a \in A(s)}{\arg \max }\left\{Q(s, a)+C \sqrt{\frac{\ln N(s)}{N(s, a)}}\right\}$

上式中C(*)表示exploration， 左边的Q表示。

Upper Confidence Bounds（UCB）

初始状态N(S, A)=0，怎么办？对这些未被访问的(s,a),赋予一定的访问概率R

$$R=1-Q\left(s, a_{\max }\right)$$, ---（没懂，为什么Q越大，概率越小）

$a_{max} = \arg\max \limits_{a \in A(s) \cap N(s, a)=0} Q(s, a)$

selection起始于根节点，终于LEAF节点,这个值得是维度组合的叶子节点，如$\{e_1, \dots, e_{12}\}$

（错误：注意，终止于LEAF的维度值，而不是树的叶子节点）。

![image-20190325133227795](../_image/image-20190325133227795.png)

2) expand：起于上一部选定的分支，终于其子节点。在里面加一个ele。

![image-20190325133205483](../_image/image-20190325133205483.png)

3）evaluation：计算2)$s'$的ps，Q，和N

![image-20190325133147984](../_image/image-20190325133147984.png)

4) 反向传播（Backpropagation）：更新

从树的根节点到$s'$的路径中，更新经过节点的Q和N。

注意：只有当子节点的Q > 父节点的Q时，我们采取更新父节点的Q

![image-20190325133126318](../_image/image-20190325133126318.png)

MCTS 将在每个迭代过程（也就是1到4）中增加一个子节点。不过，要注意其实根据不同的应用这里也可以在每个迭代过程中增加超过一个子节点。

对每个cubic使用MTCS，迭代终止，如果以下条件之一满足：

1）$$B S e t=S$$ if $$ p s(S) \geqslant P T$$

2)   集合中所有可能的节点（维度值）都已经expanded完了

3）迭代时长超过给定值（根据经验人工确定）



# 层次剪枝

为了进一步减少搜索空间 ，HotSpot使用了层次剪枝的策略。

这里剪枝不是剪的蒙特卡洛搜索树，是剪的原始的action空间。

基本原理是：

由于MTCS是一层一层搜索的cuboids的，在搜索lay较小的cubics，例如（layer=1的$B_p$）

就可以剪掉一些ps较小的elements，因为这些elements的后继节点也不太可能是根因。

## 符号说明

$$B S e t_{l, B}$$: 使用MCTS对layer=l的cubic B, 算出来的ps最大的Set。

hotspot剪枝的策略：对每层l中的elements不在$$B S e t_{l, B}$$, 剪掉。这个策略跟关联规则挖掘中的Apriori Principle很像，叫层次剪枝这个名字是因为利用到了layer的信息。



## 示例

![image-20190325135442670](../_image/image-20190325135442670.png)



假设第一层的维度MCTS搜索出来的结果：
$$B S e t_{1, B_{P}}=\{(F u j i a n, *),(\text {Jiangsu}, *)\}$$并且$$ps(B S e t_{l, B_P})=0.5$$

$$B S e t_{1, B_{I}}=\{(*,Mobile),(*, \text {Unicom})\}$$并且$$ps(B S e t_{l, B_I})=0.32$$

现在MCTS要去搜索第二层了

就把 (Zhe jiang; Unicom) 和 (Zhe jiang;Unicom)这两个elements给剪掉了，因为他们的父节点 (Zhe jiang;)不在第一层维度的BSets里面，因此，第二层维度，我们只搜索剩下的4个elements，看哪种组合最优可能是根因的维度集合。

上面的剪枝将蒙特卡洛搜索树的action空间从63减少到了15 ($2^6- 1$ 到 $2^4 -  1$). 

然后使用蒙特卡洛搜索树搜到得分最高的set，即根因维度

$$R S e t=B \operatorname{Set}_{2, B_{P, I}}=\{(\text { Fujian, Mobile), (Jiangsu, Unicom) }\}$$且$$ps(B \operatorname{Set}_{2, B_{P, I}})=1$$



----

没想明白

1. 要训练集干嘛？

![image-20190412193557043](../_image/image-20190412193557043.png)

训练集用来当做历史数据，训练一个生成模型，对测试数据进行异常检测。

2.为什么要注入异常？

真实中故障很少发生，并且故障根因大部分都比较简单，不会出现多层，多个elemnts是根因的情况。

那我们研究这些极端情况就缺少样本了，所以需要人工构造。



# 技术方案

## 整体框架

1.在训练集上使用ripple effect 注入异常，训练模型（用hotspot跑一下）。~~

（问题：维度里面的‘unknown’是代表缺失还是汇总‘*’,先默认是缺失）

1. 数据清洗，把维度值里面的‘unknown’清除。
2. 在测试集上，直接使用算法进行根因定位,
   1. 样本量太大了，先使用两天的数据跑一下，2018-09-15 00:00:00 到 2018-09-16 23:55:00
   2. 异常检测就使用3-sigma策略，

- ~~对KPI使用N-sigma进行实时异常检测，如果异常进入2~~（读取给定的label列）

- 开始定位：
  for  layer=1到layer= L:

  ​	for  cuboid in  layer=l 的cuboids:

  如果l>=2, 剪枝：cuboid l中如果的element的属性值，没有出现在l-1(上一层的Bset中)，那就把elements去掉，剪掉之后生成备选的elements给2用。

  否则，直接进入下面：

  1. mcts：使用蒙特卡洛树搜索寻找最有可能是根因的维度组合$B_{set}$，这里的reward function = potential score（potential score表示该维度为故障根因的概率)：

  2. $Q(s, a)=\max _{u \in\left\{s^{\prime}\right\} \cup \operatorname{descendent}\left(s^{\prime}\right)} p s(S(u))$，初始值就是ps(S(s))

  3. $a=\underset{a \in A(s)}{\arg \max }\left\{Q(s, a)+C \sqrt{\frac{\ln N(s)}{N(s, a)}}\right\}$

  4. $a_{max} = \arg\max \limits_{a \in A(s) \cap N(s, a)=0} Q(s, a)$

  5. for 迭代次数(即蒙特卡洛树的深度？？) 从1 到 100：

     - select：从根节点出发，逐层选择最优的action，深度遍历。

     ​	a. 当所有子节点都被访问过，那么根据QCB选择最优的action，并且state跳到子节点（探索和利用），进入下一层select

     ​	b. 如果存在一部分没有被访问过的子节点，那么以概率R = 1 - max(Q) 访问未被访问过的子节点中Q值最大的那个

     - 如果选到的action刚好对应着没有被访问过的子节点，select终止		

      - 如果选到的action还是已经访问过的节点，那么select继续，进入下一层select

     $$R=1-Q\left(s, a_{\max }\right)$$, ---（没懂，为什么Q越大，概率越小）

     $a_{max} = \arg\max \limits_{a \in A(s) \cap N(s, a)=0} Q(s, a)$

     如果state

     1. 游走到了leaf_node(没有子节点)，那么select终止

     2. 游走到了某个没有被访问过的节点（即b中的概率选择），那么select终止

        （【原始的mcts】select返回的是node + action

          【hotspot的mcts】seletc返回的是node ）

     - expand：

       【原始的mcts】 将上一步选定的action 和 state对应的node，作为输入，构造mcts的新的node，也就是上一步的node的子节点。

       【hotspot的用法】上一步只选定node ，作为本步的输入，然后从validElementsSets中选择一个ps得分最高的element作为扩展（expand）的edge

     - evaluation(快完成了): 根据expand 中输出的edge，初始化一个新的节点s'，并且计算ps, Q和N

     - backup: 更新s‘的所有父节点的Q和N(对父节点来说，后继节点集合变大了，对应着，它自己的Qvalue也要更新了)

       $Q(s, a)=\max _{u \in\left\{s^{\prime}\right\} \cup \operatorname{descendent}\left(s^{\prime}\right)} p s(S(u))$

     返回ps最大的维度

  6. 如果potential score超过指定PS或者迭代时长超过给定值或者所有维度都遍历过了，迭代终止，返回ps最大的维度值。



## 实验记录

![image-20190330182941694](../image-20190330182941694.png)



## 一些细节问题



1.关于多维时序存储的问题，需要每一层的维度值对应的时序值都存一遍吗？

存最细粒度的，需要的时候再算？

答案：可以，异常时刻少，提前算好各种维度值对应的时序数据不现实（维度组合很大很大），并且大部分有可能白算了 （被剪枝掉了）。

2.selection里面具体的逻辑是怎样的？

![image-20190401212315297](../_image/image-20190401212315297.png)

3. mcts多轮迭代是不是都是从根节点的初始状态开始的？

   - 不是的，是从根节点的最新状态开始的，迭代始终在维护的是一颗树。每一次迭代，实际上是在构造树的分支，更新叶子节点，以及更新树的所有节点的visit和Qvalue

   - 从代码上看，通过保存node，而node会指向parent和children，实际上是保存下来了整棵树的，所以每次迭代，都是基于之前访问记录和Q_value的计算结果上进行的。

   

4. node在什么时候会更新visits和Q_value？unvisited nodes为什么会有Q_value？

   1. 单个element组成的set的Q_value是有初始值的，
   2. 多个elemnts组成的set的Q_value在evaluation处才会被计算？（不确定）
   3. selection的第一层循环完成（找到一条直达叶子节点的路径），才更新这条路径上的node的visits。

5. leaf node指的是什么?

   by 作者：

   ​	leaf node指的是当前状态（搜索树）下的叶子节点，并不是指的围棋的最后一步。

   ![image-20190403150404040](/Users/stellazhao/tencent_workplace/gitlab/dataming/algorithm_doc/process-model/_image/image-20190403150404040.png)



![image-20190403150626303](/Users/stellazhao/tencent_workplace/gitlab/dataming/algorithm_doc/process-model/_image/image-20190403150626303.png)

只是应该是当前树上的叶子节点而已。因为要不停的迭代，这里只是阐述一次迭代中的 四步



6. 计算ps的效率太低，怎样提升？

   1. 设计缓存机制，给定的属性值组合（LEAF和非LEA）时，提升真实值和预测值的计算效率。

      - [x] LEAF: LEAF的真实值和预测值使用频率高，任何属性值的ps都要基于这两个值，==所以把它当成全局变量存在内存中。==
      - [ ] 非LEAF:满足可加性（待定）。

   2. 在计算ps时，LEAF中包含一些无效的属性值（即真实数据中不存在属性值对应的记录）：（4月11日）

      1. - [x] 这些LEAF的真实值和预测值都为0，se==t在这些LEAF上的a, v, f 都为置为0==在value上设置就好了，再就是计算a_value，直接置为0

         $a\left(y_{i}^{\prime}\right)=f\left(y_{i}^{\prime}\right)-h(x) \times \frac{f\left(y_{i}^{\prime}\right)}{f(x)},(f(x) \neq 0)$

         $$\text { Potential Score }=\max \left(1-\frac{d(\vec{v}, \vec{a})}{d(\vec{v}, \vec{f})}, 0\right)$$

7. 怎样在小数据集上先跑通实验验证效果？

   - [x] ==将dim_dict先设置为3个维度(已经完成)。

8. 在mcts第二次迭代的时候，使用ucb挑选子节点，发现已经被访问过的子节点，其父节点的访问次数为0，

   这说明在第四步-反向传播（Backpropagation）：更新从树的根节点到$s'$的路径中，经过节点的Q和N。==这里更新N没有生效。==

   - [x] 需要再review一下。(重新看着文章理了一遍框架已经解决，)

 ```python
   UCB = c.wins + sqrt(2*log(self.visits)/c.visits)
 ```

9.发现state，在新加进来action之后，state非常奇怪，混杂了几种类型的数据（一个tuple，一个unicode）

![image-20190410162449504](../_image/image-20190410162449504.png)

- [x] ==于是，1.考虑对维度值构建索引：==

满足：

1. 无序

2. 同时包含dim和value的信息

   方案：

   “*&r01&e1&e&*”

   2. - [x] ==对维度值集合构建索引？==这也要考虑一个无序的问题

      --不用索引，就是维度值索引的list

10.在evaluation里面addchild时发现action为None，说明上面一步（expand）返回了一个None过来。

![image-20190410201934530](../_image/image-20190410201934530.png)

解决方法：

在select后面加上了一个break条件：如果该节点已经是叶子节点了

11。发现返回的BSet的Qvalue是1.0？

沿着数据处理流，回溯

Qvalue是1.0--- 》进而观察到ps算的有问题--> 进而发现v和f都是0，。。。--->所有的LEAF都是无效LEAF， 确实取值就是0

12.【已解决】发现在evaluation里面，计算新node的Q_value时，日志打印出在算一个不存在的维度值(

```shell
*&*&*&*&i01&*&*&*&*
```
)的ps.

什么原因呢？

因为传进去的self.state是1个list，里面有两个元素， [u'i02&*&*&*&*', u'i01&*&*&*&*']，在算list的时候，合成1个维度算了。

思考：这个state表示的是什么？

应该是从根节点到该节点，所有的边（element）的集合，S.

那这个新的复合的element，怎么算ps呢？

- [x] ==--也当成一个新的element算，value的计算方式要重写==(4月11日，完成)

![image-20190410221024023](../_image/image-20190410221024023.png)

13 发现对一个layer里一个cuboid的一个element计算ps，花费了大量的时间（算了快1个小时？时间主要耗费在计算a_vector上，无效的LEAF太多）

[工程]优化方案：

- [ ] 1.pandas  ---> modin.pandas(在 04上装了一下，但是提示报错, 好烦，不想弄了)

- [x] 2.==使用多进程，算a_value（使用10个核？），但是预期只能提升10倍==。（周末搞一搞）

- [x] 3.将LEAF_value再砍少一点，==valid LEAF在告警时刻做过滤（搞了，现在算一个element的ps要10分钟）==

   做个试验：本来的valid LEAF 有767592个，如果是在报警的时刻做一下过滤，可以减少到3056个；（缩小了100倍，靠谱，可以搞一搞）

4. 看了一下文章，优化方向有：

  - [x] 1.mcts中对termina-node的定义，文中是把n设置成了5，==先取2试一试==

   - [x] 2.mcts中对迭代次数的定义M，从5到15，==先取5试一试==

   - [x] 3.mcts的精度要求PT, 取的0.75，==先取0.6试一试==

   - [x] 4.搜索cuboids的层次可以再定义小一点，现在考虑的是2。（但是竞赛中给的故障数据应该是注入的，所以这个改小了应该会极大的降低准确率。）

     ![image-20190415172954161](../_image/image-20190415172954161.png)

5. mcts重写一遍，按照mcts-survey-master

![image-20190412200517988](../_image/image-20190412200517988.png)





- 现在的问题还是跑不动
      `{
      "dim1":[
          "i06",
          "i38",
          "i46",
          "i14",
          "i02"
      ],
      "dim2":[
          "e08",
          "e10"
      ],
      "dim3":[
          "c1",
          "c5"
      ],
      "dim4":[
          "p19",
          "p10",
          "p35"
      ],
      "dim5":[
          "l3",
          "l4"
      ]
  }`

15.做层次剪枝的时候，是在每一层的所有cuboid里面取最大的Qvalue对应的一个维度值吗？

不是哦，每一个cuboid都要保留一组最好的element

16。代码运行的太慢了？根因在哪里？怎么解决？

![image-20190417124043016](/Users/stellazhao/Library/Application Support/typora-user-images/image-20190417124043016.png)

1.箭头表示调用关系，由调用者指向被调用者；

2.矩形的左上角显示模块或者函数的名称，右上角显示被调用的次数；

3.矩形中间显示运行时间和时间百分比

发现60%的时间都在计算中间数据。





第一版优化：将getcache和setcache重写，初始化从磁盘载入，后面存内存。

dataframe在切片的时候，只保留之后的结果， 由60s变成了30s

![image-20190417124515665](../_image/image-20190417124515665.png)

 

第二版优化（关键优化）

将dim的value映射成整数，而后每次计算ps只需要花费6s。



后面的优化：

1. 可以看到有一些故障时刻挑选出来的qvalue并不高。

   很有可能的原因是，small_dict_json里面没有包含这些维度的信息。

   后面的解决方案：

   新生成一份small_dict_json





## 遍历layer1 到layer5的cuboids

layer1，cuboids-$B_{DIM3}$
$B_{DIM3}$有 9个elements，
$B_{DIM3}$的集合有$2^9 - 1$个：
$$S_{dim3,1}=\{(C1, *)\}$$
$$S_{dim3,2}=\{(C2, *)\}$$
$$S_{dim3,3}=\{(C3, *)\}$$
$$...$$
$$S_{dim3,8}=\{(C8, *)\}$$
$$S_{dim3,9}=\{(C1, *), (C2, *)\}$$
$$...$$

而LEAF的elements个数为2125760个（实际远小于这个数，有很多无效的叶子节点。）
LEAF的elements长这样
$$y_{1}=\{(i51, e01, c1,p01, l1)\}$$
$$y_{2}=\{(i49, e01, c1,p01, l1)\}$$
$$...$$

先计算单个elemets组成的集合, 也就是($S_{dim3,1}$~ $S_{dim3,8}$)的ps得分：

根据hotspot给的计算公式

- (a)$S$不是LEAF

  - $y_i$ 不属于$D e s c^{\prime}(S)$（非leaf后继）

    $a\left(y_{i}\right)=f\left(y_{i}\right)$

  - $y_i$ 属于$D e s c^{\prime}(S)$(leaf后继)

    $a\left(y_{i}^{\prime}\right)=f\left(y_{i}^{\prime}\right)-h(x) \times \frac{f\left(y_{i}^{\prime}\right)}{f(x)},(f(x) \neq 0)$

- (b)$S$是LEAF:

  - $y_i$ 不属于S， $a(y_i)= f(y_i)$
  - $y_i$ 属于S，$ a(y_i)= v(y_i)$

可以算出来a

接下来算ps-score

$\text { Potential Score }=\max \left(1-\frac{d(\vec{v}, \vec{a})}{d(\vec{v}, \vec{f})}, 0\right)$

每一个



![img](../_image/Screenshot-2019-01-22-at-12.48.45-PM.png)







## 模型的评估指标

###  准确性
评估准确性的指标是F-score,该指标是准确率（Precision）和召回率(Recall)综合体现。具体计算如下所示：
F-score =(2 ∗ Precision ∗ Recall)/(Precision+ Recall)，其中：
Precision ＝ TP / (TP + FP)，
Recall = TP / (TP + FN)。

每个异常时刻都有一个真正的根因集合，记为S*，该集合中包含一个或多个属性值组合，参赛队伍的算法输出结果 记为S。对于S*中的每一个元素，S中包含其中一个，则算一次true positive （TP），遗漏一个算一次false negative （FN），多出一个S*中不存在的，记一次false positive （FP）。计算出所有异常时刻总的TP、FP、FN，最后得出F-score。

### 实时性

决赛阶段除了对结果准确性的考量，还有对实时性的要求；注意预赛阶段不考察实时性。

实时性的要求方法是：测试数据模拟实时数据，每隔固定时间间隔生成新的数据及该时刻是否异常的信息，参赛者程序需在发生异常后的规定时间内提交该时刻的结果。如，t时刻有异常，则需在t+Δt（如Δt为1分钟）内提交t时刻的定位结果，超出时间则结果无效。

### 提交结果

1 提交结果必须为csv文件, 文件名必须为英文。

2 csv文件格式示例：
timestamp,set
1501475700,a1&b2;a3&b4
1501475760,a1&b2&x3;a4&b5&x6

注意：提交csv文件 header 必须为：timestamp,set

这里就是200行时间戳ddd

3 结果文件中的timestamp列必须与“问题描述”中 表3 中的异常时刻一一对应。




使用

```python
Thresh_EEP = 0.01
Thresh_EP = 0.70
```

找到的根因结果：

(1539376500, ([[0, 8, 0, 0, 0], [0, 4, 0, 0, 0]], 0.1084257376888971, 0.706533377277869))

但是使用

```
 Thresh_EEP = 0.00001
 Thresh_EP = 0.90
```

找到的根因结果：

```python
 [[0, 0, 0, 28, 0],
  [0, 0, 0, 29, 0],
  [0, 0, 0, 10, 0],
  [0, 0, 0, 19, 0],
  [0, 0, 0, 15, 0],
  [0, 0, 0, 21, 0],
  [0, 0, 0, 11, 0],
  [0, 0, 0, 1, 0],
  [0, 0, 0, 13, 0],
  [0, 0, 0, 17, 0],
  [0, 0, 0, 7, 0],
  [0, 0, 0, 22, 0],
  [0, 0, 0, 3, 0],
  [0, 0, 0, 24, 0],
  [0, 0, 0, 18, 0],
  [0, 0, 0, 14, 0],
  [0, 0, 0, 2, 0],
  [0, 0, 0, 4, 0],
  [0, 0, 0, 5, 0]]
```

在tableu上分析

[0,4,0,29,0]自己很异常，并且孩子们都很异常，疑似根因，）

![image-20190525161238775](../_image/image-20190525161238775.png)

换一个

[0,4,0,28,0], 异常贡献不到100，这个就不用考虑了

![image-20190525161119701](../_image/image-20190525161119701.png)

1539567000000





![image-20190525211118347](../_image/image-20190525211118347.png)

![image-20190525211508867](../_image/image-20190525211508867.png)

![image-20190525211624239](../_image/image-20190525211624239.png)



1540539600000

![image-20190525213427775](/Users/stellazhao/Library/Application Support/typora-user-images/image-20190525213427775.png)

![image-20190525213705573](/Users/stellazhao/Library/Application Support/typora-user-images/image-20190525213705573.png)


![image-20190525213819017](/Users/stellazhao/Library/Application Support/typora-user-images/image-20190525213819017.png)



## 代码地址

1. https://github.com/chiechie/ADminer



## 参考

1. [Yongqian Sun, Youjian Zhao, Ya su, et al., “HotSpot:Anomaly Localization for Additive KPIs withMulti-Dimensional Attributes”, IEEE Access, 2018](https://netman.aiops.org/wp-content/uploads/2018/12/sunyq_IEEEAccess2018_HotSpot.pdf)
2. [2](https://mp.weixin.qq.com/s/Kj309bzifIv4j80nZbGVZw)
3. [智能运维 | 百度多维度数据分析实战](智能运维 | 百度多维度数据分析实战
)
