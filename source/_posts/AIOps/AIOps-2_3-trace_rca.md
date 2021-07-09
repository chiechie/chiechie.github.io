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
5. 当web service S1时，通常有两类表现，分别是调用链结构异常 和 调用耗时异常

	- 调用结构异常：即跟正常调用链关系相比，边变多了或者变少了。举个列子，微服务a本应该调用微服务b，但是a服务挂了，那么本来应该存在的调用链条就消失了。再举个列子，正常情况下本不应该存在调用关系，但是由于故障导致产生了不正常的调用关系。
	- 调用耗时异常：即调用耗时比正常的调用耗时要少/多。举个例子，正常情况下a调用b，b响应a只需要1min，但是如果b所在的机器内存满了, 就需要10min完成响应了。
6. 总结下调用链路故障定位要解决的问题：
	1. 当前微服务间的调用是否正常？有可能某些微服务之间的调用已经异常了，但是web service暂时还能对外提供正常服务。
	2. 如果异常，是哪个节点（微服务）引起的？
6. TraceAnomaly对调用链系统做异常检测的思路是这样：
   1. 生成样本：对每一个web service S1，搜集跟S1相关的所有trace数据。S1被调用一次就生成一个trace, 也就是一个样本，uuid可以认为是样本id了。1个样本包含的信息有：所有子调用span_id, 每次子调用的耗时，以及路径（起点（调用方）-->终点（被调用方）） 
   2. trace向量化：每个trace的每一次子调用call_path 都会统一存放到 call path list 中去，也就是说 call path list收集了 所有trace 的所有的子调用路径, 就好像一个字典。 然后将span参考call path list去做one-hot编码，然后构建一个等长的耗时向量，把两个向量做点乘。注意，为了区分耗时很短的子调用和实际中缺失的自调用，将one-hot encoder向量中，为0的元素设置为-1（为0表示，实际没有命中该位置对应的call path）。 
   3. 降维得到STV：如果某个web service对应的所有trace的call path的并集为32，那么样本的维度就是32, 一个样本对应的特征向量就叫STV（Service Trace Vector）。得到了这个STV后要做两个事情：检测 和 定位
   4. 异常检测： 构造一个生成模型（vae），输入STV，输出是该STV的logits，logits越小说明越异常
   5. 根因定位：通过异常检测知道了该trace异常，那么到底是哪个节点导致的呢？
	  1. 找出同构trace：在历史数据中，找出当前有问题的trace通过的其他traces（同构调用链，也叫HSTV）
	  2. 找出异常的子调用：跟同构trace对比所有子调用耗时（3-sigma） 
	  3. 找call path最长的子调用：将所有异常子调用，映射成call path，并且按照调用链路长度从大到小排序，最长的哪个call path就是根因e。

7. TraceAnomaly是这么处理冷启动问题的：如果在实时监控的时候，产生了新的正常的call path，而call path list没有这个call pah，那么正常流程会认为该call path对应的trace是异常。这时需要让运维人员进行人工审核，看call path是否符合预期，符合预期就将该call path放入白名单，暂时不告警，并且不会判断其耗时。


# 附录
	
## TraceAnomaly流程

1. TraceAnomaly整体结构
	![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FAEYpjsaibJ.png?alt=media&token=361e3b43-eeee-49af-90b7-fed65874868b)
2. TraceAnomaly在线检测：
   - 首先提取新trace中的所有子调用的call path
   - 如果trace中包含有未见过的call path就认为是异常，当然这么简单粗暴的方式会带来误警，所以后面提出了白名单的方法（让运维人员去判断，当然只用判断1次）。
   - 如果trace中没有unseen call path, 将该trace编码成一个STV向量，就丢到神经网络vae中计算vector的likelihood， 进行异常检测
   - 如果当前trace异常，触发在线的根因分析

		- step1 首先跟跟同构traces相比，找出异常的call paths？
		- step2 在异常的call paths中找出调用链路最长的path
   	> the traceAnomaly finds the longest common path between trace t's longest call path and call paths in the call path list
	    ![根因分析](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FTn7YZeNZvO.png?alt=media&token=12611153-f760-45de-843f-d2db11015f81)
	 
## 调用链路数据预处理

目的: 从一个调用链数据（trace）中提取微服务的调用路径（call paths）和 每个微服务的响应时长（response times） 

1. 调用路径（call path）: 从本次trace启动开始，直到微服务s被调用，经过的子调用组成的列表（按调用时间排序），将这个序列记为call path。
2. 某个微服务s的响应时间: $$ rt(response time) = srt(sending time of response message) - rct(recieve time of response message) $$
![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2Fi4eYxs1D2B.png?alt=media&token=fdaf077a-cddc-4e9a-bd0f-4f2c291b3b4d)
   
3. 举例介绍怎么计算微服务b的响应时间：从原始的调用数据(trace)提取微服务s的耗时，以下图为例，微服务b的响应时间（respond time） = 处理完事务并开始响应的时间点（第11行红色的时间戳） - 接受到指令的时间（第2行红色的时间戳）= 处理事务消耗的时间

![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FzI6rcYmiD2.png?alt=media&token=449f532b-a126-40b0-a6dd-f07067847e3a)

5. 构造STV：使用所有(s, call path)对应的耗时组装成向量，即STV：
	![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FIcq2XYIGEe.png?alt=media&token=ac5e5876-cc3f-4fd2-94d4-fec58ebf3a1e)
6. 调用耗时异常的例子：
	![Fig. 6: Trace t1 和 t2 结构一样, 但t1正常，t2 异常](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FL-kGMqUym3.png?alt=media&token=a388505b-f239-4b57-b5f2-770253d44ead)
	- t1 ：业务逻辑是b不调用e。
	- t2 ：业务逻辑是b调用e，但是因为e故障了导致调用失败。
	- 两种情况都看不到b->e的call path，但是t1是正常，而t2是异常。那么应该怎么做 才能找到 t2 ，而不误伤t1呢？看耗时


## 对STV做异常检测

1. VAE的输入
	![样本概念](./image_1.png)
2. 通过生成模型学习数据分布，VAE模型结构
  ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2F5CUrxFwPhA.png?alt=media&token=84b40833-873d-482b-979f-181f4fc3ad45)
2. 训练VAE
3. 使用VAE异常检测
	- $$\log p_{\theta}(\mathbf{x}) \approx \log \frac{1}{L_{z}} \sum_{l=1}^{L_{z}}\left[\frac{p_{\theta}\left(\mathbf{x} \mid \mathbf{z}_{(l)}^{(K)}\right) p_{\theta}\left(\mathbf{z}_{(l)}^{(K)}\right)}{q_{\phi}^{(K)}\left(\mathbf{z}_{(l)}^{(K)} \mid \mathbf{x}\right)}\right]$$
	- 某个trace为异常 等价于 ，该trace 的对数概率明显小于  正常trace（要求同构吗？
4. 怎么界定正常呢？用kde来拟合对数概率的分布, 然后使用p值来判断。


## 参考资料
1. [清华微众AIOps新作:基于深度学习的调用轨迹异常检测算法 -微信公众号](https://mp.weixin.qq.com/s/sqYIb6i9z6xF5nDr8fuVsA)
2. [chapter2.1.4 AIOps挑战赛2021-demo方案](https://chiechie.github.io/2021/03/09/AI/AIOps/AIOps-2_1_4-topo-rca-aiops2021/)
3. https://github.com/chiechie/TraceAnomaly
4. https://docs.qq.com/sheet/DVERzcVpWa2xtckJT?tab=BB08J2

