---
title: tf2.0的主要亮点
author: chiechie
mathjax: true
date: 2021-05-06 16:40:30
tags: 
- tensorflow
- 深度学习
- 人工智能
categories: 
- 技术
---

> tf还是很有野心的，灵活性和易用性都想要.

- TensorFlow 2.0重点关注易用性，更新了以下主要内容：
    - 使用 Keras 和 eager 模式进行更新
    - 在任何平台上都可以进行稳健的模型部署
    - 性能更好的研究实验
    - 简化多种 API
- low-level API的灵活性支持：想要突破ML边界的研究者，可以用tf2做更多的事
- 结合GPU提升性能：利用Volta和图灵GPU的混合精度，训练性能最高提升了3倍。
- 丰富的标准数据：[TensorFlow Datasets](https://www.tensorflow.org/guide/data)包含图像、文本、视频等各类数据,并对多种数据提供一个标准访问接口。
- 支持Eager Execution开发模式：Autograph可以将常规的Python 控制流直接转化为 TensorFlow 控制流，从而可以实现序列化和性能优化。
- 自动迁移脚本：2.0 版本同时也包含了自动转换脚本，帮助1.x的用户进行迁移。
- 在内部业务验证效果：开发团队和其他合作伙伴进行广泛的沟通。例如，TensorFlow2.0帮助谷歌新闻部门部署了一个BERT模型，显著减少了内存占用。
- 低门槛开发教程
    - [deeplearning.ai](https://link.zhihu.com/?target=https%3A//www.coursera.org/learn/introduction-tensorflow)
    - [Udacity 教程](https://link.zhihu.com/?target=https%3A//cn.udacity.com/course/intro-to-tensorflow-for-deep-learning--ud187)

