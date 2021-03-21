---
title: 私网IP 和 公网IP 
author: chiechie
date: 2021-02-27 17:38:13
categories:
- 阅读
- 技术

tags:
- 计算机网络
---

## 私网IP 和 公网IP 分别是什么？

- 私网IP，也叫private ip address，或者内网IP。是一个局域网内部，用于识别彼此身份的一个id。
- 公网IP，也叫public ip address，或者外网IP。是在因特网中，用户识别设备身份的一个id。


## modem 和 路由器 有什么区别？

- 一台电脑如要跟其他电脑通信，需先hook up on 一个modem，然后modem给他分配一个IP，也就是外网ip。
- 路由器是modem的升级版。 路由有两个作用：一个是连接家里的设备，另外一个是连接外面的世界。
家里的多个设备组成了一个局域网，即使不跟外面通信，他们之间是可以通信的，通信的过程中如何识别彼此呢？
这就是路由器的第二个作用，给局域网中的每个设备分配一个IP--也就是内网IP，这个是局域网的成员才能读懂的代码。

  
## 端口转发是什么？

由于外网只知道内网设备的外网地址，如何能让外网访问内网某个设备的某个端口的服务呢？
一个办法就是将局域网中的所有设备和服务，先注册到路由器中，路由器就可以根据外网的请求的端口号码，将之转移到内网提供相应服务的设备上。
这个就是端口转发做的事情。

## 网络中的三种设备-集线器，交换机 和 路由器
集线器和交换机是用来创建一个个小网络（局域网），路由器将这些小网络连接起来，变成因特网。

- 集线器（hub）： 把内网中的网络设备连接起来，只能广播。
- 交换机（switch）：维护一个交换机表，记录了连接的网络设备的mac地址，支持网络设备间p2p通信
- 路由器（router）：网关（gateway），网络出入口


## 参考
1. [私有ip和公共ip，以及端口forward](https://www.youtube.com/watch?v=92b-jjBURkw&list=LL70j7MlBEzFqOk5aIHKzmxQ)
1. [How to Enable SSH in Ubuntu---Install openssh-server？](https://www.youtube.com/watch?v=92b-jjBURkw&list=LL70j7MlBEzFqOk5aIHKzmxQ)
2. [how-do-you-run-a-ssh-server-on-mac-os-x](https://superuser.com/questions/104929/how-do-you-run-a-ssh-server-on-mac-os-x)