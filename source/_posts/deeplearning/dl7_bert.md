---
title: 深度学习7 Bert
author: chiechie
mathjax: true
date: 2021-03-14 11:50:30
tags: 
- NLP
- 神经网络
- 模型可视化
- Transformer
- Bert
- attention
categories:
- 深度学习
---

为什么有了word2vec还不够？还需要bert干嘛？因为一词多义，同样的词在不同的上下文中表达的意思不一样，而word2vec的表示是静态的。
所以我们需要一个能对上下文编码的动态的embedding方法，bert就应运而生了。


# 总结

- Bert也是是一个对词语生成词语向量的方法。
- Bert可以认为是一个预训练Transformer的encoder部分
- Bert的全称是Bidirectional Encoder Representations from Transformers，双向encoder表示
- Bert的目的是预训练transformer模型的encoder网络，
- Bert的学习目标是怎么设定的呢？bert用以下两个任务来预训练transformer中的encoder网络
    - 任务1-预测遮挡词（Predict Masked Words）： 随机遮挡上下文，让encoder根据上下文来预测被遮挡的单词，大概随机遮挡挡15%的单词
    - 任务2-预测下一个句子（Predict the Next Sentence）： 把两个句子放在一起，让encoder判断两句话是不是原文里面相邻的两句话，正样本摘自原文，负样本是随机选择50%。
- Bert学习的时候，把上面两个任务结合起来
- 假如有两个词被遮挡，就要训练三个任务，2个预测遮挡词任务，1个预测是否邻近的任务。前面两个是一个multi-class分类；后面一个是一个二分类。前两个任务的损失函数是cross entropy，第三个任务的损失函数是binary-entropy。
- 最终的目标函数，是上面三个损失函数的求和，把最终的目标函数关于模型参数求梯度，然后使用梯度下降来求参数。
- Bert的优点：不需要人工标注数据，训练数据可以从wiki/网页等，长度为2.5billion单词
- Bert可以利用海量数据训练一个超级大的模型
- Bert想法简单有效，计算大家大，bert有两个版本
  - base： 1.1yi参数, 16个tpu训练4days，不算调参数，该参数是公开的
  - large：2.35yi参数，64个tpu训练4days，不算调参数，该参数是公开的
- 想用transformer直接下载徐训练好的bert模型就好，拿到参数就可以对英文编码了
- RoBERTa建议只用masking，而且是动态masking,


# 附录

## bert模型原理

![bert模型可视化](https://images.prismic.io/peltarionv2/e69c6ec6-50d9-43e9-96f0-a09bb338199f_BERT_model.png?auto=compress%2Cformat&rect=0%2C0%2C2668%2C3126&w=1980&h=2320)

### 任务一--预测遮挡词

任务可以描述为：
输入： “The _____ sat on the mat”
输出： What is the masked word?

![img.png](./img.png)

如何学习？

- e：one-hot vector of the masked word “cat”.
- 𝐩: output probability distribution at the masked position.
- 损失函数Loss = CrossEntropy(𝐞, 𝐩 )
- • Performing one gradient descent to update the model parameters.

### 任务二-- Predict the Next Sentence

任务可以描述为：

• Given the sentence:
“calculus is a branch of math”.
• Is this the next sentence?
“it was developed by newton and leibniz”
可以表述为一个而分类问题
• 输入:
[CLS] “calculus is a branch of math”
[SEP] “it was developed by newton and leibniz” 
• Target: true

• [CLS] is a token for classification.
• [SEP] is for separating sentences.

学习过程

![img_1.png](./img_1.png)


### 结合两个任务

• Input:
“[CLS] calculus is a [MASK] of math
[SEP] it [MASK] developed by newton and leibniz”.

• Targets: true, “branch”, “was”.


## Bert实践【todo】

如何使用BERT做迁移学习（Transfer Learning）？

- (demo数据)[https://www.kaggle.com/c/fake-news-pair-classification-challenge/data]
- 预训练的中文bert模型：hugging face

## 要不要冻结BERT的部分参数？

Why not to freeze BERT？

That said, BERT is meant to be fine-tuned. The paper talks about the feature-based (i.e., frozen) vs fine-tuning approaches in more general terms, but the module doc spells it out clearly: "fine-tuning all parameters is the recommended practice." This gives the final parts of computing the pooled output a better shot at adapting to the features that matter most for the task at hand.

when you fine-tune BERT, you can choose whether to freeze the BERT layers or not. Do you want BERT to learn to embed the words in a slightly different way, based on your new data, or do you just want to learn to classify the texts in a new way (with the standard BERT embedding of the words)?

I wanted to use BertViz visualisation to see what effect the classification tuning had on the attention heads, so I did fine-tuning with the first 8 layers of BERT frozen and the remaining 4 layers unfrozen.

Some people suggest doing gradual unfreezing of the BERT layers, ie finetuning with BERT frozen, then finetuning a bit more with just one layer unfrozen, etc.





# 参考
1. Bahdanau, Cho, & Bengio. Neural machine translation by jointly learning to align and translate. In ICLR, 2015.
2. Cheng, Dong, & Lapata. Long Short-Term Memory-Networks for Machine Reading. In EMNLP, 2016.
3. Vaswani et al. Attention Is All You Need. In NIPS, 2017.
4. [BERT (预训练Transformer模型)](https://www.youtube.com/watch?v=UlC6AjQWao8&t=26s)
5. [RoBERTa](https://arxiv.org/pdf/1907.11692v1.pdf)
6. Devlin, Chang, Lee, and Toutanova. BERT: Pre-training of deep bidirectional transformers for language understanding. In ACL, 2019.
7. [attack_on_bert_transfer_learning_in_nlp-blog](https://leemeng.tw/attack_on_bert_transfer_learning_in_nlp.html)
8. [ROBERTA-pytorch](https://pytorch.org/hub/pytorch_fairseq_roberta/)
9. [huggingface](https://discuss.huggingface.co/t/fine-tune-bert-models/1554/2)
10. https://medium.com/swlh/painless-fine-tuning-of-bert-in-pytorch-b91c14912caa