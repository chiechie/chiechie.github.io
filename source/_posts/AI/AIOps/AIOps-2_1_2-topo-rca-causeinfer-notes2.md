---
title: chapter2.1.2 基于拓扑的根因定位-CauseInfer2
author: chiechie
mathjax: true
date: 2021-03-03 22:50:18
tags:
- AIOps
- 根因分析
- 微服务
- 图数据
categories: 
- AIOps

---

## 目录

- [chapter0 概览](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-0-summary/)
- [chapter1 故障发现](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-1-event-generate/)
	- [chapter1.1 单指标异常检测](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-1_1-kpi-detector/)
	- [chapter1.3 故障预测](https://chiechie.github.io/2021/03/04/AI/AIOps/AIOps-1_2-fault-prediction/)
	- [chapter1.4 指标异常关联](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-1_4-kpi-correlation/)
	- [chapter1.5 日志聚类](https://chiechie.github.io/2021/02/19/AI/AIOps/AIOps-1_5-log-analysis/)
		- [chapter1.5.1 使用logmine加强版做日志聚类](https://chiechie.github.io/2021/03/04/AI/AIOps/AIOps-1_5_1-log-analysis_logmine/)
		- [chapter1.5.2 美团日志聚类](https://chiechie.github.io/2021/03/04/AI/AIOps/AIOps-1_5_2-log-analysis_meituan/)
	
- [chapter2 故障定位](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-2-event-analysis/)
	- [chapter2.1 微服务系统的故障定位](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-2_1-topo-rca/)
		- [chapter2.1.1 CauseInfer1](https://chiechie.github.io/2021/03/02/AI/AIOps/AIOps-2_1_1-topo-rca-causeinfer-notes1/)
		- [chapter2.1.2 CauseInfer2](https://chiechie.github.io/2021/03/03/AI/AIOps/AIOps-2_1_2-topo-rca-causeinfer-notes2/)
		- [chapter2.1.3 AIOps挑战赛2020-获奖方案分享](https://chiechie.github.io/2021/03/10/AI/AIOps/AIOps-2_1_3-topo-rca-aiops2020/)
		- [chapter2.1.4 AIOps挑战赛2021-demo方案](https://chiechie.github.io/2021/03/09/AI/AIOps/AIOps-2_1_4-topo-rca-aiops2021/)
		- [chapter2.1.5 N-Softbei2020比赛](https://chiechie.github.io/2021/03/10/AI/AIOps/AIOps-2_1_5-topo-rca-cnsoftbei2020/)
		- [chapter2.1.6 MicroCause](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-2_1_6-topo-rca-MicroCause)
	- [chapter2.2 多维下钻根因定位](https://chiechie.github.io/2021/05/21/AI/AIOps/AIOps-2_2-multi-dimensional-rca/): 暂无
	- [chapter2.3 调用链根因分析](https://chiechie.github.io/2021/03/15/AI/AIOps/AIOps-2_3-trace_rca/)
	- [chapter2.4 时间序列关联性分析](https://chiechie.github.io/2021/04/14/AI/AIOps/AIOps-2_4-metric_event_correlation/)
- chapter3 故障恢复

> 本文是《CauseInfer论文笔记》的第二篇，[《CauseInfer论文笔记1》](https://chiechie.github.io/2021/03/02/technology/causeinfer-notes1/)，从high-level的角度描述了causeinfer的工作流。这里要开始涉及到方案的具体的实现了，所以内容比较硬核
>
> 不做方案实现的话，可以不用看这篇文章

## 因果建模

### 构建服务依赖图

服务依赖图长什么样？一个DAG：

- 节点（node）：一个服务。
- 边（edge）：代表服务依赖关系。


这块主要是工程性工作，需要数据采集。

- 服务依赖图是什么？以服务为节点，服务间的调用关系为边，构造出来的DAG。

- 针对本方案的应用场景，有3个假设前提：
  a. 所有服务都是使用TCP作为传输层的协议，
  b. 本文的方法依赖网络统计工具，和kprobe来探测系统调用（system call）。根据观察，大部分应用都是采用TCP协议来通信，如mysql，tomcat等；几乎所有主流的操作系统-如linux-都已经跟网络统计工具集成了，例如netstat，tcpdump 和 kprobe。
  c. 利用流量延迟，来决定服务依赖图中的依赖的方向，而不是用分析结果来确定图的结构，
  这一部分可以大大减少错误风险， 以及减少计算复杂度。 

- 本文用tuple来表示一个服务，即 (ip, service name)，chiechie： 为什么作者能抽象出来，我抽象不出来啊？为什么不像有的文章那样，用three-tuple表示(ip, port, proto)，因为考虑到一个服务可能占用多个端口。举一个例子，在一个三层的系统中，一个web server可以通过任意一个端口访问application server的。如果，使用端口作为唯一服务的属性，这个服务依赖图，就会变得非常之大，即使所有请求都是由同一个服务发起的。

> 补充：
> 
> 在分布式系统中，IP表示一台唯一的主机，服务名称表示 运行在该主机上的 唯一的服务。

- 服务依赖的定义：如果服务A需要服务B提供特定功能，来满足 客户端的请求。就说A->B. 箭头表示「依赖」的意思。举个例子，一个web service需要获取一些数据库的内容，就依赖db服务，就可以说web服务->db服务。本文只考虑client-server类型的应用（还有什么类型的应用?）

- 怎么获取服务依赖图？
  - 第一步是搭架子：使用服务间的连接信息来构建 服务依赖图 的骨架（skeleton）。通过运行工具-netstat（有点像一个采集器，类似istatus，小米手环），在一个主机中，我们可以拿到关于一个网络连接的 一堆信息，包括协议，来源，目的地，连接状态。提取来源和目的地信息，每一个连接可以表示为这个形式：source_ip:port -> desination_ip:port. 这一组信息叫一个channel。channel类似，服务依赖pair，除了一点：channel不包含服务名称，只包含port。
  - 第二步是端口映射：将第一步获取到的服务的端口映射为一个服务的名称。本地服务名称可以通过查询端口信息获取（netstats）。但是对于一个remote端口，需要传输一个查询请求给remote的主机，来获取。将端口映射为服务名之后，一个本地主机的服务依赖图的架子就搭好了。
  - 第三步是调整依赖方向：由于client和server是双向传递的，即，在不同的主机中观察时，我们可能拿到一个反向的服务依赖。举个例子：
    - 当在192.168.1.117中观察时，可以获取一个连接：(192.168.1.117,httpd) → (192.168.1.115, tomcat)，即httpd调用了tomcat。
    - 但是在192.168.1.115中观察时，又能得到这个连接：(192.168.1.115,tomcat) → (192.168.1.117, httpd), 即tomcat调用了 httpd。

  为了解决这个问题，使用流量滞后关联的方法：
 
  X是服务A的出流量的，Y是服务B的出流量
  衡量X和Y之间的滞后的相关性可以用这个指标，k

$$
\rho_{X Y}(k)=\frac{\sum_{t=0}^{N-1}\left(Y_{t}-\bar{Y}\right)\left(X_{t-k}-\bar{X}\right)}{\sqrt{\sum_{t=0}^{N-1}\left(X_{t}-\bar{X}\right)^{2} \sum_{t=0}^{N-1}\left(Y_{t}-\bar{Y}\right)^{2}}} k \in Z
$$
k取什么值合适呢？平移后两个序列的相关性最大，如下，即为最优的$k^{*}$
$$
k^{*}=\left\{\operatorname{argmax}\left(\left|\rho_{X Y}(k)\right|\right), k \in[-30,30]\right\}
$$

如果$k^{*}>0$,则A是B的因（cause），A导致了B, 记为 $$A \rightarrow B $$
如果$k^{*}<0$,则B是A的因（cause），B导致了A，记为 $$B \rightarrow A $$


### 构建指标因果图

如何指标因果图？需要用到「因果推断」技术，从数据得到因果关系。这里使用的是[PC算法](https://chiechie.github.io/2021/03/09/technology/PC-algo/)
> pc算法是一种发现因果关系的算法，在满足一定的假设前提下，使用的基于统计的方法，推导出因果关系。

1. 构建因果图的架子
2. 确定因果关系的方向

基于PC算法，我们提出了两种方法：保守的 和 激进的。 激进的意思是不使用任何先验知识，来构建因果图。 保守的意思是，使用一些先验知识初始化。

先验知识是什么呢？ 比如哪些变量没有父母--根因指标，哪些变量没有孩子--表象指标。 比如本地服务的网络延迟（TCP_LATENCY）指标就没有孩子，只是表象指标，因为网络延迟不会导致其他指标的变化。背后的原因可能是
工作负载（workload），配置错误，依赖服务的延迟。


#### 激进派算法

如果一个服务（比如db服务）只会被别人调用，那么指标因果图就只会使用该服务的本地性能指标（服务依赖图依赖的数据一起采集了）。

但是一个服务，会调用别人，如web service，这个因果图的构建，不仅仅需要本地的性能指标，还需要它调用的服务的TCP LATENCY指标。

训练数据的长度取的是默认值200，然后使用PC算法，来构建因果图，可以得到一个DAG。

激进派构建出来的因果图中，会有反直觉的因果关系和双向links；可能包含多个独立的子图，由于缺乏evidence，统计错误，
或者 压根儿就没有因果关系。

举个例子，Figure 6 (a)(下图左边)的因果图是激进派算法构建的：
- M5是一个孤立的服务
- M4 → M2 的因果关系是反直觉的
- M1 和 M4的因果关系是双向的。

![图2-激进法构建出来的因果图](inference.png)

所以激进派构建出来的因果图需要进一步的加工：根据以下限制条件，从DAG中选择最大的子图

a. 子图裁剪：TCP LATENCY指标是表象指标，他不可能是别的指标的cause了 
b. 表象指标通过图中的每条路径都可以达到。
c. 预设的根因指标没有父母。
d. 对于一个双向的连接，随机选择一个方向。（这就是随机游走的意思？）

#### 保守派算法

保守派算法，跟激进派很像。区别在于，使用PC算法之前，保守派会利用先验信息做初始化DAG。
和激进算法比，保守算法可以捕捉更多的因果关系，减少反直觉的因果关系。

举个例子(下图右边)，M1 → SLO的因果关系，激进算法会丢失，但是保守算法会保留。同时
反直觉的因果关系，M4 → M2也被消除了

![图3-保守派构建出来的因果图-右边](inference.png)

## 因果推断


因果模型构建好了，接下来就需要根据这个因果图去做因果推断了。

### 整体思路

当前端的服务可用性指标（SLO）出现异常，就会触发根因分析：

- 首先找自己的问题：使用指标因果图 推断本地， 导致服务性能问题的根因指标。
- 其次，如果问题是别人造成的，就去找别人的问题：如果根因指标是依赖服务的SLO（注意，用到了调用链关系），这个推断就会继续，传播到远程的依赖的服务。
一直追本溯源，一直到最底层的被调用方，即物理层。


### 指标因果图根因推断

指标因果图

如何定位一个节点上哪个指标的是故障根因？
 
- 使用了DFS算法--深度优先搜索，对指标因果图进行遍历。
- 访问到一个节点，对它进行异常检测（cusum）：
  
  - 如果异常，继续访问他的后继（descendants）节点，
  - 如果正常，就去访问的邻居节点。
- 如果一个异常节点没有后继节点，无法再溯源了，或者它的后继节点都正常时，他就是根因。

![图3-推断.png](inference.png)

如果SLO异常，从SLO节点开始推断，有两个推断路径

- 路径1： 检测M1，如果M1正常的，就访问M1的邻居M2，如果M2异常，就是根因。
- 路径2.：接下来M3如果异常，我们就去检测M2，因为M2是异常的，所以我们就输出根因M2。

最后，我们只找到了一个根因M2，虽然M2和M3都是异常的。

然而，在某些情况下，因为多个因果路径存在，可能得到一系列的潜在根因集合。
因此，必须把这些根因进行排序，然后输出概率最大的根因。

如何去给根因进行排序呢？找异常程度最大的那个指标，也就是对异常贡献度最大的指标，就是根因。
这里基于z-score方法提出了一个指标，来衡量性能指标的异常程度，这个评估指标是

$$\operatorname{violation}(X)=\frac{X(t)-\overline{X_{t-60, t-1}}}{\sigma_{t-60, t-1}+\varepsilon}, \varepsilon=0.001$$


- $\overline{X_{t-60, t-1}}$ ：滑动窗口的均值
- $\sigma_{t-60, t-1}$ ：滑动窗口的标准差
- $\varepsilon$ : 扰动项


## 参考文献

1. [2014-INFOCOM_CauseInfer](https://netman.aiops.org/~peidan/ANM2016/RootCauseAnalysis/ReadingLists/2014INFOCOM_CauseInfer.pdf)
2. [2007-The Journal of MachineLearning Research-pc算法](https://www.jmlr.org/papers/volume8/kalisch07a/kalisch07a.pdf)
3. [别人对CauseInfer论文的解读](https://saruagithub.github.io/2020/04/13/20200413CauseInfer%E8%AE%BA%E6%96%871/)
4. [chiechie对PC算法的总结](https://chiechie.github.io/2021/03/09/technology/PC-algo/)
5. [CauseInfer论文笔记1](https://chiechie.github.io/2021/03/02/technology/causeinfer-notes1/)