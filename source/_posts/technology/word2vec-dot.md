---
title: 点积or向量夹角？
author: chiechie
mathjax: true
date: 2021-04-02 15:47:19
tags:
categories:
---


> word2vec中的优化目标是点积，但实际使用的却是使用向量夹角的余弦？

## 总结

1. 点积和余弦, 差了一个量岗，即向量模长。
2. 如果是在一个空间中找近义词，肯定是要用自带归一化的距离函数的--余弦；
3. 在训练的时候，为了迁就损失函数-sigmod，才将量岗又还原的。

## detail

- 训练时，优化的是input/output vector的点积，预测时使用的是input vector之间的夹角
- 在训练时，word2vec的输出层为sigmoid/softmax，需要得到(当前词 w, 上下文词 c)的交互概率p=σ(w*c)。
- 如果这里将w与c向量进行归一化后计算点积，那w*c落入[-1, 1]，严重限制σ函数的表达能力(σ函数定义域本是-∞, +∞)。所以很少将向量归一化再求概率。
- 注意，w, c来自不同的空间。w为input vector，c为output vector。word2vec在预测时用到的是input vector作为词向量。
- 在预测时，word2vec的源码文件distance.c中，求词的最近邻使用的是cosine相似度，也就是归一化后的点积。
- 此外，如果效仿word2vec设计(user, item)这类点击率预估模型，预测时一般也不做归一化，直接使用点积召回user的knn item。
- word2cec和点击率模型不同之处有2点：
    - word2vec使用的是input层的wordEmb，点击率预估模型使用的是最后参与点积运算的userEmb, itemEmb；
    - word2vec预测时是求word的knn word，点击率预估模型求的是user的knn item；
- word2vec实际用的时候一般要先做归一化的，一般做L2_norm的
- 训练的时候之所以不用cos相似度，而用点积，是因为如果先做cos，会将神经网络的输出限制到[-1, 1]，使网络的表达能力受限，performance下降很多（我们之前做过类似实验）。

## 参考
1. https://mk.woa.com/q/267975?strict=true&ADTAG=daily
2. Distributed Representations of Words and Phrases and their Compositionality, Section 2
