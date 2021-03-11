---
title: aiops挑战赛2021-demo方案
author: chiechie
mathjax: true
date: 2021-03-09 22:04:42
tags:
- AIOps
categories: AIOps
---



## 数据解析
比赛数据包括： 部署拓扑，指标数据，日志数据，调用链数据，故障标签

### 部署数据

![部署数据](bushu.png)


### 指标数据
指标数据包括业务指标 和 性能指标。
业务指标用于触发根因分析，性能指标用于局部的根因定位

每个指标数据，都需要附带的信息有：归属物理设备id，以及相应用id，（这样才能做机器的指标依赖的额故障定位）

#### 业务指标
kpi_0304.csv

- rr：单位%，系统响应率
- sr：单位%，业务成功率
- cnt：单位条，交易量
- mrt：单位ms，平均响应时间
- tc：交易码, 有11种应用,
    'ServiceTest1', 'ServiceTest10', 'ServiceTest11', 'ServiceTest2',
     'ServiceTest3', 'ServiceTest4', 'ServiceTest5', 'ServiceTest6',
      'ServiceTest7', 'ServiceTest8', 'ServiceTest9'
  
![业务指标](yewu.png)

#### 性能指标

每个应用的指标, 每个kpi会有一个对应的机器id（即cmdb_id）

- cmdb_id: 指标归属的物理设备, 每个物理设备上收集到的指标一般来说包括两种，「操作系统」性能指标以及「服务」性能指标。
- kpi_name: 指标归属的服务类型，比如归属于操作系统/java虚拟机/Redis/Tomcat/Mysql等

![性能指标](xingneng.png)

### 日志数据


日志数据有3个重要字段：

- cmdb_id： 日志是从哪台物理机器上产生
- log_name: 日志对应的组件名和日志类别, 相当于4个采集器，eg catalina/gc/localhost/localhost_access_log
- value: 日志内容, 里面的信息包括，日志级别，egINFO/WARNING/SEVERE，具体报错信息


目前只有Tomcat4台服务器有日志数据；并且根据采集的不同的组件，收集的日志。


### 调用链数据
调用链路数据来自：trace/trace_0304.csv

重要的字段有：

- cmdb_id: 即对应的服务是部署在哪个机器上的, 一次全局的trace_id 可能跨越几个设备
- parent_id：子调用的上一次调用
- trace_id：一次trace对应的全局id，一个trace包括多次子调用
- span_id： 本流程的Span ID，

![调用链](trace.png)

### 故障标签

故障类型可分为--'应用故障', '网络故障', '资源故障'

- 应用故障可分为2类：'JVM CPU负载高', 'JVM OOM Heap'
- 网络故障可分为两类：'网络延迟', '网络隔离'
- 资源故障可分为三类：CPU使用率爬升，内存使用率过高，磁盘IO写使用率过高

## demo方案

流程：

1. 对「业务指标」进行异常检测，异常则触发下面的诊断。
2. 调用链异常检测和根因定位:「业务」指标异常了，定位是哪个中间件或者服务出现了异常，以及根因的那个服务。
3. 对应机器的指标进行根因定位：定位到了调用链中的根因服务，然后对该服务所在的机器进行下钻分析，进行更细粒度的根因指标定位，使用到了机器和中间件的「性能指标」。


### 业务指标异常检测(略)

找出第一个故障发生时刻：

- 2021-03-04 08:39:00	- 2021-03-04 08:44:00	

### 调用链异常检测和根因定位

1. 找出异常的调用路径--trace_id
2. 找出对应的根因的子调用--span_id
3. 找到span_id的机器

![调用链异常检测和根因定位](trace_rca.png)


### 对定位到的机器进行指标根因分析

1. 搜集该机器的所有性能指标，进行异常检测
2. 对于异常的指标，根据历史数据中学习到的因果图，进行根因推断。

![指标根因分析](metric_rca.png)


### 对下 分析结果 和 标准答案

![对比](evaluate.png)



## 参考资料

1.[aiops挑战赛2021-官网](http://iops.ai/competition_detail/?competition_id=17&flag=1)