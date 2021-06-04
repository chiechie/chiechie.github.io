---
title: karpathy-GPT3的自述
author: chiechie
mathjax: true
date: 2021-03-29 23:05:40
tags: 
- Karpathy
- NLP
- sam altman
- 人工智能
- 人工智障
- GPT-3
categories: 
- 阅读
---

> 先说背景，sam altman在twitter上介绍了Karpathy的这篇文章。sam altman和karpaty都是elon musk的商业合作伙伴。sam和Elon创办了OpenAI，Karpathy是tesla的AILab的leader。GPT-3是OpenAI的代表作品之一。
> 
> 看到twitter的第一反应是，两个有共同商业利益的人在互捧。但是两个人都是各自领域的大佬，所以想一探究竟，AI界是不是有了什么新的进展？
> 
> btw，最近陷入比较深的「反智」状态，觉得搞ai的都是大忽悠，有点上头


## 总结

1. 我对GPT-3不是很了解，很多细节没有看懂，后面看懂了再补吧。
这边文章是karpathy以GPT-3的口吻，描述了关于自己在图灵测试上的表现，还解释了原因。
可能是想号召大家不要过分关注gpt3弱智的地方，而是多关注它有用的地方吧。
2. 让gpt3显得很弱智的任务长这样：
```
   Q: How many eyes does my foot have?
   A: Your 
```
karpathy的解释是，they（people） optimize for frequency but expect correctness. And *they* make fun of me。翻译下，我学习的目标是单词频率，但是你现在却要求我回答正确。你们人类就是在玩弄我。
3. 这篇文章目的，在于给民众洗脑，塑造对ai能力的正确期望，不要有不切实际的幻想，比如让ai回答脑经急转弯题，又重述了在某些领域的价值，而关键在于定义合适的problem，然后让机器来解题。这是是一篇值得学习的ai公关文。后面跟客户或者跟领导沟通的时候，可以拿来作为教材。
4. 有个评论把这个现象拓展到人际交往中：每个人都有自己的目标函数，都是根据自己的目标去迭代地优化。但是局外人经常用自己定义的评估指标去评判这个人的水平，还经常武断地觉得这个人疯了。这很荒谬，局外人压根不知道人家的努力目标是什么。btw，大部分人都不知道自己在做啥，所以，大部分人的评价指标都不足以放在心上。朝着自己的objective继续迭代吧。
   
    > Wow! I have the feeling that we are all in a feedback loop to optimize something and when we say someone is acting irrational, we just don't get what the person is optimizing for.



## 参考
1. [karpathy-forward-pass](https://karpathy.github.io/2021/03/27/forward-pass/)
2. [王川: 关于 GPT-3 的随想 （一）](https://mp.weixin.qq.com/s?__biz=MzA3MzE5MjM2Mw==&mid=2672247401&idx=1&sn=7e8661ce3aeb407f2af6e87963890606&chksm=85a124adb2d6adbb6f4d5b59dded3c2d5fe5376c7cf35104ab60df12f3fcb1a5a4d85b9ebaee&scene=21#wechat_redirect)