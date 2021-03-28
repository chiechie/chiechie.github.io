---
title:  日志监控怎么做？
author: "chiechie"
header-style: text
categories: AIOps
date: 2021-02-19 20:05:20
mathjax: true

tags:
  - nlp
  - 日志分析

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

1. [关于logmine的实践--chiechie](https://github.com/chiechie/LogRobot)


## 参考资料
1. [logmine-paper](https://www.cs.unm.edu/~mueen/Papers/LogMine.pdf)
2. [logmine-pypi](https://pypi.org/project/logmine/)
3. [apache_2k.log](https://github.com/logpai/logparser/blob/master/logs/Apache/Apache_2k.log)
4. [硕士论文-模式识别在海量日志分析中的应用研究  "施佳奇"](https://www.ixueshu.com/h5/document/814a23b6b51168d40153bcb23ef479f1318947a18e7f9386.html)
5. [关于logmine的实践--chiechie-](https://github.com/chiechie/LogRobot)
