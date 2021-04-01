---
title: karpathy-GPT3的自述
author: chiechie
mathjax: true
date: 2021-03-29 23:05:40
tags:
categories:
---

> karpathy的这篇文章是sam altman在twitter上介绍的。
> sam altman 和 karpaty 都跟elon musk是paterner。
> sam和elon创办了open ai，karpathy是tesla的ai实验室leader。
> GPT-3是deepai的代表模型之一。
> 第一反应是两个有共同商业利益的人在互捧。
> 但是两个人都是各自领域的大佬，（karpathy的强化学习的公开课讲的蛮好的）
> 所以想一探究竟，ai界有什么新的进展？
> btw，最近反陷入比较深的「反智」状态，觉得搞ai的都是大忽悠。


## 总结
1. 我对gpt3不是很了解，很多细节没有看懂，后面看懂了再补吧。
这边文章是karpathy以gpt3的口吻，描述了关于自己在图灵测试上的表现，还解释了原因。
可能是想号召大家不要过分关注gpt3弱智的地方，而是多关注它有用的地方吧。
2. 让gpt3显得很弱智的任务长这样：
```
   Q: How many eyes does my foot have?
   A: Your 
```
karpathy的解释是， they（peple） optimize for frequency but expect correctness. And *they* make fun of me。也就是说，我学习的目标是单词频率，但是你现在却要求我回答的正确率。你们人类就是在玩弄我。

3. 这篇文章目的，在于给民众洗脑，塑造对ai的能力的正确的期望，既消灭了不切实际的对ai的一厢情愿的幻想（不要让ai来回到脑经急转弯的题目），又重述了在某些领域的价值，而关键在于人去找对problem，然后让机器来。人是一篇值得学习的ai公关文。跟客户或者跟领导沟通的时候，也可以拿来教育用户。
   
4. 有个评论把这个现象拓展到人际交往中：每个人都是在按照自己的目标去迭代地优化自己的行为，但是局外人经常武断地觉得这个人疯了，原因在于局外人压根不知道这个人自己的努力目标是什么。btw，这个人很有可能受到身边的人的影响，而改变自己的objective，这个更可怕。
   
    > Wow! I have the feeling that we are all in a feedback loop to optimize something and when we say someone is acting irrational, we just don't get what the person is optimizing for.



## 参考
1. [karpathy](https://karpathy.github.io/2021/03/27/forward-pass/)