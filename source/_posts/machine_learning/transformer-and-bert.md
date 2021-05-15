---
title: Transformer
author: chiechie
mathjax: true
date: 2021-04-18 00:04:13
tags: 
- NLP
- 神经网络
- 模型可视化
- Transformer
- Bert
categories:
- 技术
---

# Transformer

> 先概括Transformer的主要设计思路；再讲每一步的具体技术细节。
>
> 只想粗略了解Transformer的设计可以只看 “high-level ideas”这部分。

- Transformer是google2016年在论文《attention is all you need》提出的一个机器翻译模型。
- Transformer是一个很典型的seq2seq架构。
- Transformer的亮点在于将attention和self-attention机制完全剥离开之前rnn的结构，只跟dense层组合。


##  High-Level ideas

首先把模型看作一个黑盒。 在机器翻译（eg德译英）任务中，它会将一个句子（例如德文）翻译成一种语言（机器语言），然后再将其翻译成另一种语言（例如英文）。

打开Transformer的黑盒，我们看到一个encoders组件，一个decoders组件，以及它们之间的数据流。

![](./transformer_encoders_decoders.png)

encoders内部stacked encoders(论文堆了六个)，decoders内部也是stacked decoders(论文堆了六个)。

一个encoder或一个decoder叫做一个block。

![](./transformer_encoder_decoder_stack.png)

encoders包含的6个block，结构相同，但是不共享权重。

每个encoder block包含2层:

- self-attention层，参数不共享
- ffnn层，参数共享

![](./Transformer_encoder.png)

每个decoder block除了这两层，还有一个encoder-decoder attention层，用来关注encoder的输出，作用类似于  [seq2seq models](https://jalammar.github.io/visualizing-neural-machine-translation-mechanics-of-seq2seq-models-with-attention/) 中的attention。

![](./Transformer_decoder.png)

## 输入tensor

上面介绍了Transformer的主要组件，现在看一下组件之间的数据流向。

和一般的NLP任务一样，Transformer首先使用[embedding algorithm](https://medium.com/deeper-learning/glossary-of-deep-learning-word-embedding-f90c3cec34ca)将每个输入单词转换成一个向量。


![](./transformer_embeddings.png) 

如上图，每个单词都被embedded为大小为512的向量（上面的x1，x2，x3）。 

接下来，这个512的向量会流入self-attention和ffnn层。

![](./transformer_encoder_with_tensors.png)

接下来，我们以一个短句为例，看一下Transfomer内部细节。

## Encoder

前面提到，encoder的输入是多个向量构成的list：[x1，x2]。

接下来它是怎么处理这个list的呢？

1. 将这些向量传递到一个"self-attention’"层，输出z1，z2
2. 然后将z1，z2 丢入一个前馈神经网络，输出r1，r2。注意这一层的dense layer的参数对所有的z1，z2，是共享的。

然后将r1，r2 传递给下一个编码器encoder2作为输入，encoder1的输入（x1,x2）和输出（r1，r2）的维度是一样的。

![img](./transformer_encoder_with_tensors_2.png)

### self-attention

####  High Level ideas

假设下面的句子是我们要翻译的输入句子:

”`The animal didn't cross the street because it was too tired`”

这个句子中的"it"指的是什么？它指的是“street”还是“Theanimal”？对于人类来说很简单区分，但对于算法就不那么简单了。

当模型解读每个word时，self-attention会让模型回顾上下文信息，也就是输入序列中的其他word，从而帮助模型更好地理解这个word。

RNN通过hidden state策略，使得 它将 当前词  与 上下文（准确来说只有上文）的信息进行融合。  

类似的，Transformer通过self-attention来实现这个目的。

还是回到上面的例子，当模型处理单词"it"时，self-attention使它能够将"it"与"animal"联系起来。



![img](./transformer_self-attention_visualization.png) 

上图表示，当我们在encoder #5 中对"it"编码时，self-attention会将一部分注意力集中在"Animal"上，并将“Animal”的信息融入了"it"的编码中。

相关可视化工具参考Tensor2Tensor （[Tensor2Tensor notebook](https://colab.research.google.com/github/tensorflow/tensor2tensor/blob/master/tensor2tensor/notebooks/hello_t2t.ipynb) ）。

先看看如何使用向量计算self-attention，然后再看看使用矩阵如何实现。。

#### 第一步 计算3个向量

对于每个单词，我们创建三个向量：

- 一个 Query vector
- 一个 Key vector
- 一个 Value vector。

这些向量是通过将单词的embedding乘以三个矩阵来创建的。

请注意，得到的新向量的维度（64）比embedding（512）小。 

这么设计的目的是，保证在加入了multi-head self- attention之后，encoder的输出和输入维度（512）还能保持一致。

![img](./transformer_self_attention_vectors.png) 
这三个向量如何解读？为何有用？

#### 第二步 计算权重分数

第二步是计算权重分数$\alpha$，目的是量化 其他单词应该被关注的程度。

方法如下：

当前单词的query vector和其他单词的 key vector求 内积。

如下:

$<q_1, k_1>, <q_1, k_2>, ... < q1, k_m>$

![img](./transformer_self_attention_score.png) 



#### 第三步和第四步 -权重分数归一化

第三步：将第二步算出来的权重得分向量（长度为m）进行归一化，目的是让权重不要受到key vector的长度（paper里面是64）的影响，这样算梯度就更稳定，。

第四步：将归一化的权重得分向量（长度为m）送入激活函数（paper里面是softmax），目的是得到一个概率向量，所有数值加起来为1。

最终的向量，表示每个单词应该被当前单词关注的程度，值越大越应该被关注。。


![img](./transformer_self-attention_softmax.png) 



#### 第五步和第六步-计算context向量

第五步和第六步是将每个单词的value vector使用权重概率向量进行加权求和，作为当前单词的self-attention表达（长度为value vector的长度为64，用了8个头，一起输出是64* 8 = 512）。

![img](./transformer_self-attention-output.png) 

然后就结束了。

#### 使用矩阵运算对self-attention整个过程总结

把刚刚的过程再加入一个维度，也就是从一个单词 变成 多个单词，计算他们各自的self-attention表示，

下面用矩阵表达：

第一步是计算 Query、 Key 和 Value 矩阵：

- 我们将embeddings塞到一个矩阵x：行数表示单词个数，列表示embedding的长度

- 权重矩阵(WQ、 WK、 WV）：行代表embedding向量的长度，列分别代表query空间，key空间，value空间的维度
- X分别和这几个矩阵相乘

![img](./transformer_self-attention-matrix-calculation.png) 


第二步，利用第一步的结果来计算attention的输出，用一个矩阵计算来表达，简洁优雅

![img](./transformer_self-attention-matrix-calculation-2.png) 


### 多头怪兽

论文通过增加"多头"注意机制进一步细化self-attention layer。 这在两个方面改善了self-attention层的表现:

1. 它扩展了模型关注不同位置的能力。单头机制虽然会注意到其他单词，但是注意力还是 很有可能完全被当前自己的状态牵制。多头的话，相当于多审视几遍 这个注意力，减少完全 关注自己 情况发生的可能性。

2. 它给予attention层多个"表示子空间"。 正如我们接下来将看到的，通过多头，我们不仅有一组，而且有多组 query / key / value 权重矩阵(Transformer 使用八个头，因此我们最终为每个encoders / decoders设置了八组)。 这些集合中的每一个都是随机初始化的。 

   训练完后，每组的三个矩阵，用于将embedding输入(或来自底层的编encoders/decoders向量)映射到表征子空间。

![img](./transformer_attention_heads_qkv.png)

类似上面提到的单头self-attention计算，我们现在只是用8个不同的权重矩阵算了8次，并且得到了8个不同的 z 矩阵

![img](./transformer_attention_heads_z.png)


后面怎么跟dense层进行衔接呢？

1. 将这8个z矩阵进行列拼接（concat）
2. 拼接后的矩阵大小为m*512，丢入全连接层WO

![img](./transformer_attention_heads_weight_matrix_o.png)

这几乎就是multi-headed self-attention的全部内容。把整个过程放在一个图中描述：


![img](./transformer_multi-headed_self-attention-recap.png)


回顾一下之前的例子，当我们对"it"进行encoding时，8个注意力头分别关注什么？


先看2个头的情况：

如下图所示，在对"it"进行encoding时，一个head（黄色）最关注"animal"上，而另一个head（绿色）最关注“tired”——某种意义上说，这个模型对"it"表达融合了"animal"和“tied”信息。

![img](./transformer_self-attention_visualization_2.png)

再看8个头的情况：

如下图所示，我们把8个头的信息都表达在图中，就会变得很难解释：

![img](./transformer_self-attention_visualization_3.png) 

### 用位置编码（Positional Encoding）表示序列的顺序

到目前为止，我们的模型还没有考虑词的顺序关系。

为了解决这个问题，transformer向每个输入embedding向量（x1）又加上一个位置编码向量（t1）。



![img](./transformer_positional_encoding_vectors.png)


假设embedding的维度是4，那么实际的位置编码看起来是这样的:

![img](./transformer_positional_encoding_example.png)


这个模式意味什么？

在下图中，每一行对应一个向量的位置编码。第一行表示第一个word。

每行包含512个值，介于1和 -1之间。用颜色表示值的大小

![img](./transformer_positional_encoding_large_example.png) 



可以看到所有的位置编码向量被分成了两半。 左半边的值是由一个函数(使用正弦函数)生成的，而右半边的值是由另一个函数(使用余弦函数)生成的。 然后将它们进行拼接（concate）。

本文(3.5节)描述了位置编码[`get_timing_signal_1d()`](https://github.com/tensorflow/tensor2tensor/blob/23bd23b9830059fbc349381b70d9429b5c40a139/tensor2tensor/layers/common_attention.py)的公式。 这只是其中一种位置编码的方法。

### 残差连接和layer-normalization

encoder的架构中还有一个细节：在每个block中，self-attention和ffnn都有一个残差连接，然后接一个层标准化（layer-normalization）步骤。

![img](./transformer_resideual_layer_norm.png) 

如果我们将残差连接和layer normalization用矩阵表示，就是下图：

![img](./transformer_resideual_layer_norm_2.png) 



decoder的sub-layers 也同样用到了add & normalization的设计。

下图是一个简化版的Transformer架构：由2个stacked encoder和2个stacked decoder组成。

![img](./transformer_resideual_layer_norm_3.png)



## Decoder

现在来看看encoder和decoder是如何协作的：

- 第一个的encoder的输入式原始的embedding；
- 最后一个encoder输出是一个z向量，以及两个矩阵：keys matrix和value maxtrix



![img](./transformer_decoding_1.gif)



encoding阶段（phase）完成后，我们开始decoding阶段，decoding的每个step描述如下：

1. 每个step 输出一个element的概率分布。
2. 从step1的概率分布中抽样出word（或者直接选择概率最大的word），作为decoder的输入。
3. 重复step1，直到产生结束符。

   

decoder也会用到位置编码，下图的右半边

![img](./transformer_decoding_2.png)

decoder中的 self attention layers与encoder的运作方式略有不同:

在decoder中，self-attention layer只能接触到输出序列中的前半部分。 

怎么做到呢？在softmax操作之前，masking序列后半部分(设为 -inf)

"Encoder-Decoder Attention"层的工作原理和多头self-attention一样，只不过它的Queries矩阵来自下面的decoder的输出和stacked encoder中的Keys矩阵和Values矩阵 。

## 最后的dense层和Softmax层

stack decoders输出一个数值向量。 我们怎么把它变成一个词呢？

这是最后的dense层+softmax在做的事情。

dense层是一个简单的全连接神经网络，它将stack of decoders的输出映射为一个长多的logits向量（长度就是输出词汇表的大小）。

![img](./transformer_decoder_output_softmax.gif) 
这个图从底部开始，生成一个vector作为decoder stack的输出。 然后它被转换成一个输出单词。



## 训练

我们假设输出词汇表只包含六个单词("a"、"am"、"i"、"thanks"、"student"和"eos"("句子结束"的缩写))。

>  模型的输出词汇表是在开始训练之前的预处理阶段创建的。一旦定义了输出词汇表，就可以使用 one-hot 编码 来表示词汇表中的每个单词。

假设我们正在训练的目标是将"merci"翻译成"thanks"。

也就是说，当我们输入“merci”的embedding向量，希望模型输出的概率分布向量中单词"thanks"的概率值最大。

训练的过程：将模型输出与目标输出进行比较，然后使用反向传播方法调整模型的权重，使模型输出更接近目标输出。

如何比较两种概率分布？ 可查看 [cross-entropy](https://colah.github.io/posts/2015-09-Visual-Information/)和 [Kullback-Leibler](https://www.countbayesie.com/blog/2017/5/9/kullback-leibler-divergence-explained). 。

目标概率分布：

![img](./output_target_probability_distributions.png)

模型输出：

![img](./transformer_output_trained_model_probability_distributions.png)



# bert模型
![bert模型可视化](https://images.prismic.io/peltarionv2/e69c6ec6-50d9-43e9-96f0-a09bb338199f_BERT_model.png?auto=compress%2Cformat&rect=0%2C0%2C2668%2C3126&w=1980&h=2320)


## 参考

1. [Attention Is All You Need ](https://arxiv.org/abs/1706.03762) paper, the Transformer blog post ( ([Transformer: A Novel Neural Network Architecture for Language Understanding ](https://ai.googleblog.com/2017/08/transformer-novel-neural-network.html)), and the ) ，[Tensor2Tensor announcement](https://ai.googleblog.com/2017/06/accelerating-deep-learning-research.html).
2. Watch [Łukasz Kaiser’s talk  ](https://www.youtube.com/watch?v=rBCqOTEfxvg) walking through the model and its details 
3. Play with the [Jupyter Notebook provided as part of the Tensor2Tensor repo ](https://colab.research.google.com/github/tensorflow/tensor2tensor/blob/master/tensor2tensor/notebooks/hello_t2t.ipynb)
4. Explore the [Tensor2Tensor repo](https://github.com/tensorflow/tensor2tensor).
5. [Depthwise Separable Convolutions for Neural Machine Translation ](https://arxiv.org/abs/1706.03059)
6. [One Model To Learn Them All ](https://arxiv.org/abs/1706.05137)
7. [Discrete Autoencoders for Sequence Models ](https://arxiv.org/abs/1801.09797)
8. [Generating Wikipedia by Summarizing Long Sequences ](https://arxiv.org/abs/1801.10198)
9. [Image Transformer ](https://arxiv.org/abs/1802.05751)
10. [Training Tips for the Transformer Model ](https://arxiv.org/abs/1804.00247)
11. [Self-Attention with Relative Position Representations ](https://arxiv.org/abs/1803.02155)
12. [Fast Decoding in Sequence Models using Discrete Latent Variables ](https://arxiv.org/abs/1803.03382)
13. [Adafactor: Adaptive Learning Rates with Sublinear Memory Cost ](https://arxiv.org/abs/1804.04235)
