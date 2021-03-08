---
title: 神经网络红的layer如何设置初始化权重？
author: chiechie
mathjax: true
date: 2021-03-08 10:17:31
tags:
- 深度学习
- low level
categories: 技术类
  
---



## keras的实现

```python
keras.layers.Dense(10, activation="relu", kernel_initializer="he_normal")
```


![3种初始化方法](img.png) 


## 参考
1. Hands-On Machine Learning with Scikit-Learn and TensorFlow, P334