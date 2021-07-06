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
- 时间序列
- LSTM
categories:
- AI
---

## 总结

1. LSTM是一个循环神经网络，即RNN模型，是对Simple RNN的一种改进，可以避免梯度消失问题, 可以有比simple RNN更长的记忆。LSTM的论文在1997年就发表了。
2. LSTM的结构比simple-RNN的结构复杂很多，Simple RNN有1个参数矩阵，LSTM有四个参数矩阵。
3. LSTM也可以看成一个带状态的函数，输入当前词的embedding向量（维度为m）和上一个时刻的状态向量（h维）和上一个时刻的记忆单元向量（h + m 维），输出当前时刻的状态向量（h维）和当前时刻的记忆单元向量（h维）。
5. LSTM通过引入一个传输带（conveyor belt）来缓解梯度消失的问题。过去的信息通过传输带到达下一个时刻，不会发生很大变化。
6. LSTM的处理流程可以描述为，对长期记忆选择性遗忘，对短期记忆选择性保留，两个求和得到了新的长期记忆；将新的长期记忆选择性输出，得到新的短期记忆。
7. lstm有3个门，分别为输入门，输出门，遗忘门，也是三个权重向量，取值范围从0到1
   - 遗忘门：一个权重向量表示上一个时刻的传送带向量$\mathbf{c}_{t-1}$需要遗忘多少信息
   - 输入门：一个权重向量，控制当前时刻的新的状态$z$有多少信息要保留
     
      > 遗忘门 + 输入门共同作用的结果是，得到新的传送带向量
     
   - 输出门：控制当前传送带向量$\tilde{\mathbf{c}}_{t}$有多少信息要输出给外部状态

## LSTM应用-情感判断

![](./img_3.png)

![使用keras实现lstm](./image_1212.png)

![img_13.png](./image_1213.png)


## cnn 和 rnn视觉对比

有两个图还蛮有意思的，

![cnn](https://miro.medium.com/max/3058/1*W34PwVsbTm_3EbJozaWWdA.jpeg)

![rnn](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FoIsH5iVKwV.png?alt=media&token=05e8189e-dd5f-4781-910c-a46bb9fa4eaf)


## cnn vs rnn vs attention

 ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2F_n2z_XQqI2.png?alt=media&token=facfccac-e8ac-4895-a84c-7add43cd165a)


## 附录

### LSTM的内部结构

1. LSTM最重要的设计是一个传输带（conveyor belt），记为向量$C_t$, 过去的信息通过传输带直接传到下一个时刻$C_{t+1}$ ，不会发生太大的变化。LSTM就是靠传输带来避免梯度小时的问题。
2. 遗忘门的输入是上一个时刻的状态$h_{t-1}$和当前时刻的观测值$x_t$，通过遗忘矩阵$W^f$加上一个sigomid变换，得到一个概率向量。
![](./lstm.png)。
![img_3.png](image_123.png)

3. LSTM的遗忘门（forget gate）的作用是，让传输带上的信息有选择性的通过，选择性是由遗忘门向量控制的，通过遗忘门向量与传送带向量$C_t$点乘来实现。
![img_2.png](image_122.png)
4. 输入门(input gate)的输入也是上一个时刻的状态$h_{t-1}$和当前时刻的观测值$x_t$。


   ![img_5.png](image_125.png)

5. 输入门的目的是，让上一个时刻的状态和最新的信息，能有选择性的通过。通过计算输入门向量跟$[h_{t-1}, x_{t}]$的双曲正切向量的点乘来实现。
   
   > 双曲正切函数可以把上一个时刻的状态和新的观测值组成的向量映射到【-1，1]之间，
   
6. 输出门(output gate)的输入也是上一个时刻的状态$h_{t-1}$和当前时刻的观测值$x_t$

![img_8.png](image_128.png)
   
7. 输出门的目的是，让最新的长期记忆有选择性的输出，作为状态向量$h_t$。计算状态向量, 将输出门向量$o_t$跟传输带最新值$c_t$的双曲正切映射进行点乘，得到新的状态向量$h_t$。
8. 新的状态向量$h_t$有两份copies，一份作为当前时刻的输出，一份作为下一个时刻lstm单元的输入。可以认为迄今为止，t个时刻的输入信息都被编码到了状态向量$h_t$中。
![img_9.png](image_129.png)
   
9. LSTM有遗忘门，输入门，new value，输出门，一共对应4个参数矩阵。矩阵的行数是状态向量的长度，矩阵的列数是状态向量的长度+输入向量的长度。

![img_10.png](./image_1210.png)

10. 更新长期记忆-传送带向量

![](./img_1.png)

11. 更新短期机器-状态向量

![](./img_2.png)

12. LSTM处理过程用数学公式表达
![img_5.png](img_5.png)
![img_4.png](img_4.png)


## 参考资料
1. [a-comparison-of-dnn-cnn-and-lstm-using-tf-keras](https://towardsdatascience.com/a-comparison-of-dnn-cnn-and-lstm-using-tf-keras-2191f8c77bbe)
2. [dive into deep learning-lstm](https://zh.d2l.ai/chapter_recurrent-neural-networks/lstm.html)
3. [wangshusen-RNN-youtube](https://www.youtube.com/watch?v=Cc4ENs6BHQw&list=PLvOO0btloRnuTUGN4XqO85eKPeFSZsEqK&index=3)
4. [RNN模型与NLP应用(4/9)：LSTM模型](https://github.com/wangshusen/DeepLearning)
5. [神经网络与深度学习](https://nndl.github.io/nndl-book.pdf)
6. [人人都能看懂的LSTM](https://zhuanlan.zhihu.com/p/32085405)