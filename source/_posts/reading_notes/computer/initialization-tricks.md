---
title: 神经网络训练的一些小tricks
author: chiechie
mathjax: true
date: 2021-03-08 10:17:31
tags:
- 深度学习
- low level
- 最佳实践
categories: 
- 阅读
---


## 神经网络的layer如何设置初始化权重？

```python
keras.layers.Dense(10, activation="relu", kernel_initializer="he_normal")
```

![3种初始化方法](../../AI/machine_learning/deeplearning/dl-framework/img.png) 


## 训练过程太长，能否实时保存训练结果？

```python
# 1. 设置loss，优化算法
model.compile(loss=..., optimizer=...,
              metrics=['accuracy'])

EPOCHS = 10
checkpoint_filepath = '/tmp/checkpoint'

# 2. 定义回调函数
model_checkpoint_callback = tf.keras.callbacks.ModelCheckpoint(
    filepath=checkpoint_filepath,
    save_weights_only=True,
    monitor='val_accuracy',
    mode='max',
    save_best_only=True)

# Model weights are saved at the end of every epoch, if it's the best seen
# so far.
# 3.在fit中传入回调函数，模型会一边训练一边存储
model.fit(epochs=EPOCHS, callbacks=[model_checkpoint_callback])

# 4. 从缓存路径中加载模型
# The model weights (that are considered the best) are loaded into the model.
model.load_weights(checkpoint_filepath)
```




## 参考
1. Hands-On Machine Learning with Scikit-Learn and TensorFlow, P334
2. [keras-model_checkpoint-官网文档](https://keras.io/api/callbacks/model_checkpoint/)
3. [concatenate](https://keras.io/api/layers/merging_layers/concatenate/)
