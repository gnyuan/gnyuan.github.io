---
layout: post
title: 程序化交易初探
tags: stock trade program 
categories: stock
---


# 概述

双十二这天并没有加入淘宝线下剁手节，而是参加了由深圳市金融信息服务协会举办的“**第七届中国（深圳）金融信息服务发展论坛**”，本次大会主题是信息安全以及风险管理。对这两个主题，甲方公司（如基金、券商）讲得一般般，乙方公司（深信服、联软什么的）PPT做得很有诚意，分享了这些领域他们公司的产品。

最让人印象深刻的是“**名策数据**”祝清讲的《程序化交易风险管理及技术实现》，据说他们团队是华尔街回来的，专门做程序化交易，为券商、基金等服务。他说，国内号称程序化交易，八成以上是假的.如果通过券商柜台系统报盘集中交易的，那铁定是假的。。

##程序化交易系统

其介绍的程序化交易系统（EMS系统，即Execution Management System）是以ZeroMQ为通信中间件，以ProtoBuf为通信报文载体，以FIX Trading Community为标准协议，以成熟的CEP事件驱动库位框架骨干，构建事件驱动的异步通信框架，支持高并发、低延迟的高速交易系统。

以下分享的是他的PPT(忽略那个入镜的小哥...)：
![1]({{ url.site }}/static/img/2015-12-13-1.jpeg)
![1]({{ url.site }}/static/img/2015-12-13-2.jpeg)
![1]({{ url.site }}/static/img/2015-12-13-3.jpeg)
![1]({{ url.site }}/static/img/2015-12-13-4.jpeg)
![1]({{ url.site }}/static/img/2015-12-13-5.jpeg)
![1]({{ url.site }}/static/img/2015-12-13-6.jpeg)
![1]({{ url.site }}/static/img/2015-12-13-7.jpeg)
![1]({{ url.site }}/static/img/2015-12-13-8.jpeg)
![1]({{ url.site }}/static/img/2015-12-13-9.jpeg)
![1]({{ url.site }}/static/img/2015-12-13-10.jpeg)
![1]({{ url.site }}/static/img/2015-12-13-11.jpeg)
![1]({{ url.site }}/static/img/2015-12-13-12.jpeg)
![1]({{ url.site }}/static/img/2015-12-13-13.jpeg)
![1]({{ url.site }}/static/img/2015-12-13-14.jpeg)


1. [ZeroMQ](https://github.com/zeromq/libzmq)，这是个开源的项目，与其说是中间件，不如说是通信库。设计好节点间的通信模式后，就能方便的用`REQ-REQ`,`PUB-SUB`等组合进行健壮通信设计。是值得深入研究的项目。
2. [ProtoBuf](https://github.com/google/protobuf),是Google开源的项目，从名字可以看出，他是协议持久化的技术。配合MQ使用的。也非常值得深入研究。
3. [FIX Trading Community](http://www.fixtradingcommunity.org/),是金融业的数据交换协议吧。
4. “成熟的CEP”事件驱动库，这个查了下，即（Complex Event Processing）,原来跟Stream Computting是一类的。就说，越新的事件价值越高，所以把事件处理作为核心，不落地到硬盘，而直接在内存中处理。它讲的成熟的CEP库，[WIKI](https://en.wikipedia.org/wiki/Complex_event_processing#In_financial_services)这个介绍没怎么清楚，不过他列出的几个参考资料不错。

**规则引擎和CEP**：
规则引擎：开源的方案包括Esper、Storm、Spark,商业方案包括APAMA/Streambase.[参考资料](http://wenku.baidu.com/view/02cb6cc1d5bbfd0a79567315.html?from=search)
CEP：[参考资料](http://www.yeeach.com/post/1052)

##程序化交易风险管理

如PPT所讲的，程序化交易涉及到多个实时风险点，包括程序化交易本身（如收益率，买卖时机）及来自监管层的限制（如撤单率、对敲禁止。P.S. 他还不忘黑一下Exchange，15笔每毫秒是其峰值，不可突破其限制）


