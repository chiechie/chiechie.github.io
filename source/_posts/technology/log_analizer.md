---
title:  "日志监控怎么做？"
author: "chiechie"
header-style: text
categories: 技术类
date: 2021-02-19 20:05:20
mathjax: true

tags:
  - nlp
---


## 做项目之前的分析

最近要做一个nlp的项目，需求就是从一堆杂乱无章的日志中，发现一些规律。
即哪些日志其实是在说一个事情，可归为一类；
哪些日志是新奇的，没有出现过的。

有点像什么呢？就是给了一堆用户的数据，想对这些用户去做一个分群，本质是一个探索式的需求。
但是，有一些该场景特有的约束，和用户的画像维度一样，日志的画像维度也是其细分场景特有的，
比如，日志我们通常关注的是 时间，日志级别， 不关注的是具体的ip，网站，数量。
或者说，我们最先关注的不是这些微观的信息，而是最泛化的信息。

如果是顶层有异常就需要下钻到更细分的维度，即研究更细致的原因，是哪个ip，或者网站，数量导致的。这一步可以使用多维度根因定位的方法。


可以出一个这种方案：日志模版提取 + 多维度根因定位。
其中，日志模版提取，要运维去定义规则。

## 看看别人怎么做的
### logmine

日志模式有什么难点？有的日志格式很明确，但是不同来源的日志汇总到一起，格式就五花八门了。有没有什么方法对多个来源，并且从中提取出有效模式呢？有，就是在下--分布式计算，效果跟手动提pattern一样好。
![图1-层次化地提取日志模式](logmine_image-20210225214320632.png)

#### pattern是怎么定义的？
每个pattern有三类字段：fixed value， Variable 和 Wildcard
- 固定值（fixed value field ）：如www, httpd and INFO ，有明确含义的，固化的。
- 可变字段（A variable field）：如IP地址，邮箱，数字，日期，属于一个具体的类型，但是取值是可变的额。
- Wildcards ：任意的字段，are matched with values of all types。

#### 提取pattern的具体流程以及如何评估pattern？

![图2-日志分析流程](image-20210226000021042.png)
- step1. 将原始日志进行分词
- step2. 提取可变字段（variable field），模糊掉，也叫检查类型
- step3. 得到key-value格式
- step4. 将key排序
- step5. 取key的交集

![图3-评价当前pattern的信息含量](cost_function.png)
即在调超参
怎么评估一个抽象层级，即该level对应的所有pattern的好坏？从包含的信息量来衡量，一般来说wildcard个数越多，这个模版越没有什么信息。
具体来说，可以定义一个量化的指标--cost function，取值越大，提取出来的n个模式效果越差：

- ${Size}_i$: 第i个cluster包含的日志个数，
  
- ${WC}_i$: 第i个cluster中，wildcards个数
  
- ${Var}_i$:第i个cluster中，可变字段的个数。
  
- ${FV}_i$: 第i个cluster中，固定值字段的个数。

默认，重点看wildcards个数，个数越多，信息量越少。
对应的超参数配置是:a1,a2, a3 = 1,0,0

> 类似k-means聚类，对每个k，都可以计算cluster的松散度。 可以跨k进行对比。


#### 怎么评估这个方法？
关于评估：怎么评估聚类的准确率？怎么评估模式识别的准确率？
因为没有标签，所以两个算法都是跟baseline算法对比。（都有了baseline，还要你这个算法干嘛？）

怎么评估聚类的准确率？ 跟一个baseline算法，即OPTICS对比。提出了一个指标agreement score，即最大公共子集的比例。

怎么评估模式识别的准确率？ 跟一个baseline算法，即UPGMA对比。
- UPGMA算法：对一大堆日志生成摘要，输入一个cluster的原始日志，输出一个patten，找到了最好的order。
  
- 使用UPGMA的结果作为ground truth，来评估模式识别算法的准确率。
  
- 先聚类，然后让UPGMA算法和本文的算法来给每个cluster生成摘要，
  
- 然后比较两者是否一致：一个字段一个字段地比较。准确率，为，摘要中命中的字段的比率。

$\text { Total Accuracy }=\sum\limits_{i=1}^{\# \text { of clusters }}\left(A c c_{i} \times \text { Size }_{i}\right) \div \sum\limits_{i=1}^{\# \text { of clusters }} \text { Size }_{i}$



#### 自己的局限性
对于复杂的，毫无规则的原始日志，无能为力。

![图5-在毫无规律的日志上也束手无策.png](badcase.png)



## 参考资料
1. [logmine-paper](https://www.cs.unm.edu/~mueen/Papers/LogMine.pdf)
2. [logmine-pypi](https://pypi.org/project/logmine/)
3. [apach_2k.log](https://github.com/logpai/logparser/blob/master/logs/Apache/Apache_2k.log)
4. [硕士论文-模式识别在海量日志分析中的应用研究  "施佳奇"](https://www.ixueshu.com/h5/document/814a23b6b51168d40153bcb23ef479f1318947a18e7f9386.html)

