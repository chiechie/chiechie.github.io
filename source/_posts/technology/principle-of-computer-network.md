---
title: 计算机网络基本原理
author: chiechie
date: 2021-02-28 20:19:10
categories: 技术类

tags:
- 计算机网络
---

什么是协议（protocol）？
一堆定义好的标准集合没，每台电脑都要遵守，才能够正常通信

什么是计算机网络？
覆盖所有相关事物：电脑之间如何通信
网络的作用是，保证每台电脑都能倾听到彼此，并且每台电脑说的话，其他电脑也能听得懂

什么是TCP/IP模型？
![图1-TCP/IP网络模型](img_1.png)
是一个5层的模型 ，每一层都有自己的协议，除了TCP还有，其他的模型如OSI。

为什么要分层？
为了让职能分化（specialization）？让不同的人做不同的事？

TCP/IP模型中包含哪些东西？
- 物理层：即一些物理设备，跟电脑相互联系的，是通过网线（cable）连接的。物理层的几个核心元素是：cable，connectors，发送信号

![图2-物理层-使用网线将物理设备连接起来](img_2.png)
- 数据层：也叫网络接口层（networklayer），这里是第一次涉及到协议（才开始跟内容有关，物理层仅跟cable有关）
    数据连接层，负责定义如何解读这些信号。这一步之后，两边才能交流（communicate，不然就是鸡同鸭讲）。数据连接层有很多协议，最出名的叫"以太网"，尽管现在无线技术越来越popular，
  在物理层之上，以太网标准定义了一个协议，负责从同一个网络的一个节点获取数据，accross a single link。

- 网络层，也叫internetwork层
    允许不用的网络进行通信，当然，依靠的是路由器
  这里涉及到一个的概念
  internetwork ：一系列的网络，通过路由器，彼此相连接。
  最有名的internetwork叫Internet。
  这一层常用的协议叫：IP即 Internet Protocol
  IP是Internet和大部分更小的网络的核心，他的作用是将数据从一个节点传给另一个节点。
  network software通常被分类为：client 和 server
  client应用初始化一个request， 并且通过netwrok发送给server应用 ，
  server将response通过network发送回来
 
  
- 传输层，邮件的传输格式和网页的传输格式不一样，这个区别就是在这一层定义的。
  会议一下，
  网络层在两个node之间传输数据， 一个node，可以运行多个client应用或者server应用，
  传输层sort out哪一个client和server 应该去获取数据。
    第四层传输层用最常见的协议--TCP，Transmission Control Protocol。
  虽然TCP和IP经常被放到一起说，但是要注意，TCP和IP网络模型的不同层的协议。
  除了TCP，其他的传输层协议，也会利用IP先将数据送达，
  例如UDP，user datagram protocol
  UDP和TCP的区别在于，TCP机制可确保数据安全被送达，但是UDP并不要求。
  
    TCP和UDP是用来确保数据被一个node上正确的程序所接受。
    对比以下，IP只能保证数据传达到正确的node。
    类似送快递，IP可以保证包括从武汉商店送到深圳的腾讯大厦，
    TCP可以保证包裹 送至 对应的员工手中。
  
- 应用层，这一层的协议是跟具体的应用类型相关的
比如网页类型的应用要准手http协议
  邮件类型的应用要准讯smtp协议。

再用送快递打个比方
- 物理层，就是一个一运送快递的卡车和道路
- 数据连接层：卡车将包裹从一个
- 网络层：识别出 从A到B要走哪条路
- 传输层：告诉A中哪个程序来收包裹。
- 应用层：就是包裹的内容。


![图3-用送快递类比TCP/IP网络模型的运行机制](img_4.png)



## 参考资料
1. [The Bits and Bytes of Computer Networking](https://www.coursera.org/learn/computer-networking/home/welcome)
2. [西安交通大学公开课-计算机网络原理](https://open.163.com/newview/movie/free?pid=ME74DFHFC&mid=ME74E6NLA)