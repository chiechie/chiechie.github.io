---
title: chapter2.3 调用链根因分析
author: chiechie
mathjax: true
date: 2021-05-26 16:49:19
tags:
- 调用链
- AIOps
categories: 
- AIOps

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
	- [chapter2.2 多维下钻根因定位](../AIOps-2_2-multi-dimensional-rca/)
	- [chapter2.3 调用链根因分析](../AIOps-2_3-trace_rca/)
	- [chapter2.4 时间序列关联性分析](../AIOps-2_4-metric_event_correlation/)
- chapter3 故障恢复



## 调用链的数据格式

- 做调用链路分析（TraceAnomaly）用到了耗时，以及调用链的结构信息。
- 2021AIOps比赛的调用链的数据：每个span_id表示一个trace中的一次子调用，耗时表示当前服务从接受call，到开始response的时间间隔。这里的表格信息量很少，它不知道每次子调用，具体是那个应用在调用，也不知道此次子调用之前，该trace已经走过了哪些调用路径了。

## 代码

放这里:


![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FKYZHUFjpAr.png?alt=media&token=1d1635a5-9524-4918-986a-c9eb54a1a558)

Service Trace Vector

## 总结


该论文想解决的问题：监控一个web service S1的健康状况，并且在异常的时候给出根因--是哪个节点故障引起的？

具体来说，这个web service S1由多个微服务组成，当外部应用向S1发起一个请求时（请求id就是uuid），S1 会将这个外部请求拆成一系列微服务之间的子调用。而如果其中微服务s1发生了故障，所有调用他的微服务s2,s3,...都会发生故障。

发生故障的表现有两种：调用链结构异常 和 调用耗时异常。

- 调用结构异常 比如某个微服务挂了，导致调用路径都没有记录，或者本不应该调用某个微服务但是产生了调用数据。
- 调用耗时异常比如，本来a调用b，b通常的响应只需要1min，但是b所在的机器内存满了，导致s5响应完成用了10min。

那么为了监控调用链路的，希望系统能回答两个问题

1. 是当前调用是否异常？，
2. 如果异常，是哪个节点（微服务）引起的？


文章的解决方案是这样的：

针对1个web服务S1，将它的所有的trace数据搜集起来。按照uuid1个1个trace进行分析，对1个trace，提取出所有的节点和子调用路径以及耗时。此外，每个trace的每一次子调用call_path 都会统一存放到 call path list 中去，也就是说 call path list收集了 所有trace 的所有 的子调用路径。然后将这个自调用路径 作为维度，记录每一次trace 在 call path 取便 call path list中的每一个值时 对应的耗时，如果trace 中没有访问到 call path list中的某个call path，就记为-1。这样如果有100个trace，就可以得到100个样本，如果100个trace的所有call path的并集为32，那么样本的维度就是32。一个样本对应的特征向量就叫STV（Service Trace Vector）。得到了这个STV后要做两个事情：检测 和 定位
  
1. 检测： 构造一个生成模型（vae），输入是原始的STV，输出是STV的对数概率。对数概率越小说明越异常
2. 根因定位：通过1知道了trace整体是异常的，那么到底是哪个维度导致的呢？需要进一步 找到当前trace 的 同伴们（同构调用链，HSTV），  把每一个维度的耗时 跟 同伴们在 该维度的平均耗时进行对比（3-sigma），判断自己是不是异常的。进一步，将自己的所有异常的维度汇总，按照call path的 长度从大到小排序，call path长度最大的 维度 或者 它的下游call path 就是根因。

冷启动问题：上面的方法对没在训练数据中出现过的call path是无效的，这个时候需要让运维人员进行人工审核call path是否符合预期，符合预期就放入白名单，。在下一轮训练启动之前，统一认为 是正常的并且不会判断其耗时。

这样微服务和它们之间的调用（call path）关系就构成一个调用拓扑。当其中1个节点

## 基本概念

- ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2Fu9Iqc3l8Tq.png?alt=media&token=b8798779-4d40-4c62-a9f6-0322561041b5)
- ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FANIRoOIQ2o.png?alt=media&token=da808004-780a-460e-98e0-eef80cfb7fe8)
- ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FhVSU37d3Pd.png?alt=media&token=bae28c0c-f5dc-4114-9313-c987ea0cf1d2)
- call path ： A call path of a microservice s in a trace, denoted as (s, call
path), is the sequence of call messages (sorted by sending time)
before s is called.
        - 在一次调用中，从start 到 某个微服务s 被调用，所有被调用的s'.(按时间顺序)
    - srt：the sending time of the corresponding response message，某次调用的 发送时间
    - rt：response time，某次调用的respnd时间
    - $$ r t=s r t-rc t $$，某次调用响应时长
    - STV（Service Trace Vector）：微服务调用向量
    - 调用路径提取：
        - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2Fi4eYxs1D2B.png?alt=media&token=fdaf077a-cddc-4e9a-bd0f-4f2c291b3b4d)
- 
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FzI6rcYmiD2.png?alt=media&token=449f532b-a126-40b0-a6dd-f07067847e3a)
- [论文地址](https://netman.aiops.org/wp-content/uploads/2020/09/%E5%88%98%E5%B9%B3issre.pdf)
- https://mp.weixin.qq.com/s/sqYIb6i9z6xF5nDr8fuVsA
- https://netman.aiops.org/wp-content/uploads/2020/09/%E5%88%98%E5%B9%B3issre.pdf


## 论文内容：

### II TraceAnomaly总览，STV构造方法，定位算法

- A. TraceAnomaly整体结构
	- ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FAEYpjsaibJ.png?alt=media&token=361e3b43-eeee-49af-90b7-fed65874868b)
	- 在线检测时，每一个新的trace 被编码成一个向量（STV），
		- 如果trace中包含有未见过的call path就认为是异常，当然这么简单粗暴的方式会带来误警，所以后面提出了白名单的方法（让运维人员去判断，当然只用判断1次）。
		- 如果trace中没有unseen call path,就丢到神经网络中计算vector 的likelihood
- B. 从一个调用链数据（trace）中提取调用路径（call paths）和 响应时长（response times） 
	- ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FaOsHfGkpQQ.png?alt=media&token=6b899aa5-ad63-4f57-899d-d97eb7f04e33)
		- 计算微服务b的响应时间
			- 同一行的右边减去左边表示，source 到arget的通信时间。
			- rt（respond time） = 处理完事务并开始响应的时间点（第11行红色的时间戳） - 接受到指令的时间（第2行红色的时间戳）= 处理事务消耗的时间
	- ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2Fkqz7HgJgWs.png?alt=media&token=3be5adbe-1632-478e-b77e-39cae4d07068)
- C. 构造STV（Service Trace Vector）
	- Fig. 6: Trace t1 和 t2 结构一样, 但t1正常，t2 异常
		- ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FL-kGMqUym3.png?alt=media&token=a388505b-f239-4b57-b5f2-770253d44ead)
		- t1 ：业务逻辑不要 b调用e。
		- t2 ：业务逻辑要b调用e，但是因为e故障了导致调用失败。
		- 两种情况都会导致看不到b->e的call path，但是t1是正常，而t2是异常。那么应该怎么做 才能找到 t2 ，而不误伤t1呢？看耗时
- D. 定位根因
	- ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FTn7YZeNZvO.png?alt=media&token=12611153-f760-45de-843f-d2db11015f81)
	- step1 先使用预测模型判断是否当前 trace是否异常
	- step2 然后判断异常的维度是哪个？跟同构traces（valid dimension 一摸一样的traces）对比 每个维度的耗时。
	- step3 链路最长的异常维度就是根因
	- ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FZodnzghX6P.png?alt=media&token=a6870609-66c6-472d-998c-4e6efe4356cd)


### III. 异常检测算法

- A. 学习数据分布
- B. 模型结构
  ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2F5CUrxFwPhA.png?alt=media&token=84b40833-873d-482b-979f-181f4fc3ad45)
- C. 训练
- D. 异常检测
	- $$\log p_{\theta}(\mathbf{x}) \approx \log \frac{1}{L_{z}} \sum_{l=1}^{L_{z}}\left[\frac{p_{\theta}\left(\mathbf{x} \mid \mathbf{z}_{(l)}^{(K)}\right) p_{\theta}\left(\mathbf{z}_{(l)}^{(K)}\right)}{q_{\phi}^{(K)}\left(\mathbf{z}_{(l)}^{(K)} \mid \mathbf{x}\right)}\right]$$
	- 某个trace为异常 等价于 ，该trace 的对数概率明显小于  正常trace（要求同构吗？）。怎么界定正常呢？用kde来拟合对数概率的分布

### IV. 实施和部署

- A. Deployment at online services
	![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FYT-1AJDNXX.png?alt=media&token=a4984091-89d3-424f-b97f-bfdbcfbf8409)
- B. Dealing with Unseen Call Paths
	![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FSuilsPv1oH.png?alt=media&token=6262ff2c-60ab-4e31-a60a-3fddc035ff7d)

### V. 评估

- B. Anomaly Detection Baseline Approaches
  ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FUrcO_5Aovy.png?alt=media&token=92e2bcbf-7711-401b-927e-636b9ca2d296)
- E. Localizing Root Cause
	![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2F2-eRHdEbPI.png?alt=media&token=708e7973-4439-424a-b037-2dac452daf98)
  ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FFJ0FxrFnTb.png?alt=media&token=12d55bec-55c0-4df7-a6ba-c074c5d829db)
- demo复现
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FA_n0kckFDJ.png?alt=media&token=f438c18d-5396-4914-b321-903554ce6316)
- STV的构造过程
	![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FIcq2XYIGEe.png?alt=media&token=ac5e5876-cc3f-4fd2-94d4-fec58ebf3a1e)
    - 微服务e的响应时长 是故障根因，
	![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FGZoUsnaFDn.png?alt=media&token=06311178-0f22-4f3d-90ef-a67405dd20e1)


## 参考资料

1. [清华微众AIOps新作:基于深度学习的调用轨迹异常检测算法 -微信公众号](https://mp.weixin.qq.com/s/sqYIb6i9z6xF5nDr8fuVsA)
2. [chapter2.1.4 AIOps挑战赛2021-demo方案](https://chiechie.github.io/2021/03/09/AI/AIOps/AIOps-2_1_4-topo-rca-aiops2021/)
3. [Unsupervised Detection of Microservice Trace Anomalies through Service-Level Deep Bayesian Networks paper](https://netman.aiops.org/wp-content/uploads/2020/09/%E5%88%98%E5%B9%B3issre.pdf)
4. [traceanomaly-github]( https://github.com/chiechie/TraceAnomaly)
