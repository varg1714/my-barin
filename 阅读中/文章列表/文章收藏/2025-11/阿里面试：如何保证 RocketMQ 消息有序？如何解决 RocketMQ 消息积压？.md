---
source: https://mp.weixin.qq.com/s/ofaQwaOobZ_5xqcor6ObhA
create: 2025-11-18 21:01
read: true
knowledge: true
knowledge-date: 2025-11-19
tags:
  - 消息队列
summary: "[[Rocket 与 Kafka的区别]]"
---
FSAC 未来超级架构师

架构师总动员  
实现架构转型，再无中年危机

## 尼恩说在前面

在 40 岁老架构师 尼恩的**读者交流群** (50+) 中，最近有小伙伴拿到了一线互联网企业如阿里、滴滴、极兔、有赞、希音、百度、网易、美团的面试资格，遇到很多很重要的面试题：

如何保证 RocketMQ 消息有序？如何解决 RocketMQ 消息积压？

最近有小伙伴在面试阿里，又遇到了相关的面试题。

小伙伴懵了，因为没有遇到过，所以支支吾吾的说了几句，面试官不满意，面试挂了。

所以，尼恩给大家做一下系统化、体系化的梳理，使得大家内力猛增，可以充分展示一下大家雄厚的 “技术肌肉”，**让面试官爱到 “不能自已、口水直流”**，然后实现”offer 直提”。

当然，这道面试题，以及参考答案，也会收入咱们的 《[尼恩 Java 面试宝典 PDF](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247497474&idx=1&sn=54a7b194a72162e9f13695443eabe186&scene=21#wechat_redirect)》V140 版本，供后面的小伙伴参考，提升大家的 3 高 架构、设计、开发水平。

最新《尼恩 架构笔记》《尼恩高并发三部曲》《尼恩 Java 面试宝典》的 PDF，请关注本公众号【技术自由圈】获取，后台回复：领电子书

## 本文目录

**- 尼恩说在前面**

**- 1、为什么需要消息有序**

**- 2、基本概念**

 - 2.1 全局顺序

 - 适用场景

 - 2.2 分区顺序

 - 适用场景

 - 2.3 对比

**- 3、如何保证消息顺序？**

**- 4、RocketMQ 有序消息实现原理**

**- 5、有序消息的缺陷**

 - 使用 RocketMQ 如何快速处理积压消息

 - 如何确定 RocketMQ 有大量的消息积压

 - 如何处理大量积压消息

**- 参考文献**

**- 说在最后**

**- 部分历史案例**

## 1、为什么需要消息有序

假设准备去银行存取款，对应两个异步短信消息，要保证先存后取：

*   M1 存钱
    
*   M2 取钱
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WNsTD9uyEHoHGC5yUlf7DOtT8ONtZvW09Amsyib9PVDV1u5Yk34ottOKyFpUm6vDUrG4tgmgQV5d7A/640?wx_fmt=png&from=appmsg#imgIndex=0)

而 MQ 默认发消息到不同 Q 显然是行不通的，会乱序。因此，需发往同一 Q，依赖队列的先进先出机制。

再例如

如果我们有个大数据系统，需要对业务系统的日志进行收集分析，这时候为了减少对业务系统的影响，通常都会通过 MQ 来做消息中转。而这时候，对消息的顺序就有一定的要求了。例如我们考虑下面这一系列操作：

1.  用户的积分默认是 0 分，而新注册用户设置为默认的 10 分。
    
2.  用户有奖励行为，积分 + 2 分
    
3.  用户有不正当行为，积分 - 3 分
    

这样一组操作，正常用户积分要变成 9 分。但是如果顺序乱了，这个结果就全部对不上。这时，就需要对这一组操作，保证消息都是有序的。

## 2、基本概念

有序消息，又叫顺序消息 (FIFO 消息)，指消息的消费顺序和产生顺序相同。

如订单的生成、付款、发货，这串消息必须按序处理。顺序消息又可分为：

### 2.1 全局顺序

一个 Topic 内所有的消息都发布到同一 Q，按 FIFO 顺序进行发布和消费：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WNsTD9uyEHoHGC5yUlf7DOtcejw9D57ppgia6hrBdq98fdX7MAXvTlIeqia4iaWcl5J4JoRyGbrsAkaQ/640?wx_fmt=png&from=appmsg#imgIndex=1)

#### 适用场景

性能要求不高，所有消息严格按照 FIFO 进行消息发布和消费的场景。

### 2.2 分区顺序

对于指定的一个 Topic，所有消息按`sharding key`进行区块 (queue) 分区，同一 Q 内的消息严格按 FIFO 发布和消费。

*   Sharding key 是顺序消息中用来区分不同分区的关键字段，和普通消息的 Key 完全不同。
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WNsTD9uyEHoHGC5yUlf7DOtDib41wdXfjgjib5aiaYRVjFAlFAWnyygYTZo7FoaSWMMGsn3GszAjwwfg/640?wx_fmt=png&from=appmsg#imgIndex=2)

#### 适用场景

性能要求高，根据消息中的 sharding key 去决定消息发送到哪个 queue。

### 2.3 对比

*   发送方式对比
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WNsTD9uyEHoHGC5yUlf7DOt3AEA30FMQuJzNsqaN9Go793xQSrojhIWFLtafXA3ZJibCHVHTiaTccgw/640?wx_fmt=png&from=appmsg#imgIndex=3)

  

## 3、如何保证消息顺序？

在 MQ 模型中，顺序需由 3 个阶段去保障

1.  消息被发送时保持顺序
    
2.  消息被存储时保持和发送的顺序一致
    
3.  消息被消费时保持和存储的顺序一致
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WNsTD9uyEHoHGC5yUlf7DOtK8kYo4JQ1yXSiaaNgLPnQVVt9uCOZ2J3OLhht17EVBYy60f6sfZHDvw/640?wx_fmt=png&from=appmsg#imgIndex=4)

### 如何保证消息有序

MQ 的顺序问题分为全局有序和局部有序

*   **全局有序**：整个 MQ 系统的所有消息严格按照队列先入先出顺序进行消费
    
*   **局部有序**：只保证一部分关键信息的消费顺序
    

首先，我们需要分析下这个问题，在通常的业务场景中，全局有序和局部有序哪个更重要？其实大部分的 MQ 业务场景，我们只需要保证局部有序就可以了。例如我们用 QQ 聊天，只需要保证一个聊天窗口里的消息有序就可以了。而对于电商订单场景，也只要保证一个订单的所有消息是有序的就可以了。至于全局的消息的顺序，并不会太关心。而通常意义下，全局有序都可以压缩程局部有序的问题。例如以前我们常用的聊天室，就是一个典型的需要保证消息全局有序的场景。但是这种场景，通常可以压缩成只有一个聊天窗口的 QQ 来理解。即整个系统只有一个聊天通道，这样就可以用 QQ 那种保证一个聊天窗口消息有序的方式来保证整个系统的全局消息有序。

然后，落地到 RocketMQ。通常情况下，发送者发送消息时，会通过 MessageQueue 轮询的方式保证消息尽量均匀分布到所有的 MessageQueue 上，而消费者也就同样需要从多个 MessageQueue 上消费消息。而 MessageQueue 是 RocketMQ 存储消息的最小单元，他们之间的消息都是互相隔离的，在这种情况下，是无法保证消息全局有序的。

而对于局部有序的要求，只需要将有序的一组消息都存入同一个 MessageQueue 里，这样 MessageQueue 的 FIFO 设计天生就可以保证这一组消息的有序。RocketMQ 中，可以在发送者发送消息时指定一个 MessageSelector 对象，让这个对象来决定消息发入哪一个 MessageQueue。这样就可以保证一组有序的消息能够发到同一个 MessageQueue 里。

另外，通常所谓的保证 Topic 全局消息有序的方式，就是将 Topic 配置成只有一个 MessageQueue 队列（默认是 4 个）。这样天生就能保证消息全局有序了。这个说法其实就是我们将聊天室场景压缩只有一个聊天窗口的 QQ 一样的理解方式。而这种方式对整个 Topic 的消息吞吐影响是非常大的，如果这样用，基本就没有用 MQ 的必要了。

## 4、RocketMQ 有序消息实现原理

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WNsTD9uyEHoHGC5yUlf7DOtUraHJiagoPtk4JMRo4MlQZRPgoqWowWariaWVgbSQgMCR2uVshC519lA/640?wx_fmt=png&from=appmsg#imgIndex=5)

RocketMQ 消费端有两种类型：

*   MQPullConsumer
    
*   MQPushConsumer
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WNsTD9uyEHoHGC5yUlf7DOtX38giaYyJ8IbK9hvzHoVMZvJTaicThug5JgicDL9mSSk4dUR9e9MO8pWg/640?wx_fmt=png&from=appmsg#imgIndex=6)

底层都是通过 pull 机制实现，pushConsumer 是一种 API 封装而已。

*   `MQPullConsumer` 由用户控制线程，主动从服务端获取消息，每次获取到的是一个`MessageQueue`中的消息。
    

*   `PullResult`中的 `List<MessageExt> msgFoundList`
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WNsTD9uyEHoHGC5yUlf7DOtOZYdMD9BhpRwRQ79wzicnLgtSS0eiaibFXJcc0W10iboVDZrPrASZI3IjQ/640?wx_fmt=png&from=appmsg#imgIndex=7)

*   `MQPushConsumer`由用户注册`MessageListener`来消费消息，在客户端中需要保证调用`MessageListener`时消息的顺序性
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WNsTD9uyEHoHGC5yUlf7DOtG4qiamWk5JUUrcZq6NU12YicGJVibOcmTGiaDdgao278zZWVTceF62LdNg/640?wx_fmt=png&from=appmsg#imgIndex=8)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WNsTD9uyEHoHGC5yUlf7DOtU0jkgiczV9RZ7KC2k4JS6Q3xPJEny0GlIHnfunQjPuwc8Gia0mVjhbFw/640?wx_fmt=png&from=appmsg#imgIndex=9)

看源码

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WNsTD9uyEHoHGC5yUlf7DOtZ1W3uPmeufPiaU96tqecDaOrTRtb9sHAXpPhmrxdJlyXoETsFHcanMg/640?wx_fmt=png&from=appmsg#imgIndex=10)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WNsTD9uyEHoHGC5yUlf7DOt98PeEAAFv341lDMzb2TwiaycDHVmt6n2Hvpv8ZssnaianpouwkaAtWQw/640?wx_fmt=png&from=appmsg#imgIndex=11)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WNsTD9uyEHoHGC5yUlf7DOtmP3X4teGTjbBrVHK3hgE4DF4icIJyjMFFQOibYcHZF487WXYTb6Y9frw/640?wx_fmt=png&from=appmsg#imgIndex=12)

*   拉取生产端消息
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WNsTD9uyEHoHGC5yUlf7DOtwFL9fXM3MLoBKSsJ5n5kdiczBWUJbwgv0JZSthicAQScwkE1ItosRL3w/640?wx_fmt=png&from=appmsg#imgIndex=13)

*   判断是并发的还是有序的, 对应不同服务实现类
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WNsTD9uyEHoHGC5yUlf7DOtQeO38VdaYMkicyGz6mXkqo3Tsibic12qOvPEGa55TGrqYkX5136kcicicKA/640?wx_fmt=png&from=appmsg#imgIndex=14)

  

## 5、有序消息的缺陷

发送顺序消息无法利用集群的 Failover 特性，因为不能更换 MessageQueue 进行重试。

因为发送的路由策略导致的热点问题，可能某一些 MessageQueue 的数据量特别大

*   消费的并行读依赖于 queue 数量
    
*   消费失败时无法跳过
    

  

### 使用 RocketMQ 如何快速处理积压消息

#### 如何确定 RocketMQ 有大量的消息积压

在正常情况下，使用 MQ 都会要尽量保证他的消息生产速度和消费速度整体上是平衡的，但是如果部分消费者系统出现故障，就会造成大量的消息积累。这类问题通常在实际工作中会出现的比较隐蔽。

例如某一天一个数据库突然挂了，大家大概率就会集中处理数据库的问题。

等好不容易把数据库恢复过来了，这时基于数据库服务的消费者程序就会积累大量的消息。或者网络波动等情况，也会导致消息大量的积累。这在一些大型的互联网项目中，消息积压的速度是相当恐怖的。所以消息积压是个需要时刻关注的问题。

对于消息积压，如果是 RocketMQ 或者 kafka 还好，他们的消息积压不会对性能造成很大的影响。

而如果是 RabbitMQ 的话，那就惨了，大量的消息积压可以瞬间造成性能曲线下滑。

对于 RocketMQ 来说，有个最简单的方式来确定消息是否有积压。那就是使用 web 控制台，就能直接看到消息的积压情况。

在 web 控制台的主题页面，可以通过 Consumer 管理按钮实时看到消息的积压情况。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/xlgvgPaib7WNsTD9uyEHoHGC5yUlf7DOt79rqytNykpZpwoiaWAKPWUVa4q15lEC108nbgIMl6bwe8dArDdy1ibtw/640?wx_fmt=other&from=appmsg#imgIndex=15)

另外，也可以通过 mqadmin 指令在后台检查各个 Topic 的消息延迟情况。

还有 RocketMQ 也会在他的 ${storePathRootDir}/config 目录下落地一系列的 json 文件，也可以用来跟踪消息积压情况。

#### 如何处理大量积压消息

其实我们回顾下 RocketMQ 的负载均衡的内容就不难想到解决方案。

如果 Topic 下的 MessageQueue 配置的是足够多的，那每个 Consumer 实际上会分配多个 MessageQueue 来进行消费。

这个时候，就可以简单的通过增加 Consumer 的节点个数设置成跟 MessageQueue 的个数相同，但是如果此时再继续增加 Consumer 的服务节点就没有用了。

而如果 Topic 下的 MessageQueue 配置的不够多的话，那就不能用上面这种增加 Consumer 节点个数的方法了。这时怎么办呢？

这时如果要快速处理积压的消息，可以创建一个新的 Topic，配置足够多的 MessageQueue。

然后把所有消费者节点的目标 Topic 转向新的 Topic，并紧急上线一组新的消费者，只负责转储，就是消费老 Topic 中的积压消息，并转储到新的 Topic 中，这个速度是可以很快的。

然后在新的 Topic 上，就可以通过增加消费者个数来提高消费速度了。之后再根据情况恢复成正常情况。

在官网中，还分析了一个特殊情况。就是如果 RocketMQ 原来是采用的普通方式搭建主从架构，而现在想要中途改为使用 Dledger 高可用集群，这时候如果不想历史消息丢失，就需要先将消息进行对齐，也就是要消费者先把所有的消息都消费完，再来切换主从架构。

因为 Dledger 集群会接管 RocketMQ 原有的 CommitLog 日志，所以切换主从架构时，如果有消息没有消费完，这些消息是存在旧的 CommitLog 中的，就无法再进行消费了。这个场景下也是需要尽快的处理掉积压的消息。

## 参考文献

https://juejin.cn/post/7041137041593237541

## 说在最后

RocketMQ 面试题，是非常常见的面试题。

以上的内容，如果大家能对答如流，如数家珍，基本上 面试官会被你 震惊到、吸引到。

在面试之前，建议大家系统化的刷一波 5000 页《[尼恩 Java 面试宝典 PDF](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247497474&idx=1&sn=54a7b194a72162e9f13695443eabe186&scene=21#wechat_redirect)》，并且在刷题过程中，如果有啥问题，大家可以来 找 40 岁老架构师尼恩交流。

最终，**让面试官爱到 “不能自已、口水直流”**。offer， 也就来了。

## 部分历史案例

*   [年底大裁员......8 年以上小伙伴，机会在哪里？](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485914&idx=1&sn=7832d0b63155f12eb8036321a99991c8&scene=21#wechat_redirect)
    
*   [被裁后，反涨 30%，吊炸天 ..........](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485881&idx=1&sn=97aae4d5edf8ea30d363e22ca86b5955&scene=21#wechat_redirect)
    
*   [逆天啦：4 年 crud 小伙收 shein + 银行两优质 offer，狠卷 1 月收年薪 43 万大涨 30%](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485855&idx=1&sn=e9f6b1de7352f09d686c1481632fc454&scene=21#wechat_redirect)
    
*   [原来，拿大厂 offer 有捷径......](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485838&idx=1&sn=72d9733544a5310ee5068fc4533173ea&scene=21#wechat_redirect)
    
*   [太劲爆.... 被裁 4 个月，38 岁 Android 转 Java，2 个月提架构 offer](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485812&idx=1&sn=d87729bea8a8e22f3c2b3eb207d3b38b&scene=21#wechat_redirect)
    
*   [起死回生：8 年小伙高中毕业 + 频繁跳槽，狠卷 2 月，提 Java 高级 offer](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485780&idx=1&sn=03be191625369d33ac54b5c5066c7a68&scene=21#wechat_redirect)
    
*   [爽翻了：指导 3 轮，5 年小伙收 5 大 offer，涨 50%，领路模式太牛](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485663&idx=1&sn=357ab76bc6695485b7941e8eb24a0b42&scene=21#wechat_redirect)
    
*   [降维攻击，37 年大龄老伙喜提 60W 年薪 offer，1 个月顺利上岸](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485643&idx=1&sn=ce71b19e7c9745ab2fe8bfc75f54d993&chksm=97b57943a0c2f055e5c085eec48586e9ba05ae661acabe4cfb7e984c5aeb03d6beeafd204eba&scene=21#wechat_redirect)
    
*   [架构速成：从一面杀到一次过，记 7 年女架构的速成经历](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485571&idx=1&sn=c0096484fd2e1202c14a4bd9d37427c2&chksm=97b5790ba0c2f01da764b8b7b5e3c8a5b02f42d2ed30809f9d7b2d8e73a5bebc9102ef502bc8&scene=21#wechat_redirect)
    
*   [极速拿 offer：阿里 P6 被裁后极速上岸，1 个月内喜提 2 优质 offer(含滴滴)](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485452&idx=1&sn=68102559298564895b1496e2230dddb2&chksm=97b57984a0c2f0927d80f8719a7dc56f43ff13e373eb3886d39be98538180f919225b5ac05f2&scene=21#wechat_redirect)
    
*   [惊天大逆袭：失业 4 个月，3 年小伙 1 个月喜提架构 Offer，而且是大龄跨行，超级牛](http://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247496226&idx=1&sn=459a566cfa5915a720e80e1880fd730d&chksm=c14148a6f636c1b08a321d08384c5d76e184c22f8f4457f980452c45c163bdc5ab91ca17b919&scene=21#wechat_redirect)
    
*   [被裁后炸爽：10 年小伙 12 天火速上岸，反涨 20%，爽暴了](http://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247492928&idx=1&sn=82fd1c530d0ad8ad798b5c35c040641b&chksm=c1415fc4f636d6d25f16e12a73a2635073994ef742efdddcc1185c5d7727785c98fac14e8de8&scene=21#wechat_redirect)
    
*   [裁就裁，6 年小伙 60W 极速上岸，白拿 20W 还游一圈拉萨，真香](http://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247492621&idx=1&sn=f6ada03c596baedcf995e5cb60d255b2&chksm=c1415e89f636d79f13b25d7f3691fb79931515f15b421d65fab1c5e34d4cce4165b6d1772737&scene=21#wechat_redirect)
    
*   [极速拿 offer：被毕业 3 个月，11 年经验小伙 0.5 个月极速拿 offer，领路模式的巨大威力](http://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247491450&idx=1&sn=98379c07e430562a8a244121e63f47d7&chksm=c142a5fef6352ce8e3ee64090a6d0619a9d40484f358302d96990e7655ec16f8d8e7624ca88e&scene=21#wechat_redirect)
    

**职业救助站**

实现职业转型，极速上岸

![](https://mmbiz.qpic.cn/mmbiz_jpg/9qY2Gt3PBs3icE6WcM4nBOdRRcczOW4wkNDgCMZhBfqeet9suwGHfgQduP33YYqqZLOppecMI72Pv5ibNkPeIk6g/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1#imgIndex=16)

关注**职业救助站**公众号，获取每天职业干货  
助您实现**职业转型、职业升级、极速上岸**  
---------------------------------

**技术自由圈**

实现架构转型，再无中年危机

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/xlgvgPaib7WMtyFcJx45oIeWL1d6AENcdoAvuTPuq46E9n8pPe4SHXoSicuEVvhibHRSIzmEBPlC7POcDaqD6ID7w/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1#imgIndex=17)

关注**技术自由圈**公众号，获取每天技术千货  
一起成为牛逼的**未来超级架构师**

**几十篇架构笔记、5000 页面试宝典、20 个技术圣经  
请加尼恩个人微信 免费拿走**

**暗号，请在 公众号后台 发送消息：领电子书**

如有收获，请点击底部的 " 在看 "和" 赞 "，谢谢