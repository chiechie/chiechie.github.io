---
title: 强化学习5 蒙特卡洛树搜索
author: chiechie
mathjax: true
date: 2021-06-02 16:58:05
tags:
- 强化学习
- 人工智能
categories:
- AI
---



# 总结
1. 蒙特卡洛树搜索（MCTS）是一种基于模型的强化学习方法。
2. 蒙特卡洛树搜索比价值学习和策略学习更难。
3. AlphaGp是一个围棋AI，依靠MCTS做决策，决策过程中需要策略网络和价值网络辅助。
4. 假设已经有策略网络和价值网络，蒙特卡洛树搜索


# hot spot
## 算法框架

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





## sumamry

假设第一层的维度MCTS搜索出来的结果：
$$B S e t_{1, B_{P}}=\{(F u j i a n, *),(\text {Jiangsu}, *)\}$$并且$$ps(B S e t_{l, B_P})=0.5$$

$$B S e t_{1, B_{I}}=\{(*,Mobile),(*, \text {Unicom})\}$$并且$$ps(B S e t_{l, B_I})=0.32$$

现在MCTS要去搜索第二层了

就把 (Zhe jiang; Unicom) 和 (Zhe jiang;Unicom)这两个elements给剪掉了，因为他们的父节点 (Zhe jiang;)不在第一层维度的BSets里面，因此，第二层维度，我们只搜索剩下的4个elements，看哪种组合最优可能是根因的维度集合。

上面的剪枝将蒙特卡洛搜索树的action空间从63减少到了15 ($2^6- 1$ 到 $2^4 -  1$). 

然后使用蒙特卡洛搜索树搜到得分最高的set，即根因维度

$$R S e t=B \operatorname{Set}_{2, B_{P, I}}=\{(\text { Fujian, Mobile), (Jiangsu, Unicom) }\}$$且$$ps(B \operatorname{Set}_{2, B_{P, I}})=1$$


## 技术方案

1.在训练集上使用ripple effect 注入异常，训练模型（用hotspot跑一下）。

（问题：维度里面的‘unknown’是代表缺失还是汇总‘*’,先默认是缺失）

1. 数据清洗，把维度值里面的‘unknown’清除。
2. 在测试集上，直接使用算法进行根因定位,
   1. 样本量太大了，先使用两天的数据跑一下，2018-09-15 00:00:00 到 2018-09-16 23:55:00
   2. 异常检测就使用3-sigma策略，


- 开始定位：
  for  layer=1到layer= L:

  	for  cuboid in  layer=l 的cuboids:

  如果l>=2, 剪枝：cuboid l中如果的element的属性值，没有出现在l-1(上一层的Bset中)，那就把elements去掉，剪掉之后生成备选的elements给2用。

  否则，直接进入下面：

  1. mcts：使用蒙特卡洛树搜索寻找最有可能是根因的维度组合$B_{set}$，这里的reward function = potential score（potential score表示该维度为故障根因的概率)：

  2. $Q(s, a)=\max _{u \in\left\{s^{\prime}\right\} \cup \operatorname{descendent}\left(s^{\prime}\right)} p s(S(u))$，初始值就是ps(S(s))

  3. $a=\underset{a \in A(s)}{\arg \max }\left\{Q(s, a)+C \sqrt{\frac{\ln N(s)}{N(s, a)}}\right\}$

  4. $a_{max} = \arg\max \limits_{a \in A(s) \cap N(s, a)=0} Q(s, a)$

  5. for 迭代次数(即蒙特卡洛树的深度？？) 从1 到 100：

     - select：从根节点出发，逐层选择最优的action，深度遍历。

     	a. 当所有子节点都被访问过，那么根据QCB选择最优的action，并且state跳到子节点（探索和利用），进入下一层select

     	b. 如果存在一部分没有被访问过的子节点，那么以概率R = 1 - max(Q) 访问未被访问过的子节点中Q值最大的那个

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

     - evaluation: 根据expand 中输出的edge，初始化一个新的节点s'，并且计算ps, Q和N

     - backup: 更新s‘的所有父节点的Q和N(对父节点来说，后继节点集合变大了，对应着，它自己的Qvalue也要更新了)

       $Q(s, a)=\max _{u \in\left\{s^{\prime}\right\} \cup \operatorname{descendent}\left(s^{\prime}\right)} p s(S(u))$

     返回ps最大的维度

  6. 如果potential score超过指定PS或者迭代时长超过给定值或者所有维度都遍历过了，迭代终止，返回ps最大的维度值。

# 参考
1. [HotSpot-英文paper](https://netman.aiops.org/wp-content/uploads/2018/12/sunyq_IEEEAccess2018_HotSpot.pdf)
2. [HotSpot-中文](https://mp.weixin.qq.com/s/Kj309bzifIv4j80nZbGVZw)