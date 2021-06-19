---
title: 深度学习基础1 数据表示和神经网络基础
author: chiechie
mathjax: true
date: 2021-06-10 16:35:36
tags:
- 深度学习
- low level
- 最佳实践
categories: 
- AI
---


# 数据表示


在深度学习中如何表示现实中的事物？

Let’s make data tensors more concrete with a few examples similar to what you’ll encounter later. The data you’ll manipulate will almost always fall into one of the following categories:

- Vector data—2D tensors of shape(samples,  features)
- Timeseries data or sequence data—3D tensors of shape (samples, timesteps, features)
- Images—4D tensors of shape(samples,height,width,channels)or(samples,channels, height, width)
- Video —5D tensors of shape (samples, frames, height, width, channels) or (samples, frames, channels, height, width)

## 表数据

- 两个轴：samples axis 和 features axis.
- 文本数据, 假设词典长度为2k，每一个doc可以表示为1个2k维的向量，位置的值代表词在文本中出现的次数。
- 500个文件可以存储为(500, 20000).

## 时间序列数据

- ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2F_vMGBboznU.png?alt=media&token=73c12f97-acdb-4680-b4eb-79a9a07581f9)
  
时间序列数据一般表示成3-d的张亮，tf中使用3d张量存储，(samples, timestamp, features)
每一个sample可以被编码成一个2d张量， 具体的两个例子

1. 股票数据：每年有250个交易日，每个交易日的交易时长有390分钟，每个分钟可以抽取3个重要特征：当前价格，上一分钟最高成交价格，上一分钟最低价格
    - 以每一天的交易数据为1个样本，构建的样本的shape为(250,390,3)
2. TWEET数据：一条twitter长度不超过256，每个位置的字符来自128个assical码中的一个。每一条twitter的shape为（256， 128），1 百万 tweets 的shape为(1000000, 280, 128)

## 图像数据

图像数据一般表示成4维tensor，一个图像数据就是一个3d张量

![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FD24ghIvE2L.png?alt=media&token=4a68bc5a-7b95-46d2-8418-deed053d4301)

- tensorflow： (samples, height, width, color_depth)
- theano：(samples, color_depth, height, width). 
- keras两者都支持

## 视频数据的表示

视频数据表示成5维张量：(samples, frames, height, width, color_depth).

# 模型超参

## 结构超参数

结构超参数包括层数、层的类别、层的大小等数值。以一个卷积层为例，其中的超参数包括卷积核 (filter) 的大小，卷积核的数量，步长 (stride) 的大小，unit：单元，hidden_layers: 隐藏层，cell：单元 ，dropout_rate丢弃率。这些超参数决定了神经网络的结构。

### layers的基本类型

keras中layer的基本类型：

- 卷积(Conv1D)
- 池化(MaxPooling1D/GlobalAveragePooling1D)
- dropout(Dropout)
- 线性全连接层(Dense)
- high-level层(LSTM)
    
### 卷积层

1. 卷积的维度有1Ｄ，2Ｄ，3D, 区别在于? 

输入数据的shape以及卷积核如何滑动

- ![I１维卷积和２维卷积的区别](https://miro.medium.com/max/1621/1*aBN2Ir7y2E-t2AbekOtEIw.png)
- 1维卷积（Conv1D）：只有1个维度可以滑动，但是另一维度也是有待估参数的
- 2维卷积（Conv2D）：有两个维度可以滑动，
- 相同点：从参数视角都是2维的，每一个卷积kernel，参数个数是 height × width +１　（off_ set）


2. 什么情况下，1d比2d好用呢？其中2个维度上，做卷积没有意义(high,close,open,close)
    

### 关于dropout

keras中对dropout的处理，因为训练阶段需要dropout ，但是inference阶段不需要dropout，keras中如何设置？

Keras does this by default. In Keras dropout is disabled in test mode. You can look at the code[here](https://github.com/keras-team/keras/blob/dc95ceca57cbfada596a10a72f0cb30e1f2ed53b/keras/layers/core.py#L109) and see that they use the dropped input in training and the actual input while testing.

As far as I know you have to build your own training function from the layers and specify the training flag to predict with dropout (e.g. its not possible to specify a training flag for the predict functions). This is a problem in case you want to do GANs, which use the intermediate output for training and also train the network as a whole, due to a divergence between generated training images and generated test images.


## 算法超参数

算法超参数包括学习率 (learning rate),批量大小 (batch size)、epoch数量(迭代周期个数)、正则，损失，激活函数等。由于神经网络的非凸性，用不同的算法超参数会得到不同的解。

### 归一化

归一化层，目前主要有这几个方法：

- Batch Normalization（BN， 2015年）
- Layer Normalization（LN， 2016年）
- Instance Normalization（IN， 2017年）
- Group Normalization（GN， 2018年）
- Switchable Normalization（SN， 2018年）；
  
![几种归一化方法g](img_1.png)

#### 问题1 transformer 为什么要使用LN而不是 BN？

   
 在[paper: Rethinking Batch Normalization in Transformers](https://arxiv.org/pdf/2003.07845.pdf)中, 作者对比了cv和nlp的BN, 得出的结论是在nlp数据上基于batch的统计信息不稳定性过大(相比cv的数据)，导致bn在nlp上效果差。相比之下layer norm能够带来更稳定的统计信息，有利于模型学习

Batch Normalization主要的问题是计算归一化统计量时计算的样本数太少，在RNN等动态模型中不能很好的反映全局统计分布信息，而Layer Normalization根据样本的特征数做归一化，是batch size无关的，只取决于隐层节点的数量，较多的隐层节点数量能保证Layer Normalization归一化统计分布信息的代表性。

#### 问题2. IN直观上怎么理解？
   
在计算机视觉中，IN本质上是一种Style Normalization，它的作用相当于把不同的图片统一成一种风格。另外，既然IN和BN都会统一图片的风格，那么在Generator里加IN或BN应该是不利于生成风格多样的图片的，论文中也进行了展示：

![](https://pic2.zhimg.com/v2-235433127838fca762ebd10511de9ca7_b.jpg)

图e是在generator中加了BN的结果，图f是在generator中加了IN的结果。果然崩了，IN崩得尤其厉害。


### 激活函数

- ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2F4_fjPJ90ir.png?alt=media&token=9fb9e321-aed2-4c7b-bb6b-4762b7a38c81)
- ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FADUH_mebAQ.png?alt=media&token=65efc150-0f7b-42f9-8657-1ca131bfd8b5)
- ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FMWiLJVveHv.png?alt=media&token=55bc24ce-0531-4702-8d3d-394595bd4d6e)
- ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FFztLjU8MxY.png?alt=media&token=d7421445-afca-439e-85be-8631db133a41)
- https://missinglink.ai/guides/neural-network-concepts/7-types-neural-network-activation-functions-right/



### 损失函数

tensorflow中的损失函数:

- sparse_categorical_crossentropy: target是interger list，形状n*1； 
- categorical_crossentropy: target 是one-hot vector，target的shape跟模型输出一致，是n*k(类)。
    - $$C E(x)=-\sum\limits_{i=1}^{C} y_{i} \log f_{i}(x)$$
- binary_crossentropy: target 是interger list。
    $$B C E(x)_{i}=-\left[y_{i} \log f_{i}(x)+\left(1-y_{i}\right) \log \left(1-f_{i}(x)\right)\right]$$
- 如果输出标签维度为1，只能使用binary_crossentropy，否则程序会报错。不能直接使用categorical_crossentropy 或者sparse_categorical_crossentropy
- 如果输出标签维度为K：使用categorical_crossentropy 或者sparse_categorical_crossentropy
- 单标签多分类（multi-class）：softmax + CE
- 二分类：sigmoid + BCE
- 多标签多分类（multi-label）的情况：sigmoid + BCE
- 最小化交叉熵损失函数等价于最大化训练数据集所有标签类别的联合预测概率。


#### 优化算法




# 参考资料

1. [Batch Normalization](https://arxiv.org/pdf/1502.03167.pdf)
2. [Layer Normalizaiton](https://arxiv.org/pdf/1607.06450v1.pdf)
3. [Instance Normalization](https://arxiv.org/pdf/1607.08022.pdf)
6. [code](https://github.com/DmitryUlyanov/texture_nets)
4. [Group Normalization](https://arxiv.org/pdf/1803.08494.pdf)
5. [Switchable Normalization](https://arxiv.org/pdf/1806.10779.pdf)
6. [code](https://github.com/switchablenorms/Switchable-Normalization)
6. [有公式推导，写的很棒](https://blog.csdn.net/liuxiao214/article/details/81037416)
2. [用书比喻图像很好理解](https://www.jianshu.com/p/05de1f989790)
3. [Conditional Batch Normalization 详解](https://zhuanlan.zhihu.com/p/61248211)
4. [从Style的角度理解Instance Normalization](https://zhuanlan.zhihu.com/p/57875010)
5. [在时序数据上应用conv1D](https://blog.goodaudience.com/introduction-to-1d-convolutional-neural-networks-in-keras-for-time-sequences-3a7ff801a2cf)
7. https://zhuanlan.zhihu.com/p/57875010
8. https://stackoverflow.com/questions/47787011/how-to-disable-dropout-while-prediction-in-keras
