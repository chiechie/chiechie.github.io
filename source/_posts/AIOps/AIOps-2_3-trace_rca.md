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


# 总结

1. 在微服务的架构中，1个web service内部通常需要调用多个微服务。
2. 当外部对web service S1 发起请求时，请求id就是uuid。S1 会将这个外部请求拆成一系列微服务之间的子调用。
3. 这样微服务和它们之间的调用（call path）关系就构成一个调用拓扑。当其中某个节点--即某个微服务s1--发生故障了，所有调用他的微服务s2,s3,...都会发生故障。所以可以看到，故障是沿着调用的反方向传播的，或者说沿着被调用的方向传播的。
4. 对微服务架构进行故障定位，也就是需要，实时监控web service（S1）的健康状况，并且在S1异常的时候给出根因节点，即是哪个微服务故障引起的，从而去执行修复的操作。
5. 那么为了监控调用链路的，希望系统能回答两个问题
	1. 是当前调用是否异常？，
	2. 如果异常，是哪个节点（微服务）引起的？
5. 当web service S1时，通常有两类表现，分别是调用链结构异常 和 调用耗时异常

	- 调用结构异常：即跟正常调用链关系相比，边变多了或者变少了。举个列子，微服务a本应该调用微服务b，但是a服务挂了，那么本来应该存在的调用链条就消失了。再举个列子，正常情况下本不应该存在调用关系，但是由于故障导致产生了不正常的调用关系。
	- 调用耗时异常：即调用耗时比正常的调用耗时要少/多。举个例子，正常情况下a调用b，b响应a只需要1min，但是如果b所在的机器内存满了, 就需要10min完成响应了。
6. TraceAnomaly对调用链系统做异常检测的思路是这样：
   1. 生成样本：对每一个web service S1，搜集跟S1相关的所有trace数据。S1被调用一次就生成一个trace, 也就是一个样本，uuid可以认为是样本id了。1个样本包含的信息有：所有子调用span_id, 每次子调用的耗时，以及路径（起点（调用方）-->终点（被调用方）） 
   2. trace向量化：每个trace的每一次子调用call_path 都会统一存放到 call path list 中去，也就是说 call path list收集了 所有trace 的所有的子调用路径, 就好像一个字典。 然后将span参考call path list去做one-hot编码，然后构建一个等长的耗时向量，把两个向量做点乘。注意，为了区分耗时很短的子调用和实际中缺失的自调用，将one-hot encoder向量中，为0的元素设置为-1（为0表示，实际没有命中该位置对应的call path）。 
   3. 降维得到STV：如果某个web service对应的所有trace的call path的并集为32，那么样本的维度就是32, 一个样本对应的特征向量就叫STV（Service Trace Vector）。得到了这个STV后要做两个事情：检测 和 定位
   4. 异常检测： 构造一个生成模型（vae），输入STV，输出是该STV的logits，logits越小说明越异常
   5. 根因定位：通过异常检测知道了该trace异常，那么到底是哪个节点导致的呢？
	  1. 找出同构trace：在历史数据中，找出当前有问题的trace通过的其他traces（同构调用链，也叫HSTV）
	  2. 找出异常的子调用：跟同构trace对比所有子调用耗时（3-sigma） 
	  3. 找call path最长的子调用：将所有异常子调用，映射成call path，并且按照调用链路长度从大到小排序，最长的哪个call path就是根因e。

7. TraceAnomaly是这么处理冷启动问题的：如果在实时监控的时候，产生了新的正常的call path，而call path list这个字典是滞后，不管新的call path是不是符合预期，正常流程都会认为是异常。这个时候需要让运维人员进行人工审核，看call path是否符合预期，符合预期就放入白名单，暂时不告警，并且不会判断其耗时。




# 附录

## 调用链的数据格式

- 做调用链路分析（TraceAnomaly）用到了耗时，以及调用链的结构信息。
- 2021AIOps比赛的调用链的数据：每个span_id表示一个trace中的一次子调用，耗时表示当前服务从接受call，到开始response的时间间隔。这里的表格信息量很少，它不知道每次子调用，具体是那个应用在调用，也不知道此次子调用之前，该trace已经走过了哪些调用路径了。

## 代码

https://github.com/chiechie/TraceAnomaly

## 论文内容：

### 基本概念

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

	
### TraceAnomaly总览

1. TraceAnomaly整体结构
	![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FAEYpjsaibJ.png?alt=media&token=361e3b43-eeee-49af-90b7-fed65874868b)

### STV构造方法

1. 在线检测时，每一个新的trace 被编码成一个向量（STV）:
   - 如果trace中包含有未见过的call path就认为是异常，当然这么简单粗暴的方式会带来误警，所以后面提出了白名单的方法（让运维人员去判断，当然只用判断1次）。
   - 如果trace中没有unseen call path,就丢到神经网络中计算vector 的likelihood
2. 从一个调用链数据（trace）中提取调用路径（call paths）和 响应时长（response times） 
	![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FaOsHfGkpQQ.png?alt=media&token=6b899aa5-ad63-4f57-899d-d97eb7f04e33)
3. 计算微服务b的响应时间
	- 同一行的右边减去左边表示，source 到arget的通信时间。
	- rt（respond time） = 处理完事务并开始响应的时间点（第11行红色的时间戳） - 接受到指令的时间（第2行红色的时间戳）= 处理事务消耗的时间
	- ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2Fkqz7HgJgWs.png?alt=media&token=3be5adbe-1632-478e-b77e-39cae4d07068)
4. 构造STV（Service Trace Vector）
   
	![Fig. 6: Trace t1 和 t2 结构一样, 但t1正常，t2 异常](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FL-kGMqUym3.png?alt=media&token=a388505b-f239-4b57-b5f2-770253d44ead)
	- t1 ：业务逻辑不要 b调用e。
	- t2 ：业务逻辑要b调用e，但是因为e故障了导致调用失败。
	- 两种情况都会导致看不到b->e的call path，但是t1是正常，而t2是异常。那么应该怎么做 才能找到 t2 ，而不误伤t1呢？看耗时
5. STV的构造过程
	![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FIcq2XYIGEe.png?alt=media&token=ac5e5876-cc3f-4fd2-94d4-fec58ebf3a1e)
### 根因定位算法

![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FTn7YZeNZvO.png?alt=media&token=12611153-f760-45de-843f-d2db11015f81)
	- step1 先使用预测模型判断是否当前 trace是否异常
	- step2 然后判断异常的维度是哪个？跟同构traces（valid dimension 一摸一样的traces）对比 每个维度的耗时。
	- step3 链路最长的异常维度就是根因
	- ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FZodnzghX6P.png?alt=media&token=a6870609-66c6-472d-998c-4e6efe4356cd)


### 对STV做异常检测

1. 通过生成模型学习数据分布，VAE模型结构
  ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2F5CUrxFwPhA.png?alt=media&token=84b40833-873d-482b-979f-181f4fc3ad45)
2. 训练VAE
3. 使用VAE异常检测
	- $$\log p_{\theta}(\mathbf{x}) \approx \log \frac{1}{L_{z}} \sum_{l=1}^{L_{z}}\left[\frac{p_{\theta}\left(\mathbf{x} \mid \mathbf{z}_{(l)}^{(K)}\right) p_{\theta}\left(\mathbf{z}_{(l)}^{(K)}\right)}{q_{\phi}^{(K)}\left(\mathbf{z}_{(l)}^{(K)} \mid \mathbf{x}\right)}\right]$$
	- 某个trace为异常 等价于 ，该trace 的对数概率明显小于  正常trace（要求同构吗？
4. 怎么界定正常呢？用kde来拟合对数概率的分布

### 实施和评估

- A. Deployment at online services
	![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FYT-1AJDNXX.png?alt=media&token=a4984091-89d3-424f-b97f-bfdbcfbf8409)
- B. Dealing with Unseen Call Paths
	![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FSuilsPv1oH.png?alt=media&token=6262ff2c-60ab-4e31-a60a-3fddc035ff7d)
  
- B. Anomaly Detection Baseline Approaches
  ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FUrcO_5Aovy.png?alt=media&token=92e2bcbf-7711-401b-927e-636b9ca2d296)
- E. Localizing Root Cause
	![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2F2-eRHdEbPI.png?alt=media&token=708e7973-4439-424a-b037-2dac452daf98)
  ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FFJ0FxrFnTb.png?alt=media&token=12d55bec-55c0-4df7-a6ba-c074c5d829db)
- demo复现
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FA_n0kckFDJ.png?alt=media&token=f438c18d-5396-4914-b321-903554ce6316)

    - 微服务e的响应时长 是故障根因，
	![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FGZoUsnaFDn.png?alt=media&token=06311178-0f22-4f3d-90ef-a67405dd20e1)


## 参考资料
1. [清华微众AIOps新作:基于深度学习的调用轨迹异常检测算法 -微信公众号](https://mp.weixin.qq.com/s/sqYIb6i9z6xF5nDr8fuVsA)
2. [chapter2.1.4 AIOps挑战赛2021-demo方案](https://chiechie.github.io/2021/03/09/AI/AIOps/AIOps-2_1_4-topo-rca-aiops2021/)
