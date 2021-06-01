---
title: chapter1.5.2 美团的日志聚类
author: chiechie
date: 2021-05-21 11:01:18
categories: 
- AIOps
mathjax: true
tags:
- NLP
- AIOps
- 日志分析
- 论文笔记 
---

## 目录
- [chapter0 概览](../AIOps-0-summary/)
- [chapter1 故障发现](../AIOps-1-event-generate/)
	- [chapter1.1 单指标异常检测](../AIOps-1_1-kpi-detector/)
	- [chapter1.3 故障预测](../AIOps-1_2-fault-prediction/)
	- [chapter1.4 指标异常关联](../AIOps-1_4-kpi-correlation/)
	- [chapter1.5 日志聚类](../AIOps-1_5-log-analysis/)
		- [chapter1.5.1 使用logmine加强版做日志聚类](../AIOps-1_5_1-log-analysis_logmine/)
		- [chapter1.5.2 美团日志聚类](../AIOps-1_5_2-log-analysis_meituan/)
- [chapter2 故障定位](../AIOps-2-event-analysis/)
	- [chapter2.1 微服务系统的故障定位](../AIOps-2_1-topo-rca/)
		- [chapter2.1.1 CauseInfer1](../AIOps-2_1_1-topo-rca-causeinfer-notes1/)
		- [chapter2.1.2 CauseInfer2](../AIOps-2_1_2-topo-rca-causeinfer-notes2/)
		- [chapter2.1.3 AIOps挑战赛2020-获奖方案分享](../AIOps-2_1_3-topo-rca-aiops2020/)
		- [chapter2.1.4 AIOps挑战赛2021-demo方案](../AIOps-2_1_4-topo-rca-aiops2021/)
		- [chapter2.1.5 N-Softbei2020比赛](../AIOps-2_1_5-topo-rca-cnsoftbei2020/)
		- [chapter2.1.6 MicroCause](../AIOps-2_1_6-topo-rca-MicroCause)
	- [chapter2.2 多维下钻根因定位](../AIOps-2_2-multi-dimensional-rca/): 暂无
	- [chapter2.3 调用链根因分析](../AIOps-2_3-trace_rca/)
	- [chapter2.4 时间序列关联性分析](../AIOps-2_4-metric_event_correlation/)
- chapter3 故障恢复


> 本文是美团的提出的告警聚类的一种算法，其实也只是基于已有的paper做了一下应用。
> 
> 跟logmine的区别是，他可以定义多层的泛化结构，对于有多层关系的属性有很好的效果。



## **设计**

如[图2]所示，异常报警根因分析的设计大致分为四个部分：搜集告警数据、提取告警的关键特征、聚类、展示告警摘要。


![图2](img.png)


### **算法选择**

聚类算法采用论文《Clustering intrusion detection alarms to support root cause analysis》中的算法。

![img](./kjhjk.png)

我们可以将这几条报警抽象为：“全部服务器 网络调用 故障”，该泛化报警包含的范围较广；也可以抽象为：“server_room_a服务器 网络调用 产品信息获取失败”和“server_room_b服务器 RPC 获取产品类型信息失败”，此时包含的范围较小。当然也可以用其他层次的抽象来表达这个报警集群。

可以观察到，抽象层次越高，细节越少，但是它能包含的范围就越大；反之，抽象层次越低，则可能无用信息越多，包含的范围就越小。

这种抽象的层次关系可以用DAG表达，如图3所示：

![图3 泛化层次结构示例](img_1.png)

### 基本概念

- 属性（Attribute）：构成报警日志的某一类信息，如机器、环境、时间等，文中用$A_i$表示。
- 值域（Domain）：属性$A_i$的取值范围，文中用$Dom(A_i)$表示。
- 泛化层次结构（Generalization Hierarchy）：对于每个Ai都有一个对应的泛化层次结构，文中用$G_i$表示。
  ![图3 泛化层次结构示例](img_1.png)
- 属性的不相似度:令$x_1$、$x_2$为某个属性$A_i$的两个不同的值，那么$x_1$、$x_2$的不相似度为：在泛化层次结构$Gi$中，通过一个公共点父节点p连接x1、x2的最短路径长度。即
  d(x1, x2) := min{d(x1, p) + d(x2, p) | p ∈ Gi, x1 ⊴ p, x2 ⊴ p}。例如在图3的泛化层次结构中，d("Thrift", "Pigeon") = d("RPC", "Thrift") + d("RPC", "Pigeon") = 1 + 1 = 2。
- 告警的不相似度（Dissimilarity）: 对于两个报警$a_1, a_2$, 距离为：
	$$d\left(\mathbf{a}_{1}, \mathbf{a}_{2}\right):=\sum\limits_{i=1}^{n} d\left(\mathbf{a}_{1}\left[A_{i}\right], \mathbf{a}_{2}\left[A_{i}\right]\right)$$
  
	例如： a1 = ("server_room_b-biz_tag-offline02", "Thrift"),a2 = ("server_room_a-biz_tag-online01", "Pigeon"), 那么d(a1, a2) = d("server_room_b-biz_tag-offline02", "server_room_a-biz_tag-online01") + d(("Thrift", "Pigeon") = 2 + 2 + 1 + 1 = 6。

- 告警集合：C
- 告警摘要：g，$\forall a \in C, a \leq g$, 就是pattern

以报警集合{"dx-trip-package-api02 Thrift get deal list error.", "dx-trip-package-api01 Thrift get deal list error."}为例，“dx服务器 thrift调用 获取产品信息失败”是一个泛化表示，“服务器 网络调用 获取产品信息失败”也是一个泛化表示。对于某个聚类来说，我们希望获得既能涵盖它的集合又有最大信息量的摘要（pattern）。为了解决这个问题，定义以下两个指标：

$$\begin{aligned} \bar{d}(\mathbf{g}, \mathcal{C}) &:=1 /|\mathcal{C}| \times \sum_{\mathbf{a} \in \mathcal{C}} d(\mathbf{g}, \mathbf{a}) \\ H(\mathcal{C}) &:=\min \left\{\bar{d}(\mathbf{g}, \mathcal{C}) | \mathbf{g} \in \mathbf{X}_{i=1}^{n} \operatorname{Dom}\left(A_{i}\right), \forall \mathbf{a} \in \mathcal{C}: \mathbf{a} \leq \mathbf{g}\right\} \end{aligned}$$

公式2

H(C)值最小时对应的g，就是我们要找的最适合的泛化表示，我们称g为C的“覆盖”（Cover）。

基于以上的概念，将报警日志聚类问题定义为：

> 定义L为一个日志集合，$G_i(i = 1, 2, 3……n) $为属性Ai的泛化层次结构，目标是找到一个L的子集C，满足 |C| >= min_size，且H(C)值最小。

min_size是用来控制抽象程度的，min_size越大抽象的越厉害.
这是个NP问题，论文提出了如下的启发式算法

### **算法描述**

1. 假设所有的泛化层次结构$G_i$都是树，这样每个cluster都有一个唯一的、最顶层的泛化结果。
2. 将L定义为一个原始的告警日志集合，算法选择一个属性$A_i$，将L中所有报警的$A_i$值替换为$G_i$中$A_i$的父值，通过这一操作不断对报警进行泛化。
3. 持续步骤2的操作，直到找到一个覆盖报警数量大于min_size的pattern为止。
4. 输出步骤3中找到的报警。

算法伪代码如下所示：

```markdown
输入：报警日志集合L，min_size，每个属性的泛化层次结构G1,......,Gn
输出：所有符合条件的泛化报警
T := L;              // 将报警日志集合保存至表T
for all alarms a in T do
    a[count] := 1;   // "count"属性用于记录a当前覆盖的报警数量
while ∀a ∈ T : a[count] < min_size do {
    使用启发算法选择一个属性Ai;
    for all alarms a in T do
        a[Ai] := parent of a[Ai] in Gi;
        while identical alarms a, a' exist do
            Set a[count] := a[count] + a'[count];
            delete a' from T;
}
```

其中第7行的启发算法为：

```markdown
首先计算Ai对应的Fi
fi(v) := SELECT sum(count) FROM T WHERE Ai = v   // 统计在Ai属性上值为v的报警的数量
Fi := max{fi(v) | v ∈ Dom(Ai)}
选择Fi值最小的属性Ai
```

## 实现

### 提取报警特征

根据线上问题排查的经验，运维人员通常关注的指标包括时间、机器（机房、环境）、异常来源、报警日志文本提示、故障所在位置（代码行数、接口、类）、Case相关的特殊ID（订单号、产品编号、用户ID等等）等。

但是，Case相关的特殊ID不符合我们希望获得一个抽象描述的要求，因此不关注。

综上，最后选择的特征包括：机房、环境、异常来源、报警日志文本关键内容、故障所在位置（接口、类）共5个。

### 算法实现

(1) 提取关键特征：数据来源是日志中心已经格式化过的告警日志信息，包含：报警日志产生的时间、服务标记、在代码中的位置、日志内容等。

- 故障所在位置：优先查找是否有异常堆栈，如存在则查找第一个本地代码的位置；如果不存在，则取日志打印位置。
- 异常来源：获得故障所在位置后，优先使用此信息确定异常报警的来源（需要预先定义词典支持）；如不能获取，则在日志内容中根据关键字匹配（需要预先定义词典支持）。
- 报警日志文本关键内容：优先查找是否有异常堆栈，如存在，则查找最后一个异常（通常为真正的故障原因）；如不能获取，则在日志中查找是否存在“code=……,message=……” 这样形式的错误提示；如不能获取，则取日志内容的第一行内容（以换行符为界），并去除其中可能存在的Case相关的提示信息
- 提取“机房和环境”这两个指标比较简单，在此不做赘述。

(2) 聚类算法：以图4来表示。

	![图4 报警日志聚类流程图](img_2.png)

### 泛化层次结构

泛化层次结构，用于记录属性的泛化关系，是泛化时向上抽象的依据，需要预先定义。

下面是4个预先定义的泛化层次结构

![图5 机房泛化层次结构](img_9.png)

![图6 环境泛化层次结构g](img_10.png)

![图7 错误来源泛化层次结构](img_11.png)

![图8 日志文本摘要泛化层次结构](img_12.png)


“故障所在位置”此属性无需泛化层次结构，每次泛化时直接按照包路径向上层截断，直到系统包名。

## 实验

以下三个实验均使用C端API系统。

### 1. 单依赖故障

- 环境：线上
- 故障原因：产品中心线上单机故障
- 报警日志数量：939条

部分原始报警日志如图9所示，初次观察时，很难理出头绪。

![图9 单依赖故障报警日志节选](img_3.png)


经过聚类后的摘要如表1所示：

![img_4.png](img_4.png)

可以看到前三条摘要的Count远超其他摘要，并且它们指明了故障主要发生在产品中心的接口。


### 2. 无相关的多依赖同时故障

利用故障注入工具，在Staging环境模拟运营置顶服务和A/B测试服务同时产生故障的场景。

- 环境：Staging,使用线上录制流量和压测平台模拟线上正常流量环境
- 模拟故障原因：置顶与A/B测试接口大量超时
- 报警日志数量：527条

部分原始报警日志如图10所示：

![图10 无相关的多依赖同时故障报警日志节选](img_5.png)

经过聚类后的报警摘要如表2所示：

![表2](img_6.png)


从上表可以看到，前两条摘要符合本次试验的预期，定位到了故障发生的原因。

说明在多故障的情况下，算法也有较好的效果。

### 3. 中间件与相关依赖同时故障

利用故障注入工具，在Staging环境模拟产品中心服务和缓存服务同时产生超时故障的场景。

- 环境：Staging，使用线上录制流量和压测平台模拟线上正常流量环境
- 模拟故障原因：产品中心所有接口超时，所有缓存服务超时
- 报警日志数量：2165

部分原始告警日志如图11所示：

![图11 中间件与相关依赖同时故障报警日志节选](img_7.png)


经过聚类后的报警摘要如表3所示：

![img_8.png](img_8.png)

从上表可以看到，缓存（Squirrel和Cellar双缓存）超时最多，产品中心的超时相对较少，这是因为我们针对产品中心的部分接口做了兜底处理，当超时发生时后先查缓存，如果缓存查不到会穿透调用一个离线信息缓存系统，因此产品中心超时总体较少。

### 总结

- 优点：综合上述三个实验得出结果，算法对于报警日志的泛化是具有一定效果。
- 不足：有些摘要过于笼统；利用摘要进行进一步定位，也需要领域知识。


## 参考资料

1. [Cluster_analysis--wiki](https://en.wikipedia.org/wiki/Cluster_analysis)
2. [meituan-blog](https://tech.meituan.com/2019/02/28/root-clause-analysis.html)
3. [Clustering intrusion detection alarms to support root cause analysis--paper](https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.136.1949&rep=rep1&type=pdf))
