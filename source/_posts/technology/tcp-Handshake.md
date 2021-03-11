---
title: tcp3次握手和4次挥手
author: chiechie
categories: 技术类
date: 2021-03-10 13:12:08
tags:
- 计算机网络
- 有限状态机
- TCP
---

> 看一下原理介绍
- TCP-FIN-WAIT1-客户端视角： 「客户端」 发出关闭指令后， 到服务器响应，间隔的时间。
- TCP-CLOSE-WAIT-服务端视角：「服务端」从接受”关闭指令“，到执行完成，间隔的时间。
- TCP-FIN-WAIT2-客户端视角： 「服务器」从 响应命令 到 完成命令，间隔的时间。


![图1-握手原理](img.png)


## 参考
1. [TCP 握手和挥手图解（有限状态机）-CSDN博客](https://blog.csdn.net/xy010902100449/article/details/48274635)
