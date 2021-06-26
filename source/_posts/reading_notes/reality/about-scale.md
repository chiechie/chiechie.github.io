---
title: 关于Scale AI
author: chiechie
mathjax: true
date: 2021-06-26 10:39:20
tags:
- 投资
- 行业研究
- 人工智能
categories: 
- 投资
---

> 看到packy激情满满写了一篇关于Scale AI的公关文，比较好奇这个年轻的小公司有何特别之处，为什么会受到资本的一致认可。
>
> 印象中scale就是一家硅谷的数据标注公司，主要靠外包给第三方国家，赚取中间费用，这种公司有什么竞争力呢？跟大厂比起来？



1. Scale AI创立至今5年，最近一次估值73亿$。
2. Scale AI跟stripe有点像，在一个蓬勃发展的行业，找到一个关键但是不起眼的小地方，深耕细作。
3. AI这个市场未来20年还有很大的发展空间，当前只有8%的公司应用了AI技术。
4. 刚开始成立的时候，Scale AI专注做数据标注，但是他们的野心不限于此，他们想做更好用（可能是智能的方式）的数据标注工具。最理想的方式是让算法来标记，但是算法目前性能答不到，大部分时候都需要人参与（Human-in-the-Loop” ，HIL) 
5. Scale AI的故事是：随着标记数据越来越多，标注数据的边际成本会降低，因为标记样本越多，标记机器人的性能越好，能hold的事情就越多，需要分配出去的工作就越少。
   
    > 这里有一个问题，怎么确定，哪些是机器可以搞定的，哪些搞不定的？这个是不是又要多一层人工监督层了？
    >  
    > 可以从环境中获取到反馈，也可以人工抽查。
    >  
    > 总的来说，都不是很完善的方案。
7. Scale AI还有一个亮点，他的客户群体很分散，从国防部到无人驾驶，eg OpenAI, Airbnb and Lyft。这有一个好处就是。AI公司去去留留，竞争之后，留下的是最有效的ml应用场景，但是Scale AI始终有机会，因为他不参与竞争，而是给赢家提供基础设施的。Scale AI承接的业务越分散，最终成为该位置的巨头的概率就越大（活下来就是赢家）。
8. 除了标注API，Scale AI还做了一个样本调试的产品--Nucleus，data debugging SaaS product。目的是做全链路基建嘛。
![Nucleus](./img.png)
9. Alexandr Wang 说的一段话

    > At Scale, we’re building the foundation to enable organizations to manage the entire AI lifecycle. Whether they have an AI team in-house or need a fully managed models-as-a-service approach, we partner with our customers to build their strategy from the ground up and ensure they have the infrastructure in place to systematically deliver highly-performant models.

简单点说，他们的方法是，帮助企业从0到1落地一个AI应用--不管这个企业内部有没有AI团队--企业能够使用这个基建更高效地构建模型. 就是win-win,后生可畏,
10. 从刚开始的数据标注工具到后面管理建模过程，确保哪些没有内部AI团队的企业构建基于AI的产品，例如客户Brex和Flexport，他们使用的是Document产品，这个有点意思。Brex doesn’t have an AI team working on the model; they outsource it to Scale.

![img.png](./img.png)
11. 


## 参考
1. [Packy not boring](https://www.notboring.co/p/scale-rational-in-the-fullness-of)
2. [吴恩达发起新型竞赛范式！模型固定，只调数据？](https://zhuanlan.zhihu.com/p/384012257)
3. [Scale AI’s Series E: Deploying AI Across Every Industry](https://scale.com/blog/series-e)