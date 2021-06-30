---
title: 关于Scale AI
author: chiechie
mathjax: true
date: 2021-06-16 10:39:20
tags:
- 投资
- 行业研究
- 人工智能
categories: 
- 投资
---

> 看到Packy激情满满写了一篇关于Scale AI的公关文，比较好奇这个年轻的小公司有何特别之处，为什么会受到资本的一致认可。
>
> 印象中Scale AI就是一家硅谷的数据标注公司，主要靠外包给第三方国家，赚取中间费用，这种公司有什么竞争力呢？跟大厂比起来？

## 总结
1. Scale AI创立至今5年，最近一次估值73亿$。它跟stripe有点像，在一个蓬勃发展的行业，找到一个关键但是不起眼的小地方，深耕细作。
2. 刚开始成立的时候，Scale AI专注做数据标注，但是他们的野心不限于此，他们想做更好用的数据标注工具，即人机互助的方式，Human-in-the-Loop（HIL）。 
3. Scale AI的故事是：随着标记数据越来越多，标注数据的边际成本会降低，因为标记样本越多，标记机器人的性能越好，能hold的事情就越多，需要分配出去的工作就越少。
   
    > 这里有一个问题，怎么确定，哪些是机器可以搞定的，哪些是机器搞不定的？这个是不是又要多一层人工监督了？
    >  
    > 可以从环境中获取到反馈，也可以人工抽查。
    >  
    > 总的来说，都不是很完善的方案。
4. Scale AI还有一个亮点，他的客户群体很分散，从国防部到无人驾驶公司，eg OpenAI, Airbnb and Lyft。这有一个好处就是。AI公司去去留留，竞争之后，留下了最有效的ml应用场景，但是Scale AI始终有机会，因为是做基础设施的。
8. 除了标注API，Scale AI还做了一个样本调试的产品--Nucleus，data debugging SaaS product。目的是做全链路基建嘛。
![Nucleus](./img.png)
9. Scale AI的CEO Alexandr Wang 说的一段话

    > At Scale, we’re building the foundation to enable organizations to manage the entire AI lifecycle. Whether they have an AI team in-house or need a fully managed models-as-a-service approach, we partner with our customers to build their strategy from the ground up and ensure they have the infrastructure in place to systematically deliver highly-performant models.

简单点说，他们的方法是，帮助企业从0到1落地一个AI应用--不管这个企业内部有没有AI团队--企业能够使用这个基建更高效地构建模型. 就是win-win,后生可畏.

10. 对于没有AI团队的传统企业，Scale AI定制化合作（跟必示一样），客户例如Brex和Flexport。
    
    ![img.png](./img1.png)
11. 这里有一个小故事：Brex doesn’t have an AI team working on the model; they outsource it to Scale.
.Brex一开始找的是专门做OCR的大公司，帮他们做发票文本提取，但是效果一般（“they were all mediocre.” ）。后面找到scale，通过深入合作定制化了一个模型，效果100%。后面沉淀出了一个产品document。
12. scale的产品还有蛮多的：地图/文档/图像。。
    ![img.png](img2.png)

> btw, 知乎上有一个相关提问--如何评价Scale AI？大家似乎认识非常有限（只知道它是做数据标注的），还讨论的热火朝天。由此可见的，噪声膨胀的速度远远超过信息增长的速度。


## chiechie's reflection
1. packy这篇文章的公司分析框架蛮好的，后面分析科技公司可以套用：

    - 行业介绍：The State of AI and ML
    - 公司介绍：Getting to Scale. 
    - 相同模式的成功案例对比：Scaling Like Stripe.
    - 关于公司的负面观点：The Bear Case for Scale.
    - 关于公司的正面观点：The Bull Case for Scale.
    - 前景展望：Scale’s Compounding Vision.
2. AI市场未来还有这么大的成长空间，当前只有8%的公司应用了AI技术,看到这个数字有点吃惊。但是，即便如此，AI从业人员的需求也不用过分乐观估计。
3. AI应用目前还在探索期。Scale AI比较鸡贼，让AI公司在前面探路，让他们相互pk，把有价值可落地的场景摸索清楚，自己在后面提供军火库，妥妥的赢家。

## 参考
1. [not boring](https://www.notboring.co/p/scale-rational-in-the-fullness-of)
2. [吴恩达发起新型竞赛范式！模型固定，只调数据？](https://zhuanlan.zhihu.com/p/384012257)
3. [Scale AI’s Series E: Deploying AI Across Every Industry](https://scale.com/blog/series-e)
4. [nucleus](https://dashboard.scale.com/nucleus/)