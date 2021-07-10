---
title: 深度学习10 非监督学习
author: chiechie
mathjax: true
date: 2021-07-09 18:20:08
tags: 
- 人工智能
- 深度学习
- backtesting
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

## 生成模型0-summary

1. 简单的生成模型比如直方图/高斯分布。复杂的生成模型比如自回归模型（Autoregressive Models）

2. 自回归模型的技术关键词有参数化分布（Parameterized distributions） 和最大似然（maximum likelihood）。

3. 常见的两个自回归模型有循环神经网络（Recurrent Neural Nets）和遮挡模型（Masking-based Models）

4. 基于likelihood的模型要解决的任务是什么？

  1. 生成数据：合成的图像，合成的语音，合成的文本，合成的视频

  2. 数据压缩：对原始文件高效编码，原始数据的entropy越大，压缩比例就越低，怎么知道当前数据的entropy有多大从而使用合适的压缩比例呢？需要去学习一个模型，这个模型可以输出当前数据的信息含量。

  3. 异常检测：比如在自动驾驶的场景中，如果遇到了一个训练样本中未见过的样本，就不一样让机器强行决策，正确的方法是，使用异常检测--将该场景先和训练样本先对比一下--如果该场景是异常的，就让人来接手。

    > Good!!!可以用在回测中

5. 基于likelihood的模型怎么实现？从样本数据中估计出样本分布的参数。

6. The field of deep generative models is concerned with jointly designing these ingredients to train *flexible and powerful* models $$p_\theta$$ capable of approximating distributions over high-dimensional data $$\mathbf{x}$$.

7. 怎么设计分布函数？effectively represent complex joint distributions over x,并且yet remain easy to train

8. 设计玩分布函数之后，使用相应的训练方法hand-in-hand学习参数，损失函数=真实分布和拟合分布的距离：

9. 最大似然的损失函数为样本的对数概率的相反数，记为：

  ​	$$\arg \min _{\theta} \operatorname{loss}\left(\theta, \mathbf{x}^{(1)}, \ldots, \mathbf{x}^{(n)}\right)=\frac{1}{n} \sum_{i=1}^{n}-\log p_{\theta}\left(\mathbf{x}^{(i)}\right)$$

  等价于
  $$
  \arg \min _{\theta} \mathbb{E}_{\mathbf{x} \sim \hat{p}_{\text {data }}}\left[-\log p_{\theta}(\mathbf{x})\right]
  $$

10. 求解最大似然对应的优化问题--SGD。why？兼容神经网络，即使用神经网络去代替猜想分布。


13. 现在要做的事情就是设计一个神经网络来代替$p_{\theta}$,主要想法是：设计一个贝叶斯网络 **Bayes net** structure，网络中包括其他变量，并且使用神经网络来对条件概率分布建模。 将问题转换为，设计一个基于likelihood-based 的模型，对于每一个变量（下图中的一个节点），很简单，将条件变量的值输入nn，nn会输出待预测变量的概率分布。其实就是构建一个nn去拟合边的属性---即在条件变量取不同的值时，相应的变量的概率分布是如何变化的。

![img](/Users/shihuanzhao/research_space/chiechie.github.io/source/_posts/deeplearning/dl10_lvm/K1rwN9LAHJs509x8IvCcix_7CkI6u8c5YsPUT0R1eoxuP2WvACnAQTVQQVAuqoWMtO5JIflxLnrNl_eFPnF1F3XBT658a6wyLfOSaynjAjyyWf5nFBwSww-TDvy7oYpwNCD3bRHo3Rc.png)


11. 统计学家说 if the model family is expressive enough and if enough data is given, then solving the maximum likelihood problem will yield parameters that generate the data 

12. 跟最大似然等价的问题为最小KL--经验分布和拟合分布的距离

    经验分布假设样本来自iid，则每个观测值的分布都为1（就是直方图）

    $$\hat{p}_{\text {data }}(\mathbf{x})=\frac{1}{n} \sum_{i=1}^{n} \mathbf{1}\left[\mathbf{x}=\mathbf{x}^{(i)}\right]$$

    那么经验分布和猜想分布$p_{\theta}$之间的kl divergence为：

    $$\mathrm{KL}\left(\hat{p}_{\mathrm{data}} \| p_{\theta}\right)=\mathbb{E}_{\mathbf{x} \sim \hat{p}_{\mathrm{data}}}\left[-\log p_{\theta}(\mathbf{x})\right]-H\left(\hat{p}_{\mathrm{data}}\right)$$

## 生成模型1- 自回归模型

1. 给定贝叶斯网络，将条件概率分布设置为nn, 会得到一个可处理的对数likelihood和梯度，很好训练,

$$\log p_\theta(\mathbf{x}) = \sum_{i} \log p_\theta(x_i \,|\, \mathrm{parents}(x_i))$$

2. 但是用nn建模出来的条件概率的表达能力够强吗？任何联合概率分布都可以表达为条件概率的乘积。 

3. 自回归模型 **autoregressive model**:
   $$
   \log p(\mathbf{x})=\sum_{i=1}^{d} \log p\left(x_{i} \mid \mathbf{x}_{1: i-1}\right)
   $$
    一个使用nn预测条件概率的表达充分的贝叶斯网络，可以对p(x)构建一个表达充分的模型，同时也很好训练

4. 一个简单的自回归模型：

   两个随机变量: x1, x2

   对联合分布建模：Model: p(x1, x2) = p(x1) p(x2|x1)

   边际概率分布分布p(x1)是一个直方图

   条件概率分布p(x2|x1)是一个mlp，输入是$x_1$, 输出是$x_2$的logits

5. 如何让不同的边--即不同的标间概率之间共享信息？RNN和masking方法

6. [Karpathy, 2015]提出char-rnn方法

   ![image-20210710152800599](/Users/shihuanzhao/research_space/chiechie.github.io/source/_posts/deeplearning/dl10_lvm/image-20210710152800599.png)

## 生成模型2-隐变量模型

1. 为什么需要latent variable模型?对比下自回归模型：采样的计算量太大了，采样只能按照顺序进行。
2. latent varible只依赖latent varible，采样效率更高，相对于ar模型。
3. 如果知道生成数据的因果过程，就可以设计一个latent varible了。
4. 一般来说，我们不知道隐变量是什么，以及这些latent variables是怎么跟observation进行互动的。
  确定隐变量的最好的方法，仍未有定论。

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

4. 求最大似然的时候，z的维度太多了怎么办？采样。

5. 如果z的分布很难采样怎么办？分布变换：从原始的p转换成到q（q中也有待估计的参数），然后再q中采样，这就是变分（分布）法

   ![image-20210710154905312](/Users/shihuanzhao/research_space/chiechie.github.io/source/_posts/deeplearning/dl10_lvm/image-20210710154905312.png)

6. 怎么设计q？有一种常规的思路，找一个高斯分布的q，使得这个q尽可能逼近z的后验概率。

   ![image-20210710155220160](/Users/shihuanzhao/research_space/chiechie.github.io/source/_posts/deeplearning/dl10_lvm/image-20210710155220160.png)

7. 接着上面找q，即求解优化问题

   ![image-20210710155544308](/Users/shihuanzhao/research_space/chiechie.github.io/source/_posts/deeplearning/dl10_lvm/image-20210710155544308.png)

8. 变分推断（Amortized Inference





1. ）

   how?






## 参考
1. https://sites.google.com/view/berkeley-cs294-158-sp20/home

2. [L1 Introduction -- CS294-158-SP20 Deep Unsupervised Learning -- UC Berkeley, Spring 2020](https://www.youtube.com/watch?v=V9Roouqfu-M)
3. [https://drive.google.com/file/d/1zWvkB5BNFs1IzyXarsf6ItXpfEc2OfZc/view](https://drive.google.com/file/d/1zWvkB5BNFs1IzyXarsf6ItXpfEc2OfZc/view)
4. https://www.youtube.com/watch?v=V9Roouqfu-M
5. [L2 Autoregressive Models -- CS294-158-SP20 Deep Unsupervised Learning -- UC Berkeley, Spring 2020]( https://www.youtube.com/watch?v=iyEOk8KCRUw)

