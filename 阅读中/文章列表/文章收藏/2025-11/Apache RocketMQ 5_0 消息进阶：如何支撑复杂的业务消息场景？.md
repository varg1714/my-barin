---
source: https://mp.weixin.qq.com/s/LWrh-H7FLa5tZSAapUqPlQ
create: 2025-11-18 21:00
read: true
knowledge: true
knowledge-date: 2025-11-21
tags:
  - 消息队列
summary: "[[消息队列进阶-常见消息队列的核心原理]]"
---
![](https://mmbiz.qpic.cn/mmbiz_gif/yvBJb5IiafvmiaBnXvbGDru5fwoNCGwhCdc2xA5ahKkfBePMIjkboicYBKINVBY43ZVG1CibXJORibIAzkhpAke0PhQ/640?wx_fmt=gif&wxfrom=5&wx_lazy=1#imgIndex=0)

_**一致性**_

_Cloud Native_

首先来看 RocketMQ 的第一个特性 - 事务消息，事务消息是 RocketMQ 与一致性相关的特性，也是 RocketMQ 有别于其他消息队列的最具区分度的特性。

![](https://mmbiz.qpic.cn/mmbiz_png/yvBJb5Iiafvl76l9eTrS3GqsfBRC0qFXRaib7n5GbsCLGuBicNmxI93icLNdxSjVtc0Y41wrlxJwxtu5PW10qkMTGA/640?wx_fmt=png#imgIndex=1)

以大规模电商系统为例，付款成功后会在交易系统中订单数据库将订单状态更新为已付款。然后交易系统再发送一条消息给 RocketMQ，RocketMQ 将订单已付款的事件通知给所有下游应用，保障后续的履约环节。

但上述流程存在一个问题，交易系统写数据库与发消息互相分开，它不是一个事务，会出现多种异常情况，比如数据库写成功但消息发失败，这个订单的状态下游应用接收不到，对于电商业务来说，可能造成大量用户付款但卖家不发货的情况；而如果先发消息成功再写数据库失败，会造成下游应用认为订单已付款，推进卖家发货，但是实际用户未付款成功。这些异常都会对电商业务造成大量脏数据，产生灾难性业务后果。

而 RocketMQ 事务消息的能力可以保障生产者的本地事务（如写数据库）、发消息事务的一致性，最后通过 Broker at least once 的消费语义，保证消费者的本地事务也能执行成功，最终实现生产者、消费者对同一业务的事务状态达到最终一致。

  

**一致性：事务消息 - 原理**  

如下图所示，事务消息主要通过两阶段提交 + 事务补偿机制结合实现。

![](https://mmbiz.qpic.cn/mmbiz_png/yvBJb5Iiafvl76l9eTrS3GqsfBRC0qFXRckwao6ZvLRbtDvyftpk0PeJiaB64oZNIxcdeuib4pXC8Kb9sGDc044MA/640?wx_fmt=png#imgIndex=2)

首先生产者会发送 half 消息，也就是 prepare 消息，broker 会把 half 存到队列中。接下来生产者执行本地事务，一般是写数据库，本地事务完成后，会往 RocketMQ 发送 commit 操作，RocketMQ 会把 commit 操作写入 OP 队列，并进行 compact，把已提交的消息写到 ConsumeQueue 对消费者可见。反过来如果是 rollback 操作，则会跳过对应的 half 消息。

面对异常的情况，比如生产者在发送 commit 或者 rollback 之前宕机了，RocketMQ broker 还会有补偿检查机制，定期回查 Producer 的事务状态，继续推进事务。

无论是 Prepare 消息、还是 Commit/Rollback 消息、或者是 compact 环节，在存储层面都是遵守 RocketMQ 以顺序读写为主的设计理念，达到最优吞吐量。

  

**一致性：事务消息 demo**  

接下来来看一个事务消息的简单示例。使用事务消息需要实现一个事务状态的查询器，这也是和普通消息一个最大的区别。如果我们是一个交易系统，这个事务回查器的实现可能就是根据订单 ID 去查询数据库来确定这个订单的状态到底是否是提交，比如说创建成功、已付款、已退款等。主体的消息生产流程也有很多不同，需要开启分布式事务，进行两阶段提交，先发一个 prepare 的消息，然后再去执行本地事务。这里的本地事务一般就是执行数据库操作。然后如果本地事务执行成功的话，就整体 commit，把之前的 prepare 的消息提交掉。这样一来消费者就可以消费这条消息的。如果本地事务出现异常的话，那么就把整个事务 rollback 掉，之前的那条 prepare 的消息也会被取消掉，整个过程就回滚了。事务消息的用法变化主要体现在生产者代码，消费者使用方式和普通消息一致，demo 里面就不展示了。

![](https://mmbiz.qpic.cn/mmbiz_png/yvBJb5Iiafvl76l9eTrS3GqsfBRC0qFXR2SmMiaiaq0XPrye8ZjvZ6kGA22TN4dnnAqRYt6htdcevxhoNuj37rfNg/640?wx_fmt=png#imgIndex=3)

  

**一致性：顺序消息场景 + 原理**  

RocketMQ 的第二个高级特性是顺序消息，也是特色能力之一。它解决了顺序一致性的问题，保障同一业务的消息，生产与消费的顺序保持一致。

阿里曾有一个场景是买卖家数据库复制，由于阿里订单数据库采用分库分表技术，面向买卖家不同的业务场景，会分别按照买家主键与卖家主键拆分为买卖家数据库。两个数据库的同步采用 Binlog 顺序分发的机制，通过使用顺序消息，将买家库的 Binlog 变更按照严格顺序在卖家库回放，以此达到订单数据库的一致性。如果没有顺序保障，则可能出现数据库级别的脏数据，会带来严重的业务错误。

顺序消息的实现原理如下图所示，充分利用 Log 天然顺序读写的特点高效实现。

![](https://mmbiz.qpic.cn/mmbiz_png/yvBJb5Iiafvl76l9eTrS3GqsfBRC0qFXRGI3SdyUxlo0xaueTr3VSsw7mPVqtibNFbDN1emTSnvca2qvyztBtuJg/640?wx_fmt=png#imgIndex=4)

在 Broker 存储模型中，每个 Topic 都会有固定的 ConsumeQueue，可以理解为 Topic 的分区。生产者为发送消息加上业务 Key，在这个 case 里面可以用订单 ID，同一订单 ID 的消息会顺序发送到同一个 Topic 分区，每个分区在某个时刻只会被一个消费者锁定，消费者顺序读取同一个分区的消息串行消费，以此来达到顺序一致性。

  

**一致性：顺序消息 demo**  

接下来来看顺序消息的一个简单 demo。对于顺序消息而言，生产者与消费者都有需要注意的地方。

![](https://mmbiz.qpic.cn/mmbiz_png/yvBJb5Iiafvl76l9eTrS3GqsfBRC0qFXRY8iaTflkk7SrHhonxrktRzIvED4BBPq1IjSJtaJBXsLCiaoGicr7N93kg/640?wx_fmt=png#imgIndex=5)

在生产阶段，首先要定义消息的 group。每条消息都可以选择业务 ID 作为消息 Group，业务 ID 尽量离散、随机。因为同一业务 ID 会分配到同一数据存储分片，生产与消费都在该数据分片上串行，如果业务 ID 有热点，会造成严重的数据倾斜与局部消息堆积。

比如在电商交易的场景，选择订单 ID、买家 ID 会比较好，比较离散。如果选择的是卖家 ID，则可能会出现热点，热点卖家的流量会远大于普通卖家。

消费阶段也与常规的消息收发有所区别，主要有两种模式，分别是全托管的 push consumer 模式和半托管主动获取消息的 simple consumer 模式。RocketMQ SDK 会保障同一分组的消息串行进入业务消费逻辑。需要注意，自身的业务消费代码也要串行进行，然后同步返回消费成功确认。不要将同一分组的消息放到另外的线程池消并发费，会破坏顺序语义。

_**复杂业务**_

_Cloud Native_

  

**复杂业务：SQL 过滤场景**  

RocketMQ 的第三个高级特性是 SQL 消费模式，也是复杂业务场景的刚需。

![](https://mmbiz.qpic.cn/mmbiz_png/yvBJb5Iiafvl76l9eTrS3GqsfBRC0qFXR3I1px4vVaMSefVNwlQawBN1LjsYiaBN7vkp7mPjBiadZsrkBXIqLxwRg/640?wx_fmt=png#imgIndex=6)

如上图，阿里的电商业务围绕着交易展开，有数百个不同的业务在订阅交易消息。业务基本面向某个细分领域，都只需要交易 Topic 下的部分消息。按照传统的模式，一般是全量订阅交易 Topic，在消费者本地过滤即可，但这样会消耗大量计算、网络资源，特别是在双十一，该方案的成本是无法接受的。

  

**复杂业务：SQL 过滤原理**  

为了解决上述问题，RocketMQ 提供了 SQL 消费模式。在交易场景下，每笔订单消息都会带有不同维度的业务属性，包括卖家 ID、买家 ID、类目、省市、价格、订单状态等属性，而 SQL 过滤能让消费者通过 SQL 语句过滤消费目标消息。比如，某个消费者只想关注某个价格区间内的订单创建消息，创建订阅关系 Topic=Trade SQL：status=ordercreate and（Price between 50 and 100），Broker 会在服务端运行 SQL 计算，只返回有效数据给消费者。

为了提高性能，Broker 还引入了布隆过滤器模块。在消息写入分发时刻提前计算结果，写入位图过滤器，减少无效 IO。

总体而言，其本质为将过滤链路不断前置，从消费端本地过滤，到服务端写时过滤，达到最优性能。

![](https://mmbiz.qpic.cn/mmbiz_png/yvBJb5Iiafvl76l9eTrS3GqsfBRC0qFXRPPgEwQVRWqZgOBPictqbYqN4zXfOdTPlicpkHTs4mO9MoJb1Mwu5YWTA/640?wx_fmt=png#imgIndex=7)

  

**复杂业务：SQL 过滤 demo**  

接下来看一个 SQL 订阅的示例。目前 RocketMQ SQL 过滤支持属性非空判断、属性大小比较、属性区间过滤、集合判断与逻辑计算，能满足绝大部分的过滤需求。

![](https://mmbiz.qpic.cn/mmbiz_png/yvBJb5Iiafvl76l9eTrS3GqsfBRC0qFXRfWNZamBgX4Smu0euibzUBkanlJSJhmZQsB3abDybsdfuqd7iabZTkcxA/640?wx_fmt=png#imgIndex=8)

在消息生产阶段，除了设置 Topic、Tag 之外，还能添加多个自定义属性。比如在这案例里，设置了一个 region 的属性，表示该条消息从杭州 region 发出。消费时可根据自定义属性来进行 SQL 过滤订阅。第一个 case 是用了一个 filter expression，判断 region 这个字段不为空且等于杭州才消费。第二个 case 添加更多的条件，如果这是一笔订单消息，还可以同时判断 region 条件和价格区间来决定是否消费。第三个 case 是全接收模式，表达式直接为 True，这个订阅方式会接收某一个主题下面的全量消息，不进行任何过滤。

  

**复杂业务：定时消息场景 + 原理**  

RocketMQ 的第四个高级特性是定时消息。

![](https://mmbiz.qpic.cn/mmbiz_png/yvBJb5Iiafvl76l9eTrS3GqsfBRC0qFXRdn9BK5vzicib4iaLR8Y5FSGGfPc7qiaKwdFHGHVHwNzEU0Ol4R72F8CT3Q/640?wx_fmt=png#imgIndex=9)

生产者可以指定某条消息在发送后经过一定时间后才对消费者可见。有不少业务场景需要大规模的定时事件触发，比如典型的电商场景基本都有订单创建 30 分钟未付款自动关闭订单的逻辑，定时消息能为上述场景带来极大的便利性。

RocketMQ 的定时消息基于时间轮（TimerWheel）来实现。通过模拟表盘转动来达到对时间进行排序的目的。

TimerWheel 中的每一格代表最小的时间刻度，称为 Tick。RocketMQ 里，每一个 Tick 为一秒，同一时刻的消息会写入到同一格子里。由于每个时刻可能会同时触发多条消息，并且每条消息的写入时刻都不一样，因此 RocketMQ 也同时引入了 Timerlog 的数据结构，Timerlog 按照顺序 append 的方式写入数据，每个元素都包含消息的物理索引以及指向同一时刻的前一条消息，组成逻辑链表。TimeWheel 的每个格子都维护该时刻的消息链表的头尾指针。

TimerWheel 会有指针，代表当前时刻，绕着 TimerWheel 循环转动，指针所指之处代表该 Tick 到期，所有内容一起弹出，写到 ConsumeQueue，对消费者可见。

目前 RocketMQ 的定时消息性能已经远超 RabbitMQ 与 ActiveMQ。

![](https://mmbiz.qpic.cn/mmbiz_png/yvBJb5Iiafvl76l9eTrS3GqsfBRC0qFXR3NYVMmdpKjGfHJic6aAp8fDCRHLY7slQDdFlN9SX0VkrUf12JnD4WBA/640?wx_fmt=png#imgIndex=10)

_**全局高可用**_

_Cloud Native_

接下来再讲一下 RocketMQ 的全局高可用技术解决方案。RocketMQ 的高可用架构主要指 RocketMQ 集群内的数据多副本与服务高可用。而本文的高可用是全局的、业界常说的同城容灾、两地三中心、异地多活等架构。

![](https://mmbiz.qpic.cn/mmbiz_png/yvBJb5Iiafvl76l9eTrS3GqsfBRC0qFXRlyN6YHpcSgJoKIWYD1W2tLuAUH4uiajt398Po9bVDJx2joYAHQ4NjaA/640?wx_fmt=png#imgIndex=11)

目前，蚂蚁支付与阿里交易均采用异地多活的架构，异地多活相对于冷备、同城容灾、两地三中心模式具备更多优点，可以应对城市级别的灾难，如地震、断电等事件。除此之外，针对一些因为人为操作引起的问题，比如某个基础系统变更引入新的 bug，导致整个机房级别的不可用，异地多活架构可以直接将流量切到可用机房，优先保障业务连续性，再定位具体的问题。

另一方面，异地多活还能实现机房级别的扩容，单一机房的计算存储资源有限，而异地多活架构可以将业务流量按照比例分散在全国各地机房。同时，多活架构实现了所有机房都提供业务服务，而不是冷备状态，资源利用率大幅度提升。得益于多活状态，面对极端场景的切流，可用性更有保障，信心更足。

在异地多活的架构中，RocketMQ 承担的是基础架构的多活能力。多活的架构分为几个模块：

*   **接入层：**通过统一接入层按照业务 ID 将用户请求分散到多个机房，业务 ID 一般可采用用户 ID。
    
*   **应用层：**应用层一般无状态，当请求进入某个机房后，需要尽量保障该请求的整个链路都在单元内封闭，包括 RPC、数据库访问、消息读写，可降低访问延迟，保障系统性能不会因为多活架构衰退。
    
*   **数据层：**包括数据库、消息队列等有状态系统。RocketMQ 通过 connector 组件实现按 topic 粒度实时同步消息的数据，按照 Consumer 与 Topic 的组合粒度实时同步消费状态。
    
*   **全局的管控层：**需要维护全局的单元化规则，分配哪些流量走到哪些机房；还需要管理多活元数据配置，哪些应用需要多活、哪些 Topic 需要多活；另外，在切流时刻需要协调所有系统的切流过程，控制切流顺序。
    

_**总结**_

_Cloud Native_

本篇文章介绍了很多 RocketMQ 的高阶特性。首先是一致性的特性，这里面就包括了顺序的一致性、分布式业务的一致性；RocketMQ 在应对大规模复杂业务的特性有 2 个，一个是 SQL 过滤订阅，可以应对那种单一超大业务大量消费者过滤需求；还有一个是定时消息，这也是很多互联网交易业务常见的场景。最后，介绍了 RMQ 在高阶的容灾能力方面的建设，提供了一个异地多活的解决方案。

_**【活动】带你玩转 RocketMQ，角逐「RocketMQ 首席评测官」**_  

为了更好地长期得到开发者实际使用中的反馈和建议，联合阿里云开发者社区推出了 “寻找 RocketMQ 首席评测官” 活动，寻找在消息领域有技术实践经验、愿意深度评测产品并提出宝贵建议的开发者。期待您的加入，帮助 Apache RocketMQ 以及阿里云消息产品持续提升竞争力。

  

**活动入口  
**

点击此处立即参与活动：（或前往文末阅读原文进入）

_https://developer.aliyun.com/topic/rocketmq?utm_content=g_1000377381&spm=1000.2115.3001.5954_

可以直接进行产品评测：

_https://developer.aliyun.com/mission/review/rocketmqtest?spm=a2c6h.28281744.J_2889796290.5.c66c5bacLDNt46_

![](https://mmbiz.qpic.cn/mmbiz_png/yvBJb5Iiafvl76l9eTrS3GqsfBRC0qFXRaDH91UYokZcarGPUoBovHWicra3smaLbaKwOoMYxtovNgibC46G49ibxQ/640?wx_fmt=png#imgIndex=12)

点击阅读原文，立即参与活动