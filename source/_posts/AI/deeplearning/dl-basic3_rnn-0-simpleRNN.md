---
title: 深度学习基础4 循环神经网络1-simple RNN
author: chiechie
mathjax: true
date: 2021-03-10 10:47:27
tags:
- 神经网络
- 深度学习
- 人工智能
- RNN
- NLP
categories:
- AI
---

## 总结

- 在自然语言处理的问题中，数据量足够的情况下，更常用transformer。但是在小数据量上，RNN还是很有效的。
- 序列数据（sequential data）包括文本，语音，时时间序列等。
- 全连接神经网络（FC Nets）和卷积神经网络（Conv Nets）处理样本的方式是one to one，即输入一张图片，输出该图片的类别。同时输入样本的大小和输出的大小是固定的。
- 序列数据面对的问题是，many to one 或者many to many，即输入很长一篇文章，输出该文章所属的类别。同时输入样本的大小不是固定的，一篇文章可长可短，翻译的输出也是可变的。RNN很适合。
- RNN运行原理，类似人脑处理信息的过程：阅读一篇长文章时，是逐句阅读，渐渐积累信息，然后再阅读下一个词。
- RNN接受最新时刻word的embedding向量，通过内部单元权重将embedding向量和上一个时刻的隐藏状态映射为最新的隐藏状态。
- RNN也可以看成一个带状态的函数，输入当前词的embedding向量（维度为m）和上一个时刻的状态向量（h维）输出当前时刻的状态向量（h维）。
- 最后一个状态向量$h_t$，积累整句话的所有信息，所以可以只保留这个state，将更老时刻的state都丢掉。
- $h_t$也可以作为整个序列的特征，提供给下游使用。
- simple rnn有一个Long term dependency problem，即不擅长对长时间建模，因为梯度消失了，最老的state的梯度要么爆炸要么消失，而梯度是调整权重的指路灯，灯坏了，权重就不知道该怎么调了。
- simple rnn很擅长对短间隔依赖关系建模，离输出层最近的梯度还是没毛病的
- 怎么解决Long term dependency problem？主要通过引入门控来达到这个目的，这一类网络也叫基于门控的循环神经网络（Gated RNN）,主要是LSTM和GRU。

## simeple RNN

- 待估参数：权重矩阵A，行数为：dim(h)，列数为：dim(h) + dimm(x)
  ![img.png](img.png)
- 激活函数：双曲正切函数

  为什么要使用双曲正切作为激活函数？使用别的方法（比如relu），隐藏状态的值传着传着就爆炸了或者没了。，
  如果A的特征根大于1，向前传播时，状态值呈指数增长，时间足够长，状态向量取值会爆炸
  如果A的特征根小于1，向前传播时，状态值呈0指数衰减，时间足够长，状态向量取值会接近0
  
  双曲正切作为激活函数的好处是，将每个rnn内部线性映射之后的结果scale到-1到1之间。所以向前传播时，不会出现指数爆炸或者衰减。

- simple RNN的参数个数：即矩阵A的大小 + intercept, dim(h) ✖ [dim(h) + dimm(x)] + dim(h)

  > keras在实现RNN默认使用intercept，所以还有一个intercept参数，大小是dim(h)

## simple-RNN应用-情感判断

判断电影评价是正面还是负面, 

![img_1.png](dl-basic2/img_1.png)


## 参考
1. [wangshusen-RNN-youtube](https://www.youtube.com/watch?v=Cc4ENs6BHQw&list=PLvOO0btloRnuTUGN4XqO85eKPeFSZsEqK&index=3)
2. [wangshusen-slide-github](https://github.com/wangshusen/DeepLearning)
3. [神经网络与深度学习](https://nndl.github.io/nndl-book.pdf)