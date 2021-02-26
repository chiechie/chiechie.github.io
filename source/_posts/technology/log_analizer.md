---
title:  "日志分析"
author: "chiechie"
header-style: text
categories: 技术类
date: 2021-02-19 20:05:20

tags:
  - nlp
---


## 做项目之前的分析
最近要做一个nlp的项目，需求就是从一堆杂乱无章的日志文本中，发现一些规律，即哪些日志其实是在说一个事情，可归为一类；
哪些模式的新奇的，没有出现过的。

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

### high-level方式

每个pattern中，有三类字段：fixed value， Variable and Wildcard
-  固定值（fixed value field ）：有明确含义的，固化的，如www, httpd and INFO 。
-  可变字段（A variable field）：如IP地址，邮箱，数字，日期，属于一个具体的类型的。
- Wildcards ：任意的，are matched with values of all types

![image-20210225214320632](/images/logmine_image-20210225214320632.png)

### 具体的做法

![image-20210226000021042](/images/image-20210226000021042.png)

- step1. 将原始日志进行分词

- step2. 模糊化ip类信息，也叫检查类型，类型就是IP，网址

- step3. 得到k-v格式

- step4. 将k排序

- step5. 取key的交集


![dede.png](/images/image-20210225215133870.png)
### logmin的demo



## 参考资料
[logmine-paper](https://www.cs.unm.edu/~mueen/Papers/LogMine.pdf)
[logmine-pypi](https://pypi.org/project/logmine/)
[apach_2k.log](https://github.com/logpai/logparser/blob/master/logs/Apache/Apache_2k.log)
[硕士论文-模式识别在海量日志分析中的应用研究  "施佳奇"](https://www.ixueshu.com/h5/document/814a23b6b51168d40153bcb23ef479f1318947a18e7f9386.html)

