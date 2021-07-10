---
title: 深度学习10 非监督学习
author: chiechie
mathjax: true
date: 2021-07-09 18:20:08
tags: 
- 人工智能
- 深度学习
categories: 
- 深度学习
---

## 非监督学习

1. 非监督学习是指，从原始数据中学习rich模式，以一种label free的方式。

2. 非监督学习又分为两类：生成模型和自监督学习。

3. 生成模型指的是, 学习原始数据的分布，例如基于高斯分布的异常检测

4. 自监督学习指的是, 学到关于原始输入的一些reprentation，可以做到语意理解，可能在未来的有一些任务中会有用，有点像做题训练（puzzle）。例如， Bert训练需要解决两个puzzle：遮挡词预测；下一个句子预测。再例如，对图像翻转90度/180度，喂给neural network，让nn来预测对应的操作（90/180），这个似乎在监督学习中用的很多

5. 非监督学习有哪些应用？

   1. 生成正常数据，比如生成回测中的合成数据

   2. 条件的合成技术,包括WaveNet和GAN-pix2pix，比如talktotranfomer.com

   3. 数据压缩

   4. 【最重要】在监督学习之前做自监督学习可以提升下游任务的准确率。

      > 这个技术在工业界已经有应用了：谷歌搜索是由bert支持的

6. What is true now,but not true ayear before?

   > 以前自监督的预训练比监督模型在检测和识别中效果更差，现在效果赶超有监督了

7. 目前已经成熟的技术：语言生成模型（GPT3），图像生成模型（VIT），预训练的语言模型（Bert），预训练的图像模型。

8. 目前还没有研究成熟的方向：自回归密度模型（AgressiveRgression Density Modelings），Flows，VAE，UnsupervisedLearning for RL。

## 生成模型

1. 简单的生成模型比如直方图/高斯分布。复杂的生成模型比如自回归模型（Autoregressive Models）
2. 自回归模型的技术关键词有参数化分布（Parameterized distributions） 和最大似然（maximum likelihood）。
3. 常见的两个自回归模型有循环神经网络（Recurrent Neural Nets）和遮挡模型（Masking-based Models）
4. 基于likelihood的模型要解决的任务是什么？
  1. 生成数据：合成的图像，合成的语音，合成的文本，合成的视频
  2. 数据压缩：创建有效的编码，原始数据的entropy越大，压缩比例就越低，怎么知道当前数据的entropy有多大从而使用合适的压缩比例呢？需要去学习一个模型，这个模型可以输出当前数据的信息含量。
  3. 异常检测：
5. 基于likelihood的模型怎么实现？从样本数据中估计出样本分布的参数。
6. 为什么需要latent variable模型?对比下自回归模型：采样的计算量太大了，采样只能按照顺序进行。
7. latent varible只依赖latent varible，采样效率更高，相对于ar模型。
8. 如果知道生成数据的因果过程，就可以设计一个latent varible了。
9. 一般来说，我们不知道隐变量是什么，以及这些latent variables是怎么跟observation进行互动的。
  确定隐变量的最好的方法，仍未有定论。







## what？
1. latent variable模型更像是一种更抽象的层次更高的认知。

2. 隐变量模型的一个例子：如下，Z是一个K维度的随机向量，X是一个L维的随机变量

   对Z抽样得到一个K维的0/1向量z，通过一个函数（DNN）映射为一个参数$\theta$, X的分布变为一个参数为$\theta$的bernoulli分布，从该分布中采样得到一个L维的0/1向量x。举例子，z代表股市是牛市/熊市，X代表股票上涨/下跌，牛市时X的分布的均值要高于熊市时X的分布均值

   ![image-20210709233421079](./image-20210709233421079.png)

3. 如何训练上面的DNN？最大似然估计，再往前可以追朔到亚里士多德的三段论，大前提，小前提==> 结论。

   大前提：我们观测到的一系列$x^{(i)}$, 是客观存在的，不是伪造的

   小前提：有一个隐变量X，服从某个bernouli分布，有一个可观测的随机变量X服从某个bernouli分布，并且该bernouli分布的参数跟X有关

   结论: 一系列$x^{(i)}$的概率很大

   为了使得三段论成立，我们希望找到一个最优的DNN，使得$x^{(i)}$的likelihood尽可能大。

   其实，即使likelihood很大，也不能一定保证小前提成立，这里只是折中罢了。

   ![image-20210709234327994](./image-20210709234327994.png)

4. 求最大似然的时候，z的维度太多了怎么办？

   重要性采样，找到另外一个随机变量，
   
   

## how?






## 参考
1. https://sites.google.com/view/berkeley-cs294-158-sp20/home

2. [L1 Introduction -- CS294-158-SP20 Deep Unsupervised Learning -- UC Berkeley, Spring 2020](https://www.youtube.com/watch?v=V9Roouqfu-M)
3. [https://drive.google.com/file/d/1zWvkB5BNFs1IzyXarsf6ItXpfEc2OfZc/view](https://drive.google.com/file/d/1zWvkB5BNFs1IzyXarsf6ItXpfEc2OfZc/view)
4. https://www.youtube.com/watch?v=V9Roouqfu-M
5. [L2 Autoregressive Models -- CS294-158-SP20 Deep Unsupervised Learning -- UC Berkeley, Spring 2020]( https://www.youtube.com/watch?v=iyEOk8KCRUw)

