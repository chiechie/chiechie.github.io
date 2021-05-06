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


- 2.0主要功能和改进
    - TensorFlow 2.0 重点关注易用性，更新了以下主要内容：
        - 使用 Keras 和 eager 模式进行更新
        - 在任何平台上都可以进行稳健的模型部署
        - 性能更好的研究实验
        - 简化多种 API
- low-level API的灵活性支持： 针对那些想要突破ML边界的研究者，谷歌在 TensorFlow 的low-level API 上投入了大量精力
- 结合GPU提升性能：TensorFlow 2.0 在 GPU 上有很多性能改进。通过几行代码，并利用 Volta 和图灵 GPU 上的混合精度，TensorFlow 2.0 的训练性能最高提升 3 倍。TensorFlow 2.0 高度集成 TensorRT，并在谷歌云的英伟达 T4 云 GPU 的推理过程中通过改进的 API 实现更好的使用性和高性能。
- 丰富的标准数据：TensorFlow 中构建模型至关重要的一点是对训练和验证数据的有效访问。因此，谷歌推出了 TensorFlow Datasets，从而为包含图像、文本、视频等各类数据的众多数据集提供一个标准访问界面。TensorFlow Datasets 地址：https://www.tensorflow.org/guide/data
- 支持Eager Execution开发模式：tf.function 装饰器可用于将代码转化为图，从而可以实现序列化和性能优化。这得益于 Autograph 的补充，它可以将常规的 Python 控制流直接转化为 TensorFlow 控制流。
- 自动迁移脚本：为了消除用户对于从 1.x 迁移到 2.0 版本的顾虑，谷歌推出了一份迁移指南。2.0 版本同时也包含了自动转换脚本，帮助用户进行迁移。
- 在内部业务验证效果：开发团队和其他合作伙伴进行广泛的沟通。例如，TensorFlow2.0 帮助谷歌新闻部门部署了一个 BERT 模型，显著减少了内存占用。
- 多语言支持：同时，对于非 Python 语言的开发者而言，TensorFlow2.0 也提供了 TensorFlow.js ([https://www.tensorflow.org/js](https://link.zhihu.com/?target=https%3A//www.tensorflow.org/js))，官方表示 Swift 语言的版本也在开发中。
- 开发教程降低入门门槛：
    - deeplearning.ai 教程地址：[https://www.coursera.org/learn/introduction-tensorflow](https://link.zhihu.com/?target=https%3A//www.coursera.org/learn/introduction-tensorflow)
    - Udacity 教程地址：[https://cn.udacity.com/course/intro-to-tensorflow-for-deep-learning--ud187](https://link.zhihu.com/?target=https%3A//cn.udacity.com/course/intro-to-tensorflow-for-deep-learning--ud187)
