---
title: chapter1.1 使用GAN做异常检测
author: chiechie
mathjax: true
date: 2021-05-21 16:37:13
tags:
- AIOps
- 异常检测
- 异常分类
- GAN
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
	- [chapter2.2 多维下钻根因定位](../AIOps-2_2-multi-dimensional-rca/): 暂无
	- [chapter2.3 调用链根因分析](../AIOps-2_3-trace_rca/)
	- [chapter2.4 时间序列关联性分析](../AIOps-2_4-metric_event_correlation/)
- chapter3 故障恢复

> 使用gnn做异常检测

# 不仅仅生成图片，用GAN做无监督的异常检测

代码：https://github.com/chiechie/wgan-gp-anomaly

GAN被LeCun赞为继CNN之后最为重要的一个工作，其原因在于让各位惊呼“这才有点人工智能的样子”。相比于CNN或者RNN而言，GAN是一种完全不一样的思路。CNN或者RNN，其本质都是一种有监督的学习方式，相比于传统的方式而言，得益于网络强大的表达能力和自动学习特征的end-to-end的学习能力，CNN和RNN在很多任务上实现了巨大提升从而引领了这一次的人工智能的兴起。但是有监督学习的缺陷在于需要大量的数据进行学习，所以很多人工智能公司都是花了大量的资金和人力来收集和标注数据。记得在学校听“驭势科技”的黄波博士的一次技术分享，他就谈到数据的问题，笑称其实现在最赚钱的不是做人工智能技术的公司，而是那些做数据标注的公司。如果我没有记错的话，似乎买一张给自动驾驶标注的**精细的语义分割**的标签图需要1块人民币左右（敲黑板！商机啊！同学们）。

GAN是一种对抗学习网络，通过生成器G和判别器D的对抗学习来学习训练集的数据分布从而学会生成图片（[Goodfellow的paper](https://link.zhihu.com/?target=https%3A//arxiv.org/pdf/1406.2661.pdf)）。GAN一出来就被大家广泛的讨论，并且大量的人开始研究GAN，2017年以来arxiv上GAN的paper数量如同火箭一般的速度上升。原因就是GAN给做unsupervised learning提供一种很好的思路，最直接的应用在于我们可以用GAN学习真实的数据分布，从而生成图片可以给各种任务做**数据增强**。然后GAN也被应用在其他领域取得了非常不错的效果，有名的比如Twitter的那篇做超分辨的[SRGAN](https://link.zhihu.com/?target=https%3A//arxiv.org/abs/1609.04802)，最近最火的GAN应该是Berkeley的[Pix2Pix GAN](https://link.zhihu.com/?target=https%3A//arxiv.org/abs/1611.07004)和[Cycle GAN](https://link.zhihu.com/?target=https%3A//arxiv.org/abs/1703.10593)。

**Pix2Pix GAN和普通的GAN的区别是实现图片域的转换（Image Translation）**， 比如从手绘画到实物图的转换，而不是从一个噪声生成图片，效果非常之好。但是**Pix2Pix GAN要求需要pair的数据训练**，即两个图片域之间要有对应好的训练数据。Cycle GAN和Pix2Pix GAN一脉相承，加入了cycle consistency loss从而不需要像Pix2Pix GAN一样需要不同图片域里面pair的数据才能训练GAN，实现了unpair的训练就可以完成不同图片域之间的转换。由于开源代码写的非常好（特别是[Pytorch版的代码](https://link.zhihu.com/?target=https%3A//github.com/junyanz/pytorch-CycleGAN-and-pix2pix)，简直是Pytoch的良心模板，强推），实验效果也好，并且Image Translation本来就有很多的应用场景（除了刚提到的手绘图还有什么场景？），估计今年的CVPR里面应该会有很多魔改Cycle GAN和Pix2Pix GAN来做各种应用的paper出现。

## 把GAN应用于做异常检测

这篇文章要讲的不是用GAN来做图片的生成，而是一个非常有意思的应用方向-把GAN应用于做异常检测。首先解释一下什么是异常检测，顾名思义，异常检测就是**检测出异常情况**并且**定位出异常位置**。异常检测有着非常广泛的应用，例如在监控中，人行道中行人正常行走是正常情况，但是一旦出现车、自行车甚至滑板等等就是异常情况；或者在工厂生产产品中，正常外形的产品是合格的，但是也有出现一些瑕疵的产品。一般异常检测任务意味着**异常的复杂性和异常数据的少量性**：

- 1、复杂性：异常检测不同于分类任务，分类任务是有确定的类别个数，所有的结果都在确定的分类标签中，但是对于异常检测而言，只要是和正常情况有出入就是异常情况，所以异常情况非常多而且事先无法预知，所以如果用分类的方式来做异常检测有很大的局限性。
- 2、异常数据的少量性：异常情况往往不常出现，所以导致异常情况的数据不是很方便收集，但是正常情况的数据通常是很多的。

有很多正常的数据，但是没有很多异常的数据，那我们可不可以通过一个model来学习正常数据的分布，然后需要检测的异常图通过前面学习到的model找到它应该的正常图的样子，这样一对比不是可以找到异常吗？这样的思路简直完美契合GAN的思想！这篇文章里面我将会介绍一篇相关的paper：Unsupervised Anomaly Detection with Generative Adversarial Networks to Guide Marker Discovery， arxiv传送门：[https://arxiv.org/abs/1703.05921](https://link.zhihu.com/?target=https%3A//arxiv.org/abs/1703.05921)。

这篇paper是把GAN用在医学图像里面做异常检测，我们可以先看下整体的框架图：

![img](https://pic4.zhimg.com/80/v2-3afb9ef54ee08b8983abcec88975ff17_hd.jpg)Fig.1 图中使用的数据是眼部的SD-OCT图像，这种图像信噪比比较高，然后眼部结构相比于其他部位又比较简单一点。眼部OCT图像需要经过一些预处理：视网膜区域的提取，展平，还有给GAN输入的图像patch的提取和normalization。

这个框架图基本已经解释了这篇paper的方法，其实就是用正常的图片去训练GAN，然后通过GAN生成与异常图对应的正常图来对比找到异常。

![img](/Users/stellazhao/research_space/EasyMLBOOK/_image/v2-592e8ddc687eda45212d8ddf70853aeb_hd.png)Fig.2 (a)是网络结构图，(b)是指正常和异常数据的分布

Fig.2 是paper中使用的GAN的结构图，

其实在这篇paper里面使用的GAN就是普通的DCGAN，从一个噪声向量Z学习生成一张图片。我们可以看到正常的眼部OCT图的纹理是一种比较正常的过渡，但是异常的OCT图明显纹理产生了变化。DCGAN只用正常的OCT图像训练，这样DCGAN就只能从噪声生成正常纹理的OCT图像。当输入一个异常图时，通过比较DCGAN的生成图和异常图的差异去更新输入的噪声Z，从而生成一个与异常图尽可能相似的正常图。通过这样的方式，可以认为重建出了异常区域的理想的正常情况，这样两张图一对比不仅仅可以认定异常情况，同时还可以找到异常区域。

这样paper的重点是如何更新GAN的输入噪声Z，最直接的想法就是把在正常图中训练得到的生成器G的参数固定，然后通过计算生成图和输入的L1 loss或者L2 loss来更新输入的噪声Z。这篇paper中同样也是使用了L1 loss，在paper中作者命名为Residual loss，但是本质就是算pixel-wise的L1 loss：

![img](/Users/stellazhao/research_space/EasyMLBOOK/_image/v2-77779aba2e1a63a5ef312c0de9c38e2e_hd.png)

Fig.3 更新噪声Z的Residual loss

在这篇paper中作者还加了一个loss去迭代噪声Z，An improved discrimination loss based on feature matching：

![img](https://pic1.zhimg.com/80/v2-13c9d37e5f19a28390fc17de03097cf4_hd.jpg)Fig.4 更新噪声Z的Improved discrimination loss

加入这个loss目的是希望同时利用到训练好的判别器D，取判别器D中一个layer的输出，对比生成图和输入图之间在这层layer上feature map的差异，从而更新噪声Z。这样的目的在于加入了一个更high level的对比，希望生成图和输入尽量靠近。最后总体的loss是这两个loss的加权和：

![img](https://pic1.zhimg.com/80/v2-8b08b59f3a23aa990b2fb63bf65b3acc_hd.jpg)Fig.5 Final loss fuction

下面是作者贴出来的一些实验结果图，可以看到对于异常区域都可以比较明显的找到。

![img](/Users/stellazhao/research_space/EasyMLBOOK/_image/v2-9dbf3f09ea6ba47de03569ad1ad0bd6e_hd.png)Fig.6 

第一行：真实的输入图；第二行：迭代噪声Z之后的生成器G的生成图；第三行：计算生成与输入直接差异找到的异常区域；第四行：异常区域的heatmap图。图中的红色和黄色标注分别表示通过Residual loss和Improved discriminator loss认定的异常图片。

这篇paper提出了一个GAN很有意思的应用方向，对于异常情况，通过这样使用GAN就可以不需要任何异常数据而仅通过正常数据的训练就可以找到异常，相当于是一种无监督的异常检测方法。

但是这篇paper并没有开源代码，同时数据也没有给下载的link，我自己用Pytorch重现了这篇paper，github传送门：[oyxhust/wgan-gp-anomaly](https://link.zhihu.com/?target=https%3A//github.com/oyxhust/wgan-gp-anomaly)（来都来了，点个star再走吧）。和原始的paper不同的是我用了wgan-gp实现，而没有用DCGAN，wgan-gp应该是目前gan训练最稳定的一种方式之一。并且我也没有下载到paper中的眼部OCT数据集，目前是在MNIST在做的测试，我选择数字“0”的图像当成正常图像，然后其他数字图就是异常图。WGAN-gp只用数字“0”的图像进行训练，在测试中我用所有数字的图来测试model。

下面是我复现的效果图：

![img](https://pic1.zhimg.com/80/v2-c66190ea550aee83736e30e8e0ff02d0_hd.jpg)

可以看到GAN只能生成数字“0”（相当于正常情况），对于“0”的测试图，GAN可以生成几乎一样的输出。对于其他数字（相当于异常情况），GAN只能生成形状非常类似的“0”的图像。在我自己的实验中，对于“0”的测试图，L1 loss可以降低0.02左右，但是其他数字的测试图最低也是大概0.05左右，所以还是可以很明显地检测出异常。

我觉得GAN用来做异常检测是一个非常有意思的方向，但是目前还有很多问题，这篇paper里面的做法我认为只能检测出一些比较大的异常。对于比较小的异常，因为GAN生成图并不能做到细节非常明显，所以很难检测。




![image-20191113125624723](../_image/image-20191113125624723.png)

![image-20191113125714058](/Users/stellazhao/research_space/EasyMLBOOK/_image/image-20191113125714058.png)



## 参考资料
1. https://www.youtube.com/watch?v=pXGqDiE4N0I
