---
title: 深度学习6 word2vec
author: chiechie
mathjax: true
date: 2021-03-12 22:23:20
tags:
- NLP
- word2vec
- 人工智能
categories:
- 深度学习
---

> 词嵌入（word embedding）是指把一个词映射为一个实数域的向量，这个向量也叫词向量。
> 
> word2vec是一个词嵌入模型，有两个变形，跳字模型和连续词袋模型。跳字模型是基于中心词来生成背景词。连续词袋模型假设基于背景词来生成中心词。
> 
> word2vec模型学习到了一个词的两个表示（背景词向量和中心词向量），下游节点用的是哪个呢？

## why word2vec？

实现词嵌入的方法有很多，最简单的就是one-hot，即，先构建一次词典，然后将该word映射为索引向量。

那么为什么还要提出来word2vec方法？直接使用one-hot有什么问题吗？

one-hot vector没有表达word在语义上的相似性。
举个例子，假如用余弦函数来描述word之间的相似度，
那么one-hot的表达方式，就会使所有word之间的相似度为0，这样就丢失了词语的语义上的信息。

所以，需要一种词的表示方法，能表达语义上的相关性，word2vec就应运而生了。

## what word2vec？

wod2vec跟传统的线性降维方法如PCA一样，本质上都是在寻找对原始输入（词语）的一种更经济、高效的表达。

PCA和Word2vec的区别在于，两者对于何为「最优表达」的定义不一样：

- PCA认为重构误差最小的表达，就是最好的表达。即经过编码-解码之后,重构回原来空间时丢失尽可能少。
- word2vec认为如果一个中心词的表达跟上下文表达关联最大，就是最好的表达。即经过编码-解码之后，中心词映射为背景词，或者，背景词映射为中心词。想预测另外一个东西。 其中，前者是（Skip-gram model，SG），后者是连续词袋模型（Continuous Bag-of-Words，CBOW）。

![img.png](img.png)


## how word2vec?

先看跳字模型

### 跳字模型（skip gram）--基于中心词来生成背景词

#### high-level

跳字模型就是将一个中心词映射成多个背景词，大概思路，

![](./img1.png)

1. 对词典中每个词，构建2个向量表示，这个是待估参数。
2. 类似多分类问题，构造一个映射函数，输入中心词的one-hot（长度为$\mathcal{V}$），输出背景词的概率分布（长度为$\mathcal{V}$）
3. 类似逻辑回归的参数估计，使用交叉熵作为损失函数。

总体来说，从low-level，跳字模型，就很像是一个多分类的逻辑回归，跳字模型保留的是参数，而逻辑回归需要用的是输出的概率。


下面是一些细节可以跳过

#### 细节

- 词典索引集： $\mathcal{V} =\{0,1, \ldots,|\mathcal{V}|-1\}$
- 假设某个词在词典中的索引为i，这个词被表示成2个d维向量:
    - 中心词向量：$\boldsymbol{v}_{i} \in \mathbb{R}^{d}$
    - 背景词向量：$\boldsymbol{u}_{i} \in \mathbb{R}^{d}$
- 模型的输出是条件概率，假设中心词$w_c$在词典中的索引为c，背景词$w_o$在词典中的索引为o，那么给定中心词生成背景词的条件概率为
    
  $$P\left(w_{o} \mid w_{c}\right)=\frac{\exp \left(\boldsymbol{u}_{o}^{\top} \boldsymbol{v}_{c}\right)}{\sum\limits_{i \in \mathcal{V}} \exp \left(\boldsymbol{u}_{i}^{\top} \boldsymbol{v}_{c}\right)}$$
- 假设给定一个长度为 T 的文本序列，设时间步 t 的词为 w(t) 。假设给定中心词的情况下背景词的生成相互独立，当背景窗口大小为 m 时，跳字模型的似然函数即给定任一中心词生成所有背景词的概率
    $$P\left(w_{o} \mid w_{c}\right)=\frac{\exp \left(\boldsymbol{u}_{o}^{\top} \boldsymbol{v}_{c}\right)}{\sum_{i \in \mathcal{V}} \exp \left(\boldsymbol{u}_{i}^{\top} \boldsymbol{v}_{c}\right)}$$
这里小于1或大于 T 的时间步可以被忽略。


- 参数估计方法：MLE, 即最大化如下对数似然函数来估计参数，等价于最小化交叉熵（用来计算两个函数或者概率之间的距离）
    
    $$  \prod\limits_{t=1}^{T} \prod\limits_{-m \leq j \leq m, j \neq 0} P\left(w^{(t+j)} \mid w^{(t)}\right)$$
    等价于最小化如下 损失函数
    $$ -\sum\limits_{i=1}^{T} \sum\limits_{-m \leq j \leq m, j \neq 0} \log P\left(w^{(t+j)} \mid w^{(t)}\right)$$
    根据定义
    $$\log P\left(w_{o} \mid w_{c}\right)=\boldsymbol{u}_{o}^{\top} \boldsymbol{v}_{c}-\log \left(\sum\limits_{i \in \mathcal{V}} \exp \left(\boldsymbol{u}_{i}^{\top} \boldsymbol{v}_{c}\right)\right)$$
    
    按照梯度上升的思路，先求损失函数其中一项相对于$v_c$的梯度
    $$\begin{aligned} \frac{\partial \log P\left(w_{o} \mid w_{c}\right)}{\partial \boldsymbol{v}_{c}} &=\boldsymbol{u}_{o}-\frac{\sum_{j \in \mathcal{V}} \exp \left(\boldsymbol{u}_{j}^{\top} \boldsymbol{v}_{c}\right) \boldsymbol{u}_{j}}{\sum_{i \in \mathcal{V}} \exp \left(\boldsymbol{u}_{i}^{\top} \boldsymbol{v}_{c}\right)} \\ &=\boldsymbol{u}_{o}-\sum_{j \in \mathcal{V}}\left(\frac{\exp \left(\boldsymbol{u}_{j}^{\top} \boldsymbol{v}_{c}\right)}{\sum_{i \in \mathcal{V}} \exp \left(\boldsymbol{u}_{i}^{\top} \boldsymbol{v}_{c}\right)}\right) \boldsymbol{u}_{j} \\ &=\boldsymbol{u}_{o}-\sum_{j \in \mathcal{V}} P\left(w_{j} \mid w_{c}\right) \boldsymbol{u}_{j} \end{aligned}$$
    
训练结束后，对于词典中的任一索引为i的词，我们均得到该词作为中心词和背景词的两组词向量 $v_i$ 和 $u_i$ 。在自然语言处理应用中，一般使用跳字模型的中心词向量作为词的表征向量。

#### 使用tensorflow实现一个跳字模型

##### 建模
word2vec将每个词表示成一个定长的向量，并使得这些向量能较好地表达不同词之间的相似和类比关系

- 第一步：获取训练样本
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FjpuHAmzTik.png?alt=media&token=47e0004c-bc7c-4fc5-af22-d227532a7548)
    
- 第二步：构建1个神经网络
    - 输入：上面的中心词的one-hot向量，形状为（词典大小，）
    - 输出：中心词周围出现的词的概率向量（相近词的概率），形状也是（词典大小）
    - ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FsFuwRdSRxR.png?alt=media&token=03f6ea4c-aee0-4e11-993f-468505022f8d)
    - 模型参数（or权重向量）的形状和含义：
        - 网络 第一层有300（词向量长度）个单元，权重向量的形状是 (词典大小，词向量长度)，表示 每个词的中心词向量
        - 网络 第二层有10000（词典大小）个单元，权重向量的形状是（词向量长度，词典大小），表示 每个词的背景词向量

##### 评估

- 影响词向量质量的三个因素：
    - 训练数据的数量和质量
    - 词向量的大小
    - 训练算法

google提供了 测试数据 和 测试脚本

- chiechie：先人工定义一些 近义词组，反义词组，不相关词组，计算这些词组的 余弦距离，看是否跟之前定义的语义距离 一致。
- 提供2份 相关性测试集（relation test set）:
    - word relation test set :**./demo-word-accuracy.sh**,
    -  phrase relation test set:**./demo-phrase-accuracy.sh**
- 最好的结果：准确率 70% + 覆盖率 100%.


### 连续词袋模型（continous bag of words）

连续词袋模型与跳字模型类似。与跳字模型最大的不同在于，连续词袋模型假设基于某中心词在文本序列前后的背景词来生成该中心词。在同样的文本序列“the”“man”“loves”“his”“son”里，以“loves”作为中心词，且背景窗口大小为2时，连续词袋模型关心的是，给定背景词“the”“man”“his”“son”生成中心词“loves”的条件概率（如图10.2所示），也就是

![](img2.png)

一般使用连续词袋模型的背景词向量作为词的表征向量。

## 优化版

梯度上式的复杂度为|V|, 为了提高计算效率，有两个近似训练方案：负采样 和 层次化softmax

### 近似的学习方法-负采样

负采样的想法是，最后一层不用softmax，而是sigmoid,损失函数交叉熵：观测到就是正样本，没观测到就是负样本（通过对词典采样K次得到）
（类似异常检测）

$P\left(w^{(t+j)} \mid w^{(t)}\right)=P\left(D=1 \mid w^{(t)}, w^{(t+j)}\right) \prod\limits_{k=1, w_{k} \sim P(w)}^{K} P\left(D=0 \mid w^{(t)}, w_{k}\right)$

取对数就是

$\begin{aligned}-\log P\left(w^{(t+j)} \mid w^{(t)}\right) &=-\log P\left(D=1 \mid w^{(t)}, w^{(t+j)}\right)-\sum_{k=1, w_{k} \sim P(w)}^{K} \log P\left(D=0 \mid w^{(t)}, w_{k}\right) \\ &=-\log \sigma\left(\boldsymbol{u}_{i_{t+j}}^{\top} \boldsymbol{y}_{i_{t}}\right)-\sum_{k=1, w_{k} \sim P(w)}^{K} \log \left(1-\sigma\left(\boldsymbol{u}_{h_{k}}^{\top} \boldsymbol{v}_{i_{t}}\right)\right) \\ &=-\log \sigma\left(\boldsymbol{u}_{i_{t+j}}^{\top} \boldsymbol{y}_{i_{t}}\right)-\sum_{k=1, w_{k} \sim P(w)}^{K} \log \sigma\left(-\boldsymbol{u}_{h_{k}}^{\top} \boldsymbol{v}_{i_{t}}\right) \end{aligned}$

好处是：计算 梯度的 复杂度 从N 减少到 K




## 点积or余弦

> word2vec中的优化目标是点积，但实际使用的却是使用向量夹角的余弦？


1. 点积和余弦, 差了一个量岗，即向量模长。
2. 如果是在一个空间中找近义词，肯定是要用带归一化的距离函数--余弦；
3. 在训练的时候，为了迁就损失函数-sigmod，才将量岗还原。

## detail

- 训练时，优化的是input/output vector的点积，预测时使用的是夹角余弦.
- 训练的时候之所以不用cos相似度，而用点积，是因为如果先做cos，会将神经网络的输出限制到[-1, 1]，使网络的表达能力（p=σ(w*c)）受限，performance下降很多。w, c来自不同的空间， w为input vector，c为output vector。
- 如果效仿word2vec设计(user, item)这类点击率预估模型，二者的不同之处：
    - word2vec使用的是input vector，点击率预估模型使用的是最后参与点积运算的user vector, item vector；
    - word2vec预测时是求word的knn word（word2vec的源码文件distance.c，用的是余弦距离），
    - 点击率预估模型求的是user的knn item

## 参考
1. https://mk.woa.com/q/267975?strict=true&ADTAG=daily
2. Distributed Representations of Words and Phrases and their Compositionality, Section 2



## 参考
1. [word2vec开源实现](https://github.com/tmikolov/word2vec)
2. [dive into deep learning](https://zh.d2l.ai/chapter_natural-language-processing/word2vec.html#%E8%BF%9E%E7%BB%AD%E8%AF%8D%E8%A2%8B%E6%A8%A1%E5%9E%8B)