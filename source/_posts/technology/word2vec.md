---
title: word2vec的原理
author: chiechie
mathjax: true
date: 2021-04-10 22:23:20
tags:
- NLP
- word2vec
categories:
- 技术
---

> 灵魂拷问！
> 
> 如果把word2vec用neural nets来实现，神经网络的框架如何reformulate这个问题？模型参数是什么，词向量属于网络中的哪部分？
> 
> 既然模型学习到了一个词的两个表示（背景词向量 和 中心词向量），那么 下游节点用的是哪个表示呢？怎么用呢？

## 总结

我对wod2vec的理解: 本质上，是对一个word的one-hot vector降维的过程。

跟传统的降维方法如PCA比，区别在于，目标不一样：

- PCA： 经过编码-解码之后，信息丢失尽可能少。
- word2vec： 经过编码-解码之后，背景词可以映射为中心词（Continuous Bag-of-Words，CBOW），或者中心词映射为背景词（Skip-gram model，SG）

![img.png](./img.png)


下面重点说一下Skip-gram model的原理

### 为何不用one-hot vector表示一个word？

one-hot vector没法表达word在语义上的相似性，为什么呢？
一般用



### 跳字模型（skip gram）

每个词被表示成2个d维向量，

- 词典索引集： $\mathcal{V} =\{0,1, \ldots,|\mathcal{V}|-1\}$
- 词的索引为i，
- 中心词向量为：$\boldsymbol{v}_{i} \in \mathbb{R}^{d}$
- 背景词向量为：$\boldsymbol{u}_{i} \in \mathbb{R}^{d}$
- 模型的输出是：条件概率
- 目标函数：MLE
    $$ \max \prod\limits_{t=1}^{T} \prod\limits_{-m \leq j \leq m, j \neq 0} P\left(w^{(t+j)} \mid w^{(t)}\right)$$
    等价于   
    $$\max \sum\limits_{i=1}^{T} \sum\limits_{-m \leq j \leq m, j \neq 0} \log P\left(w^{(t+j)} \mid w^{(t)}\right)$$
    进一步，logP就是
    $$\log P\left(w_{o} \mid w_{c}\right)=\boldsymbol{u}_{o}^{\top} \boldsymbol{v}_{c}-\log \left(\sum\limits_{i \in \mathcal{V}} \exp \left(\boldsymbol{u}_{i}^{\top} \boldsymbol{v}_{c}\right)\right)$$

直接照着最大化条件概率的目标学习可以吗？

不行，可以看到上式的复杂度为|V|, 为了提高计算效率，有两个近似训练方案：负采样 和 层次化softmax

## 近似的学习方法-负采样

负采样的想法是，最后一层不用softmax，而是sigmoid,目标函数是 交叉熵：观测值就是正样本，未出现的就是负样本（通过对词典采样K次得到）

$P\left(w^{(t+j)} \mid w^{(t)}\right)=P\left(D=1 \mid w^{(t)}, w^{(t+j)}\right) \prod\limits_{k=1, w_{k} \sim P(w)}^{K} P\left(D=0 \mid w^{(t)}, w_{k}\right)$

取对数就是

$\begin{aligned}-\log P\left(w^{(t+j)} \mid w^{(t)}\right) &=-\log P\left(D=1 \mid w^{(t)}, w^{(t+j)}\right)-\sum_{k=1, w_{k} \sim P(w)}^{K} \log P\left(D=0 \mid w^{(t)}, w_{k}\right) \\ &=-\log \sigma\left(\boldsymbol{u}_{i_{t+j}}^{\top} \boldsymbol{y}_{i_{t}}\right)-\sum_{k=1, w_{k} \sim P(w)}^{K} \log \left(1-\sigma\left(\boldsymbol{u}_{h_{k}}^{\top} \boldsymbol{v}_{i_{t}}\right)\right) \\ &=-\log \sigma\left(\boldsymbol{u}_{i_{t+j}}^{\top} \boldsymbol{y}_{i_{t}}\right)-\sum_{k=1, w_{k} \sim P(w)}^{K} \log \sigma\left(-\boldsymbol{u}_{h_{k}}^{\top} \boldsymbol{v}_{i_{t}}\right) \end{aligned}$

好处是：计算 梯度的 复杂度 从N 减少到 K

## 将word2vec重定义为一个nn问题

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
- 影响词向量质量的三个元素：
    - 训练数据的数量和质量
    - 词向量的大小
    - 训练算法

## 怎么评估词向量的质量？

google提供了 测试数据 和 测试脚本

- chiechie：先人工定义一些 近义词组，反义词组，不相关词组，计算这些词组的 余弦距离，看是否跟之前定义的语义距离 一致。
- 提供2份 相关性测试集（relation test set）:
    - word relation test set :**./demo-word-accuracy.sh**,
    -  phrase relation test set:**./demo-phrase-accuracy.sh**
- 最好的结果：准确率 70% + 覆盖率 100%.
  


## 参考
1.[word2vec开源实现](https://github.com/tmikolov/word2vec)
