---
title: 循环神经网络2-长短期记忆网络（LSTM）
author: chiechie
mathjax: true
date: 2021-05-19 10:48:04
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

1. LSTM是一种RNN模型，是对Simple RNN的一种改进，为了避免长期的梯度消失问题。
2. LSTM也可以看成一个带状态的函数，输入当前词的embedding向量（维度为m）和上一个时刻的状态向量（h维）和上一个时刻的记忆单元向量（h维），输出当前时刻的状态向量（h维）和当前时刻的记忆单元向量（h维）。
3. LSTM通过引入一个传输带（conveyor belt）来缓解梯度消失的问题。过去的信息通过传输带到达下一个时刻，不会发生很大变化。
   
## LSTM

![](./lstm.png)

LSTM引入新的内部状态专门进行先行的循环信息传递，同时将非线性加工后的信息给隐藏层的外部状态

lstm有3个门，分别为输入门，输出门，遗忘门，也是三个权重向量

- 输入门：一个权重向量，控制当前时刻的候选状态$\mathbf{C}_{t}$有多少信息要保留
- 输出门：控制当前内部状态$\tilde{\mathbf{c}}_{t}$有多少信息要输出给外部状态
- 遗忘门：一个权重向量表示上一个时刻的内部状态$\mathbf{c}_{t-1}$需要遗忘多少信息

![](./img.png)

更新当前记忆
![](./img_1.png)

更新当前时刻的state

![](./img_2.png)


lstm的参数个数：

三个门参数 + 记忆单元参数: (dim(x) + dim(h)) ✖ dim(h) ✖4


## LSTM应用-情感判断

![](./img_3.png)


## cnn 和 rnn视觉对比

有两个图还蛮有意思的，

![cnn](https://miro.medium.com/max/3058/1*W34PwVsbTm_3EbJozaWWdA.jpeg)

![rnn](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FoIsH5iVKwV.png?alt=media&token=05e8189e-dd5f-4781-910c-a46bb9fa4eaf)


## cnn vs rnn vs attention

 ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2F_n2z_XQqI2.png?alt=media&token=facfccac-e8ac-4895-a84c-7add43cd165a)



## 参考资料
1. [a-comparison-of-dnn-cnn-and-lstm-using-tf-keras](https://towardsdatascience.com/a-comparison-of-dnn-cnn-and-lstm-using-tf-keras-2191f8c77bbe)
2. [dive into deep learning-lstm](https://zh.d2l.ai/chapter_recurrent-neural-networks/lstm.html)
3. [wangshusen-RNN-youtube](https://www.youtube.com/watch?v=Cc4ENs6BHQw&list=PLvOO0btloRnuTUGN4XqO85eKPeFSZsEqK&index=3)
4. [wangshusen-slide-github](https://github.com/wangshusen/DeepLearning)
5. [神经网络与深度学习](https://nndl.github.io/nndl-book.pdf)