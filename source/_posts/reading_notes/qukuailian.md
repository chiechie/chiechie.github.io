---
title: 区块链：通往资产数字化之路
author: chiechie
mathjax: true
date: 2021-04-30 08:50:28
tags: 
- 数字货币
- 区块链
- 比特币
categories: 
- 阅读
---


## 基本概念

- 区块(block):一组交易的集合，标上了时间戳，并包含前个区块的指纹。区块头经过哈希计算生成工作量证明，从而验证所有交易的有效性。经过验证的区块将通过网络共识添加到主区块链中。
- 区块链(blockchain):有效区块的列表，每个区块均指向其前序区块，直到创世区块（genesisblock）。

## 引言

1. 比特币不仅是数字货币，他是构建信用网络的基础，可以在更广泛的领域应用。
2. 比特币不是一种货币，而是一个去中心化的信用网络。
3. 步入农耕社会的蚁群：切叶蚁实际上并不吃树叶，而是利用树叶来培育真菌，这些真菌是切叶蚁群体的主要食物来源。蚁群是一个种性社会，但是没有中央政权，群落中没有领导者。
4. 在没有中央集权，没有层级，没有复杂结构的社会中可以产生不可思议的复杂性，蚁群是一个很好的例子。
5. 比特币是一个复杂的信用网络，网络中单个节点仅仅遵循几个简单的数学规则，并且不需要很复杂或者信用机制。
6. 就像蚁群，比特币网络由很多歌遵循简单规则的节点组成，不需要中心协作却能完成各种不可思议的事情。


## 第一章 欢迎来到比特币的世界

### 什么是比特币

1. 比特币：构成「数字货币生态」的概念和技术组合。
2. 比特币的货币单位也叫「比特币」，用于存储和传递价值。
3. 比特币用户之间主要通过「比特币协议」在互联网上通信。（互联网协议有点像蚁群之间交换的化学味素--信息素）
4. 比特币可以在专业的货币交易所与其他货币进行兑换；也可以完成一切传统货币的使命-买卖商品，转账，发放贷款。
5. 比特币的挖矿机制使得央行的货币发行和清算机制得以去中心化。???
6. 「比特币协议」规范了去全网挖矿的行为，通过算法控制挖矿速度（全网10min挖到1个block）。协议同样规定，每隔思念，新btc的创建速度将减半，总量在2100万内，大概在2140年达到。长期来看，比特币是维持通货紧缩的。不过反过来看，他是的无法通过"印钞票"导致通货膨胀。
7. 比特币作为货币，只是比特币协议第一个应用，还有可能有更多的应用，比特币生态类似货币的互联网。

---
比特币之前的数字货币

1. 数字货币取得人们信任的两个前提:
    - 防伪：相信货币是真实的，不是伪造的
    - 防止双重支付
2. 为了防伪，纸币的发行者采用先进的印刷技术来印钞。「防双重支付」对纸币来说也不是问题：一张钞票不可能同时在两个地方出现。
3. 传统货币以电子的方式进行转账和存储的时候，如何防伪和防双重支付？央行对电子交易进行集中清算。央行为什么可以做到？因为他拥有货币流通的全局视角。
4. 数字货币没法参考纸币和电子货币，它利用密码学来保障用户财产的安全。「加密数字签名算法」可以解决双重支付的问题。
5. 早期（20世纪80年代），数字货币项目发行的货币，通常由国家法定货币或者贵金属进行背书。但是他们还是中心化的，利用中央清算机构定时处理所有交易，但是很容易成为政府担忧的目标，从而被政府起诉，然后就消失了
6. 为了应对反对势力的干扰（政府or黑客），需要引入去中心化的数字货币系统，like 比特币。
7. 比特币是密码学+分布式系统融合的产物，整个系统包括以下四个部分：

- 比特币协议：去中心化的点对点网络，类似互联网
- 区块链：公共交易账本，类似人人都是清算人员
- 分布式挖矿：去中心化的基于数学的货币发行体系，类似计算机充当央行制定发货币的预算
- 交易脚本：去中心化的交易验证系统。


### 比特币的历史

1. 08年，中本聪发表了一片paper《Bitcoin:A Peer-to-Peer ElectronicCash System》，
主要创新点在于，基于「工作量证明」算法，让分布式计算系统每10min组织一次全局"选举"，使网络形成对交易状态的共识（来～大家一起来对一下账本～），这个机制优雅地解决了双重支付问题，避免了一个货币能被多次消费。之前，双重支付一直是数字货币系统的弱点，不得不引入一个中央清算机构来完成交易清算。
    
    > 为什么需要中央清算机构？我想到一个场景，比如我从招商银行账户转钱到建设银行账户，正常情况下，是从招行扣余额，建行增加余额，但是如果没有清算机构盯着他们，他们俩是不是可以作弊呢？比如招行不扣余额，建行还是增加余额，或者招行还给自己增加余额，这俩银行岂不就是相当于在自己给自己发货币了？？不行，还是得要第三方机构盯着。
2. 比特币网络开始于09年，现在网络的计算能力已经超过了全世界最强大的超级计算机的处理能力。
3. 中本聪自2011年4月起从公众视野中消失,不管是中本聪还是任何其他人均无法对比特币系统进行控制，这个系统只依赖于完全透明的数学法则.
4. 中本聪的发明，也是对之前未能解决的拜占庭将军问题的一个实用的解决方案。拜占庭将军问题可以描述为，如何在一个不可靠且存在潜在背叛风险的网络中，交换信息并且达成共识（非常fit现在社会的状况，没有共识，分化严重）。中本聪的方案：在一个没有中央可信节点的情况下，利用「工作量证明」来达成共识。方案的除了可应用于货币领域，也可用于证明选举，彩票，资产注册，数字公正等活动的公正性。
    
    > 距离本书的出版已经过去了3年，3年期间发生了一些魔幻的事件：特朗普质疑美国总统选举造假，台湾的彩票造假风波，艺术收藏品采用数字货币NFT来注册资产...可能比特币的市场在慢慢成熟的路上

### 比特币的使用，用户，以及他们的故事

> 比特币代表的货币，本质上是人与人之间实现价值交换的语言. 下面使一些数字货币在现实生活中的应用

1. 低价值零售零售：从朋友处获得一些比特币，使用比特币从帕洛阿尔托的鲍勃咖啡屋购买一杯咖啡。
2. 高价值零售：旧金山一家画廊的老板，以比特币计价出售昂贵的画作。高价值零售商面临的51%共识攻击的风险。
3. 离案合同服务：比特币可以在外包、合同服务及国际汇款中使用。
4. 慈善捐款：用比特币进行跨币种和跨国界的筹款过程，以及利用开放账本实现慈善组织的透明化
5. 进出口：用于大型商业机构间基于物理货品的国际支付，无货币兑换成本，交易快速。
6. 挖矿

### 新手入门

1. 下载客户端：客户端的选择基于客户对资金的控制意图,完全客户端提供了最高级别的独立控制，但是带来了备份和安全方面的负担。web客户端(like Multibit软件)使用门槛低，但是由于安全和控制是与web服务提供方共享的，存在交易风险。
2. 比特币地址:like一个电子邮箱地址或者二维码，可以将这个地址分享给别人，别人就可以给这个地址转钱，转完后钱将直接进入你的钱包。
3. 获取比特币：货币交易所（like huobi）。与银行开户类似，你可能需要花上几天到几周的时间来开设使用这项服务的账户。他们要求你提供各种形式的身份证明以满足KYC（了解你的客户）和AML（反洗钱）等银行监管规定。一旦你拥有了一个比特币交易所账户，你就可以像使用代理账户买卖外币一样开始买卖比特币了。
   
   > Bitstamp（http://bitstamp.net）一个欧洲货币市场，它支持欧元（EUR）和美元（USD）等几种货币，通过电子转账的方式完成。
   > Coinbase（http://www.coinbase.com）一个位于美国的比特币钱包和交易平台，在这个平台上，商家和客户可以通过比特币进行交易。Coinbase允许用户通过自动清算系统（ACH系统）将交易所账户与美元支票账户相连接，使得比特币的买卖变得非常简单。
   > Bitcoin Charts（http://bitcoincharts.com/markets）是一个提供价格行情及大量货币交易所市场数据的网站，在这里你可以找到更完整的列表。
5. 发送和接收比特币:直到作为交易的价值接收方将该比特币地址发布到比特币账本（区块链）前，该地址只是无数可能的比特币“有效”地址的一部分。一旦它和交易相关联，它就变成了网络中已知地址的一部分，而爱丽丝也就可以在公共账本上查看它的余额了。


## 参考
1. http://bit.ly/mastering_bitcoin
2. https://weread.qq.com/web/reader/6c4321805e3c8a6c44811dck16732dc0161679091c5aeb1