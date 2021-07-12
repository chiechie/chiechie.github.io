---
title: SVD和compression
author: chiechie
mathjax: true
date: 2021-07-10 11:40:31
tags: 
- SVD
- 信息论
- 效率
categories: 
- 阅读
---

> 大矩阵怎么存？最简单的方式就是矩阵分块, 然后分chunk存/读/算，下一个chunk覆盖上一个chunk，始终占用一个chunk的内存。
>
> 如果还想去噪，可以用svd，下面推演一下用svd可以节省多少空间

SVD技术可以应用于信号压缩

SVD是这样的, 随意一个矩阵A可以做如下分解

$$A = U\Lambda V^T$$==>$$U^T A = \Lambda V^T  $$

可以把$U^T$看成是encoding,  $U$看成是decoding, 压缩之后的信号是$\Lambda V^T$

![奇异值分解](./奇异值分解.png)

detail

假设$A \in R^{n*m}, U\in R^{n*n}, \Lambda \in R^{n*m}, V \in R^{m*m}$

$\Lambda$中的元素从大到小排列，前面k个元素绝对值大于0

对三个矩阵分块，图中的阴影部分是我们要存储的部分

$$\Lambda = [\Lambda_k, \mathbf{0_{m-k}}], \Lambda_k \in R^{n * k}, \Lambda_{(m-k)} \in R^{n * (m-k)}$$

$$V = [ \mathbf{v_k},  \mathbf{v_{m-k}}],  \mathbf{v_k} \in R^{m*k},  \mathbf{v_k} \in R^{m*(m-k)} $$

$$\Lambda V^T = [\Lambda_k, \mathbf{0_{m-k}}].[v_k, v_{m-k}]^T = \Lambda_k.v_k^T $$

经过U做encoding之后的信息仅有 $\Lambda_k$和$v_k$

其中$\Lambda_k$可以存为稀疏矩阵，含k个非零元素

$v_k \in R^{m*k}$占用空间m*k

 U做类似的分解占用空间n*k

一起需要存储的元素个数为$k + m*k + n*k$

不做encoding的话，需要存储m * n 个元素

总结一下:用了压缩算法之后，空间复杂度从O(n*m)降低到O(m)或O(n)

当m,n很大，又很容冗余信息（k很小），svd分解能大量降低矩阵存储空间









