---
title: "一年同行：我的TiDB社区之旅"
showSummary: true
summary: "加入 TiDB 社区快一年了，回顾一下这一年和 TiDB 有关的经历 👀"
layoutBackgroundBlur: true
layoutBackgroundHeaderSpace: false
date: 2024-07-31
tags: ["随笔闲谈"]
---


## 初识 TiDB

我是某云厂商toB业务的小研发。去年的时候国产化这个词在 ToB 领域特别火，也是在那个时候知道了 TiDB。

我是在 8月注册了社区，但是刚来也比较漫无目的，只是很单纯的看课程中心的视频，也不怎么看社区。

![tidb简介.png](img%2Ftidb%E7%AE%80%E4%BB%8B.png)

直到11月，社区有一个 [**TiDB 认证培训五周年**](https://asktug.com/t/topic/1015721/1)的活动。只要看过课程的就可以兑换，我一下来了兴致。然后领了 PTCP 的兑换码，整理了一下视频课程中的知识点，当时为了巩固还去看了 tidb, pd 和 tikv 的部分代码，感觉挺有意思的。

最后在考过 PTCA 之后，写了第一篇[专栏](https://tidb.net/blog/fe4f7b05)，把之前整理的一些入门的知识点发了出来。

![PCTA.jpeg](img%2FPCTA.jpeg)

## 渐入佳境

这个活动过后，我就开始经常看 TiDB 社区的一些活动动态并水水贴，不过对于解答问题的贴子到现在还是不太能回复 😶

再之后就是[文档挑战赛](https://asktug.com/t/topic/1019364)的活动，这个上手难度比较低，还可以白嫖一个 Contributor，索性就报名参加了。

![文档挑战赛.png](img%2F%E6%96%87%E6%A1%A3%E6%8C%91%E6%88%98%E8%B5%9B.png)

一开始其实只加了几个功能点的文档，之后自己又捡了几个。在做的时候发现在 vercel 上使用 TiDB Serverless 非常方便，而且5G以内免费！之后就自己在 vercel 部署了 Umami，主要用作自己博客网站的数据统计，持久化就接的 TiDB Serverless。这个真的好用，到现在还在一直用。

![vercel+umami.png](img%2Fvercel%2Bumami.png)

再之后因为到1月份，那个时候 大模型 和 AIOps 特别火，比如Coze，Dify 这种。当时就想搞一个类似文档助手的东西，又联想起之前做文档挑战赛和学习 TiDB 的时候。**TiDB 的文档真的很全**。所以就尝试在 Coze 简单地做了一个 TiDB 文档助手，又水了一篇[专栏](https://tidb.net/blog/ba5cdb19)。这篇还被发到了**Ti研Ti语**，真的挺开心的。🥳

等到春节假期放假回来没多久，4月初的时候，社区又有了[免费考证](https://tidb.net/blog/e1cc92b4)的活动，这次的目标主要是想考 PCSD。之后就是抽空看视频、水水社区帖子，直到5月通过了 PCSD。

![PCSD.jpeg](img%2FPCSD.jpeg)

## 意外之喜

因为一直在关注大模型相关的，4月份的时候还有一个消息就是 [TiDB Vector](https://asktug.com/t/topic/1024952) 的内测。我看到的第一时间就申请了，然后差不多等到考完 PCSD，我的内测申请也通过了。

![TiDB Vector 邮件.png](img%2FTiDB%20Vector%20%E9%82%AE%E4%BB%B6.png)

刚刚好当时在看 Dify，看到 TiDB Vector 支持了 [llama_index](https://github.com/run-llama/llama_index) 和 [langchain](https://github.com/langchain-ai/langchain)，但是还没有支持 Dify，心想这怎么能行 （又能水一个PR）！于是抽空在 [Dify 悄悄支持](https://github.com/langgenius/dify/pull/4588)了。顺便又双叒水了一篇[专栏](https://tidb.net/blog/d93ee4ac)。

![Dify TiDB Vector PR.png](img%2FDify%20TiDB%20Vector%20PR.png)

我本以为只是提交了一个 Feature，代码其实也并不是很多，自己写 python 尤其是 mock 测试用例也还比较生疏 （之前确实没怎么写过）。

但是这个 Feature 引来了不少的关注，TiDB Vector 的运营和产品同学也很快联系了我，确实有点受宠若惊。并且TiDB Vector 还主动和 Dify 那边的研发联系推进这个 Feature 的合入等等。突然受到了不小的关注。

这之后写的关于 TiDB Vector + Dify 的专栏也被转发到了 **Ti研Ti语**，那个时候功能已经合入，Dify 的新版本已经发了。除此之外，还收到了TiDB 送的小礼物 😇

那一刻突然觉得在TiDB社区活跃，给开源做贡献确实还挺有意义的。

同时也真的很感谢 TiDB 的运营。从我的描述来看基本每个月都会有最少一个活动，社区文化做的真的挺好。这些活动也能让我从一个边缘人，逐渐开始关注 TiDB 的一些动态等。

不过回到自己，我自己对 TiDB 的了解感觉还不是很深入，只是停留在使用层面。平时根本用不出bug，就懒得追源码。之前对 [TiDB Talent Plan](https://asktug.com/t/topic/1026636)感兴趣，不过时间比较紧就没参加orz

未来有机会的话，希望能多给 TiDB 找bug提PR！👀

最后的最后，借用 TiDB Vector 内测通过的邮件中的一句话：

> 合抱之木，生于毫末；九层之台，起于累土。
>