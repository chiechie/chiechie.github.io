---
title: 深度学习基础知识
author: chiechie
mathjax: true
date: 2021-06-10 16:35:36
tags:
categories:
---



## 深度学习中集中归一化的方法

归一化层，目前主要有这几个方法：

- Batch Normalization（2015年）
- Layer Normalization（2016年）
- Instance Normalization（2017年）
- Group Normalization（2018年）
- Switchable Normalization（2018年）；
  
![几种归一化方法g](img_1.png)

### 问题：
1. transformer 为什么要使用layer nmormalization 而不是 BN？
   
    - 回答1: 在[paper: Rethinking Batch Normalization in Transformers](https://arxiv.org/pdf/2003.07845.pdf)中, 作者对比了cv和nlp的BN, 得出的结论是在nlp数据上基于batch的统计信息不稳定性过大(相比cv的数据)，导致bn在nlp上效果差。相比之下layer norm能够带来更稳定的统计信息，有利于模型学习
    - 回答2: Batch Normalization主要的问题是计算归一化统计量时计算的样本数太少，在RNN等动态模型中不能很好的反映全局统计分布信息，而Layer Normalization根据样本的特征数做归一化，是batch size无关的，只取决于隐层节点的数量，较多的隐层节点数量能保证Layer Normalization归一化统计分布信息的代表性。

2. instance normalization 直观上怎么理解？
   
    - 分享一种理解Instance Normalization (IN) 的新视角：在计算机视觉中，IN本质上是一种Style Normalization，它的作用相当于把不同的图片统一成一种风格。这个视角是在黄勋学长和Serge Belongie大大的《[Arbitrary Style Transfer in Real-time with Adaptive Instance Normalization](http://link.zhihu.com/?target=https%3A//arxiv.org/abs/1703.06868)》[1] 中看到的。
    - 链接：https://zhuanlan.zhihu.com/p/57875010
    - 另外，既然IN和BN都会统一图片的风格，那么在Generator里加IN或BN应该是不利于生成风格多样的图片的，论文中也进行了展示：
    - ![](https://pic2.zhimg.com/v2-235433127838fca762ebd10511de9ca7_b.jpg)
    - 图e是在generator中加了BN的结果，图f是在generator中加了IN的结果。果然崩了，IN崩得尤其厉害。


## 激活函数

- ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2F4_fjPJ90ir.png?alt=media&token=9fb9e321-aed2-4c7b-bb6b-4762b7a38c81)
- ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FADUH_mebAQ.png?alt=media&token=65efc150-0f7b-42f9-8657-1ca131bfd8b5)
- ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FMWiLJVveHv.png?alt=media&token=55bc24ce-0531-4702-8d3d-394595bd4d6e)
- ![](https://firebasestorage.googleapis.com/v0/b/firescript-577a2.appspot.com/o/imgs%2Fapp%2Frf_learning%2FFztLjU8MxY.png?alt=media&token=d7421445-afca-439e-85be-8631db133a41)
- https://missinglink.ai/guides/neural-network-concepts/7-types-neural-network-activation-functions-right/


## 其他实践

- updatede0929-最近做时序预测实验的一些经验：
    - 1. 不同品种的训练数据，分组训练 比 汇总训练 效果好。
    - 2. 一定要汇总训练的话，要加入每个品种的静态特征，不然会学到 混乱的模式。
    - 3. 数据颗粒度太细时 噪声很大， 直接丢给模型，也可能造成模型学不好。（这个已经在quantML实践上面得到了证实）
        - 为啥同样是lstm，商品期货准确率不及股指期货？
            - 因为1min的股指期货数据平滑，噪声少，而1min的商品期货毛刺非常多
            - 将1min的商品期货聚合成5min之后，再使用过去1h的数据预测未来1h的数据，准确率非常之高
- 问一个问题，调参集的mse比训练集低，但相关系数训练集远远大于调参集，这是有bug？
- [[update1015-使用keras实现简单版tcn]]
- [[keras调试小技巧]]
- [[Amazon SageMaker Debugger内置规则]]
- [[AI分类如何处理单个badcase？]]



## 参考资料
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
