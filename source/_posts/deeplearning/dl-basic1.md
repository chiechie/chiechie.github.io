---
title:  深度学习基础2 神经网络训练的一些小tricks
author: chiechie
mathjax: true
date: 2021-03-08 10:17:31
tags:
- 深度学习
- low level
- 最佳实践
categories: 
- 深度学习
---


### layer的输入输出

```python
model_m = Sequential()
model_m.add(Reshape((TIME_PERIODS, num_sensors),
                    input_shape=(input_shape,)))
model_m.add(Conv1D(100, 10, activation='relu',
                   input_shape=(TIME_PERIODS, num_sensors)))
model_m.add(Conv1D(100, 10, activation='relu'))
model_m.add(MaxPooling1D(3))
model_m.add(Conv1D(160, 10, activation='relu'))
model_m.add(Conv1D(160, 10, activation='relu'))
model_m.add(GlobalAveragePooling1D())
model_m.add(Dropout(0.5))
model_m.add(Dense(num_classes, activation='softmax'))
```

![Image for post](https://miro.medium.com/max/2073/1*Y117iNR_CnBtBh8MWVtUDg.png)

- Conv1D：　model_m.add(Conv1D(100, 10, activation='relu', input_shape=(TIME_PERIODS, num_sensors)))：
    - 输入：(timesteps,  num_series) 
    - 输出：(timesteps - kerner_size+1, num_kernels)
    - 参数个数：(kernel_size * num_sensors  + 1) * num_kernels
- MaxPooling1D: model_m.add(MaxPooling1D(3))
    - 输入：(timesteps,  num_series)
    - 输出：(timesteps//3,  num_series)
    - 备注：书的页数（channel）没变，但是每页的字数变成了1/3
    - ![Image for post](https://miro.medium.com/max/3058/1*W34PwVsbTm_3EbJozaWWdA.jpeg)
- GlobalAveragePooling1D：model_m.add(GlobalAveragePooling1D())
    - 输入：　(timesteps,  num_series) 或者（feature_values, feature_detectors）
    - 输出：　（１，num_series）或者（１, feature_detectors）
    - 备注： 书的页数（channel）没变，但是每页的字数变成了1
- Dropout: model_m.add(Dropout(0.5))： 形状不变
    输入：（１，num_series）
    输出：（１，num_series）
- LSTM：model.add(LSTM(units=128,  dropout=0.5, return_sequences=True, input_shape=input_shape))
    - 输入： (timestep, series)
    - 输出：(timestep, units)
- Dense： model_m.add(Dense(num_classes, activation='softmax'))
    - 输入：（１，num_series）
    - 输出：（１，num_classes　×　２）
- BatchNormalization：
    - 输入:  (timestep, series)
    - 输出：(timestep, series) 
    - 参数个数：４ * series (channel) 
        - $$y=\gamma\left(\frac{x-\mu(x)}{\sigma(x)}\right)+\beta $$


## layer如何设置初始化权重？

```python
keras.layers.Dense(10, activation="relu", kernel_initializer="he_normal")
```

![3种初始化方法](img.png) 


## 训练过程太长？

保存训练结果

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
