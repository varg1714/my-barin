---
source: https://mp.weixin.qq.com/s/gnRiDUhAvTgjXaQiROB66A
create: 2025-11-18 21:02
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

说说 Rocketmq 的推模式、拉模式？

这个题目，是非常常见的面试题，回答的时候， 有两个层面

*   第一个层面：应用开发层
    
*   第二个层面：底层源码层
    

关于 Rocketmq 的核心面试题，尼恩前面也梳理过几篇文章：

[阿里面试：如何保证 RocketMQ 消息有序？如何解决 RocketMQ 消息积压？](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247500158&idx=1&sn=483783e762febb12f46b60b99ae1180b&scene=21#wechat_redirect)

[RocketMQ 顺序消息，是 “4 把锁” 实现的（顺序消费）](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247500939&idx=1&sn=36dd5fa5f5d7d26da67410e96e098968&scene=21#wechat_redirect)

这里，又新增一个核心面试的答案，“说说 Rocketmq 的推模式、拉模式？”。

这些文章，在底层都是相同的。帮助大家从 Rocketmq 源码层去解答，那就更加让面试官 “不能自已、口水直流、震惊不已”，然后实现”offer 直提”。

当然，这道面试题，以及参考答案，也会收入咱们的 《[尼恩 Java 面试宝典 PDF](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247497474&idx=1&sn=54a7b194a72162e9f13695443eabe186&scene=21#wechat_redirect)》V158 版本，供后面的小伙伴参考，提升大家的 3 高 架构、设计、开发水平。

《尼恩 架构笔记》《尼恩高并发三部曲》《尼恩 Java 面试宝典》的 PDF，请到文末公号【技术自由圈】获取

## 本文目录

**- 尼恩说在前面**

**- 经典的推模式 / 拉模式**

 - 经典的推模式

 - 经典的拉模式

**- 推拉模式，如何选型？**

**- Rocketmq 的推模式和拉模式**

**- RocketMQ 中的 PushConsumer 推模式**

 - 客户端没有消息怎么办呢？

 - PUSH 模式的应用开发

 - 消费者消息处理过程

**- Rocketmq 拉模式 / PULL 模式**

 - RocketMQ 中的 PushConsumer 推模式的源码分析

 - pull 模式，消费者消息处理过程

**- Rocketmq 的 Push 与 Pull 模式比较**

**- 说在最后：有问题可以找老架构取经**

**- 部分历史案例**

## 经典的推模式 / 拉模式

首先，明确一下业务场景

这里谈论的推拉模式，指的是 Consumer 和 Broker 之间，不是 producer 与 broker 之间。

### 经典的推模式

经典的推模式，指的是消息从 Broker 推向 Consumer。

Consumer 被动的接收消息，由 Broker 来主导消息的发送。作为代理人，Broker 接受完消息之后，可以立马推送给 Consumer。

Consumer 等着就行，消息会有 broker 主动推过来。所以 Consumer 的处理策略很简单。

**推模式的缺点：Consumer 可能就 “消化不良 / OOM”。**

当 Broker 推送消息的速率大于 Consumer 消费速率时，Consumer 可能就 “消化不良”，出现内存积压，内存溢出，OOM，因为根本消费不过来啊。

所以，经典的推模式，适用于消息量不大、Consumer 消费能力强的场景。

### 经典的拉模式

经典的拉模式，指的是 Consumer 主动向 Broker 请求拉取消息。

上面讲到，经典的推模式，适用于消息量不大、Consumer 消费能力强的场景。如果 Consumer 消费能力弱， 那么就改变方向好了， 由推改完拉。

拉的话，主动权就在 Consumer 身上了， 能消化多少，吃多少。

假设当前 Consumer 消化不过来、消费不过来了，它可以根据一定的策略，暂停拉取甚至停止拉取，或者间隔拉取都行。

凡事有利必有弊。

**拉模式的缺点：消息延迟 + 消息积压**。 如果 Consumer 隔个 2 天采取拉取一批，消息就很有可能延迟，甚至出现严重的消息延迟。而且 Broker 服务端大概率会消息积压。

## 推拉模式，如何选型？

选择推模式的消息队列中间件，主要有 ActiveMQ

选择推模式的消息队列中间件，主要有 RocketMQ 和 Kafka

虽然 RocketMQ 和 Kafka 都选择了拉模式。也就是允许消息延迟 + 允许消息积压。

所以，选择 RocketMQ 和 Kafka，就需要做好消息积压的监控。

关于消息积压，参考答案请参见尼恩《技术自由圈》前面的一篇文章

[阿里面试：如何保证 RocketMQ 消息有序？如何解决 RocketMQ 消息积压？](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247500158&idx=1&sn=483783e762febb12f46b60b99ae1180b&scene=21#wechat_redirect)

关于积压监控，请参考尼恩的 《Rocketmq 四部曲视频》，如果能够回答到上面的层次，已经非常牛掰了。

## Rocketmq 的推模式和拉模式

Rocketmq 的客户端，也定义了两个模式：推模式和拉模式

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPnoPkt6uHL3Fl8hzAJkNWP3cicXGCAmLbBumdRQXJ7WqTZyfnibQzT0Hv2sn5dp5AS8g5X809jhUlQ/640?wx_fmt=png&from=appmsg#imgIndex=0)

但是实际上，RocketMQ 中的 PushConsumer 推模式，仅仅是披着拉模式的方法，本质还是拉模式。

## RocketMQ 中的 PushConsumer 推模式

接下来，我们首先看看 RocketMQ 中的 PushConsumer 推模式。

这里，建议大家看看，尼恩的前面一篇文章：

[惊呆：RocketMQ 顺序消息，是 “4 把锁” 实现的（顺序消费）](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247500939&idx=1&sn=36dd5fa5f5d7d26da67410e96e098968&scene=21#wechat_redirect)

介绍了  RocketMQ  拉取消息的核心流程，具体如下图所示。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPnoPkt6uHL3Fl8hzAJkNWPG3yF4CK6vH9ddRSQsjTFs8MYv2jB732ku3Uvu1HpaJiauicMOWLj2erw/640?wx_fmt=png&from=appmsg#imgIndex=1)

一个消费者至少需要涉及队列自动负载、消息拉取、消息消费、位点提交、消费重试等几个部分。

MQClientInstance 客户端实例，会开启多个异步并行服务：

*   负载均衡服务 rebalanceService ：再平衡服务.
    
    专门进行 queue 分区的 再平衡，再分配，然后发布拉取消息的请求 pullRequest 实例。
    
*   消息拉取服务 pullMessageService：专门负责拉取消息。
    
    从请求队列 pullRequestQueue 队列 获取一个一个的 pullRequest，
    
    通过内部实现类 DefaultMQPushConsumerImpl 拉取 消息。
    
    注意，拉取的消息，放在另一个队列 messageQueue 缓存，拉取之前，会进行流控检查，如果这个队列满了（>1000 个消息或者 >100M 内存） 则延迟 50ms 再拉取， 当然，下一次执行拉取之前，同样也会进行流控检查
    
*   消息消费线程：ConsumeMessageOrderlyService  有序消息消费， 或者 并行消息。 从 messageQueue  拉取消息，进行消费。
    

上面设计 3 类线程，在 3 类线程之间，通过两个队列进行 同步：

*   拉取消息的请求队列 pullRequestQueue
    
*   缓存消息的队列  messageQueue
    

Rocketmq 的推模式，本质是一种拉模式， 只是为了让客户端不会  累死， 在拉取之前进行流控。

具体请参见 尼恩 《Rocketmq 四部曲视频》配套的 注释版源码：

```java
//   接下来就是消费者的拉取流量控制，阈值为 1000个消息， 或者 100M
// 消费者消费的太慢了，broker推送的太快了，进行 Flow control

if (cachedMessageCount > this.defaultMQPushConsumer.getPullThresholdForQueue()) {

    // 将pullRequest放入队列，只不过是经过后台的定时线程池延50 ms 迟放入，进行 Flow control
    // 流量控制， 减缓拉取消息的速度
    //     * Flow control threshold on queue level, each message queue will cache at most 1000 messages by default,
    //     * Consider the {@code pullBatchSize}, the instantaneous value may exceed the limit
    //     */
    //    private int pullThresholdForQueue = 1000;

    this.executePullRequestLater(pullRequest, PULL_TIME_DELAY_MILLS_WHEN_FLOW_CONTROL);
    if ((queueFlowControlTimes++ % 1000) == 0) {
        log.warn(
            "the cached message count exceeds the threshold {}, so do flow control, minOffset={}, maxOffset={}, count={}, size={} MiB, pullRequest={}, flowControlTimes={}",
            this.defaultMQPushConsumer.getPullThresholdForQueue(), processQueue.getMsgTreeMap().firstKey(), processQueue.getMsgTreeMap().lastKey(), cachedMessageCount, cachedMessageSizeInMiB, pullRequest, queueFlowControlTimes);
    }
    return;
}


```

  

### 客户端没有消息怎么办呢？

上一节讲到， RocketMQ 三类线程，相互配合：在背后偷偷的帮我们去 Broker 拉消息。

第一类线程  RebalanceService ，根据 topic 的队列数量和消费者个数做负载均衡，对于分配到 queue 产生的 pullRequest 拉取请求，并讲请求 队列 pullRequestQueue 中。

第二类线程  PullMessageService ，不断的 pullRequestQueue 队列 中获取 pullRequest，然后从 broker 拉取消息。

那么，如果 broker 暂时没有消息，怎么办呢？

PullMessageService    把拉取请求，重新放进 pullRequestQueue 队列， 大致的代码如下：

```java
//broker 没有 新消息
case NO_NEW_MSG:
pullRequest.setNextOffset(pullResult.getNextBeginOffset());

DefaultMQPushConsumerImpl.this.correctTagsOffset(pullRequest);

// 把拉取请求，重新放进队列

DefaultMQPushConsumerImpl.this.executePullRequestImmediately(pullRequest);
break;
//broker 没有 匹配消息
case NO_MATCHED_MSG:
pullRequest.setNextOffset(pullResult.getNextBeginOffset());

DefaultMQPushConsumerImpl.this.correctTagsOffset(pullRequest);
// 把拉取请求，重新放进队列

DefaultMQPushConsumerImpl.this.executePullRequestImmediately(pullRequest);


```

Broker 处理拉取消息命令的  处理器，叫做 PullMessageProcessor。

PullMessageProcessor 里面的 processRequest 方法是用来处理 pullRequest 拉消息请求.

如果 broker 有消息， processRequest 方法就直接返回，

如果 broker 没有消息， processRequest 方法怎么办呢？

我们来看一下代码。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPnoPkt6uHL3Fl8hzAJkNWPDupdYuTKBNhPficrdzQF4l3ZawPdtzTRcfcnTZ1bKU5KBZ3A1IvqqTg/640?wx_fmt=png&from=appmsg#imgIndex=2)

我们再来看下 suspendPullRequest 方法做了什么。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPnoPkt6uHL3Fl8hzAJkNWPEKDE7dQ7o26LnJSpVia7micGIHyejxMNIXhhFP9ncCl2QkJTAicXKNVKg/640?wx_fmt=png&from=appmsg#imgIndex=3)

这里有个 broker 异步线程 PullRequestHoldService

这个线程会每 5 秒从 pullRequestTable 取 PullRequest 请求，然后进行检查，看看是否有新的消息

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPnoPkt6uHL3Fl8hzAJkNWPiagUyoqanjesNkAXGdc9sr1Hs0HGD2eDmMHC5ReicofhNSXQYNDlplPA/640?wx_fmt=png&from=appmsg#imgIndex=4)

检查方法是：计算 待拉取消息请求的偏移量是否小于当前消费队列最大偏移量，如果条件成立则说明有新消息了，

一旦有消息，PullRequestHoldService 则会调用 notifyMessageArriving ，最终调用 PullMessageProcessor 的 executeRequestWhenWakeup() 方法重新尝试处理这个消息的请求，也就是再来一次，整个长轮询的时间默认 30 秒。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPnoPkt6uHL3Fl8hzAJkNWPgkSbQIHiaeQff9ZAHQxFzrWqgY7jibPWT44Pd0EcLvnlJjhuBwawaP6A/640?wx_fmt=png&from=appmsg#imgIndex=5)

简单的说就是 5 秒会检查一次消息时候到了，如果到了则调用 processRequest 再处理一次。

这里是一个定期检查的流程。除此之外，如果 commitLog 有消息，也会执行唤醒的工作，做到准实时。

brocker 端的 ReputMessageService 线程，不断地为 commitLog 追加数据并分发请求，构建出 ConsumeQueue 和 IndexFile 两种类型的数据，**并且也会有唤醒请求的操作，来弥补每 5s 一次这么慢的延迟**

### PUSH 模式的应用开发

下面是 RocketMQ 推模式的一个官方示例：

```java
public static void main(String[] args) throws InterruptedException, MQClientException {
    Tracer tracer = initTracer();

    DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("CID_JODIE_1");
    consumer.getDefaultMQPushConsumerImpl().registerConsumeMessageHook(new ConsumeMessageOpenTracingHookImpl(tracer));

    consumer.subscribe("TopicTest", "*");
    consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);

    consumer.setConsumeTimestamp("20181109221800");
    consumer.registerMessageListener(new MessageListenerConcurrently() {
        @Override
        public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {
            System.out.printf("%s Receive New Messages: %s %n", Thread.currentThread().getName(), msgs);
            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
        }
    });
    consumer.start();
    System.out.printf("Consumer Started.%n");
}


```

消费者会定义一个消息监听器 MessageListenerConcurrently，并且把这个监听器注册到 DefaultMQPushConsumer ，

这个监听器，最终会注册到内部 DefaultMQPushConsumerImpl，当内部拉取到消息时，就会使用这个监听器来处理消息。

### 消费者消息处理过程

下面用并发消费方式下的同步拉取消息为例总结一下消费者消息处理过程：

第一类线程  RebalanceService ，根据 topic 的队列数量和消费者个数做负载均衡，对于分配到 queue 产生的 pullRequest 拉取请求，并讲请求 队列 pullRequestQueue 中。

第二类线程  PullMessageService ，不断的 pullRequestQueue 队列 中获取 pullRequest，然后从 broker 拉取消息。具体来说，这里调用了 DefaultMQPushConsumerImpl 类的 pullMessage 方法；pullMessage 方法调用 PullAPIWrapper 的 pullKernelImpl 方法真正去发送 PULL 请求，并传入 PullCallback 的 回调函数；拉取到消息后，调用 PullCallback 的 onSuccess 方法处理结果，会把消息放入到 缓存消息的队列  messageQueue

第三类线程消息消费线程：ConsumeMessageOrderlyService  有序消息消费， 或者 ConsumeMessageConcurrentlyService 并行消息服务。 从 messageQueue  拉取消息，进行消费。这里调用了 ConsumeMessageConcurrentlyService 的 submitConsumeRequest 方法，通过里面的 ConsumeRequest 线程来处理拉取到的消息；处理消息时调用了消费端定义的消费逻辑，也就是 MessageListenerConcurrently 的 consumeMessage 方法。

## Rocketmq 拉模式 / PULL 模式

下面是来自官方的一段 拉模式 / PULL 模式拉取消息的代码：

```java
DefaultLitePullConsumer litePullConsumer =
                new DefaultLitePullConsumer("lite_pull_consumer_test");
litePullConsumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);
litePullConsumer.subscribe("TopicTest", "*");
litePullConsumer.start();
try {
    while (running) {
        List<MessageExt> messageExts = litePullConsumer.poll();
        System.out.printf("%s%n", messageExts);
    }
} finally {
    litePullConsumer.shutdown();
}


```

上面代码中写了一个死循环 ， 客户端通过 PULL 模式，不断的调用 poll 方法，不停的去拉取消息。

从这段代码可以看出， 通过拉模式 / PULL 模式 的 pullRequest 请求，不是 Rocketmq 源码去发出，也不用 PullMessageService  线程，这个 pullRequest  请求是有 客户端应用程序自己去发。

Rocketmq 源码内部，拉模式消费使用的是 DefaultMQPullConsumer/DefaultLitePullConsumerImpl，核心逻辑是先拿到需要获取消息的 Topic 对应的队列，然后依次从队列中拉取可用的消息。拉取了消息后就可以进行处理，处理完了需要更新消息队列的消费位置。

下面有一个更加生产化的案例

```java
@Test
public void testPullConsumer() throws Exception {
    DefaultMQPullConsumer consumer = new DefaultMQPullConsumer("group1_pull");
    consumer.setNamesrvAddr(this.nameServer);
    String topic = "topic1";
    consumer.start();

    //获取Topic对应的消息队列
    Set<MessageQueue> messageQueues = consumer.fetchSubscribeMessageQueues(topic);
    int maxNums = 10;//每次拉取消息的最大数量
    while (true) {
        boolean found = false;
        for (MessageQueue messageQueue : messageQueues) {
            long offset = consumer.fetchConsumeOffset(messageQueue, false);
            PullResult pullResult = consumer.pull(messageQueue, "tag8", offset, maxNums);
            switch (pullResult.getPullStatus()) {
                case FOUND:
                    found = true;
                    List<MessageExt> msgs = pullResult.getMsgFoundList();
                    System.out.println(messageQueue.getQueueId() + "收到了消息，数量----" + msgs.size());
                    for (MessageExt msg : msgs) {
                        System.out.println(messageQueue.getQueueId() + "处理消息——" + msg.getMsgId());
                    }
                    long nextOffset = pullResult.getNextBeginOffset();
                    consumer.updateConsumeOffset(messageQueue, nextOffset);
                    break;
                case NO_NEW_MSG:
                    System.out.println("没有新消息");
                    break;
                case NO_MATCHED_MSG:
                    System.out.println("没有匹配的消息");
                    break;
                case OFFSET_ILLEGAL:
                    System.err.println("offset错误");
                    break;
            }
        }
        if (!found) {//没有一个队列中有新消息，则暂停一会。
            TimeUnit.MILLISECONDS.sleep(5000);
        }
    }
}


```

下面代码就演示了使用 DefaultMQPullConsumer 拉取消息进行消费的示例。

核心方法就是调用 consumer 的 pull() 拉取消息。

该示例中使用的是同步拉取，即需要等待 Broker 响应后才能继续往下执行。如果有需要也可以使用提供了 PullCallback 的重载方法。同步的 pull() 返回的是 PullResult 对象，其中的状态码有四种状态，并且分别对四种状态进行了不同的处理。

只有状态为 FOUND 才表示拉取到了消息，此时可以进行消费。

消费完了需要调用 updateConsumeOffset() 更新消息队列的消费位置，这样下次通过 fetchConsumeOffset() 获取消费位置时才能获取到正确的位置。

如果有需要，用户也可以自己管理消息的消费位置。

### RocketMQ 中的 PushConsumer 推模式的源码分析

那 PULL 模式中 poll 函数是怎么实现的呢？

跟踪源码可以看到，消息拉取的时候，DefaultLitePullConsumerImpl 工作过程基本与 DefaultMQPushConsumer 过程相似。

DefaultLitePullConsumerImpl 允许设置是否需要自动 commit offset（默认自动），并且把拉取到的消息缓存在内存中，Conumser 需要主动通过 poll 从内存中获取消息，进行业务处理。

DefaultLitePullConsumerImpl 类中的一个方法，首先根据负载均衡服务分配到的 queue 分区，启动 拉取任务

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPnoPkt6uHL3Fl8hzAJkNWPjzoFeu81ib9ycmutH08Z41b7hWxKeU8LsBL9CwcoYictlOAJ6x86iaJ3w/640?wx_fmt=png&from=appmsg#imgIndex=6)

通过定时任务进行消息的拉取

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPnoPkt6uHL3Fl8hzAJkNWPtnwA5AW2KHuliac4JUawO61fMTrbW6OqUzax9Fkqbj9sU9K9noO0xGg/640?wx_fmt=png&from=appmsg#imgIndex=7)

PullTaskImpl 拉取到消息后，封装成 ConsumeRequest , 提交的 consumeRequestCache 缓存中

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPnoPkt6uHL3Fl8hzAJkNWPDhTmusibRufpeA7Ho4mQnSicZNtXAvTO3tvI8VI7EN6KwsT533RqMhCA/640?wx_fmt=png&from=appmsg#imgIndex=8)

内存缓存 consumeRequestCache 类型为 BlockingQueue

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPnoPkt6uHL3Fl8hzAJkNWPsjHAXlKfTParD4tDFd3MIC8kP99zbU9dzNCqYMGyeHZok4bVMhggog/640?wx_fmt=png&from=appmsg#imgIndex=9)

消费者代码中，通过循环，调用 poll 方法，不停地从 consumeRequestCache 拉取消息进行处理

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPnoPkt6uHL3Fl8hzAJkNWPFKSiciaAO282O0HgBCuWzKTGmomEibXgjUA1EzViaahRTnKDPKpicvGicT9Q/640?wx_fmt=png&from=appmsg#imgIndex=10)

  

### pull 模式，消费者消息处理过程：

总结一下，pull 模式，消费者消息处理过程：

1.  消费者启动过程中，负载均衡线程 RebalanceService 线程发现 ProcessQueueTable 消费快照发生变化时，启动消息拉取线程；
    
2.  消息拉取线程 PullTaskImpl 拉取到消息后，把消息放到 consumeRequestCache，然后进行下一次拉取；
    
3.  消费者调用 poll 方法，不停地从 consumeRequestCache 拉取消息，进行业务处理。
    

## Rocketmq 的 Push 与 Pull 模式比较

1、Push 模式拉取消息，拉取到消息马上推送 lisener 进行业务处理。

应用程序对消息的拉取过程参与度不高，可控性不足，仅仅提供消息监听器的实现。

2、Pull 模式自，自主决定如何拉取消息，从什么位置拉取消息。

应用程序对消息的拉取过程参与度高，由可控性高，可以自主决定何时进行消息拉取，从什么位置 offset 拉取消息

上面的流程梳理，涉及到 Rocketmq 源码学习。

Rocketmq 源码用了大量的架构模式、设计模式，可以理解为中间件架构的巅峰之作。尼恩的 《RocketMQ 四部曲视频》，从架构师视角揭秘 RocketMQ 的架构哲学，让大家彻底的了解这个高深莫测 RocketMQ 组件的宏观架构，提升大家的架构水平和设计水平。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPnoPkt6uHL3Fl8hzAJkNWPvUZ1PI27fA2ePyexm6lxUq2bgAico3HnnKYtNyIQtXBL243MdAVNT7Q/640?wx_fmt=png&from=appmsg#imgIndex=11)

## 说在最后：有问题可以找老架构取经

Rocketmq 相关的面试题，是非常常见的面试题。

以上的内容，如果大家能对答如流，如数家珍，基本上 面试官会被你 震惊到、吸引到。

最终，**让面试官爱到 “不能自已、口水直流”**。offer， 也就来了。

在面试之前，建议大家系统化的刷一波 5000 页《[尼恩 Java 面试宝典 PDF](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247497474&idx=1&sn=54a7b194a72162e9f13695443eabe186&scene=21#wechat_redirect)》，里边有大量的大厂真题、面试难题、架构难题。很多小伙伴刷完后， 吊打面试官， 大厂横着走。

在刷题过程中，如果有啥问题，大家可以来 找 40 岁老架构师尼恩交流。

另外，如果没有面试机会，**可以找尼恩来改简历、做帮扶。**

尼恩指导了大量的小伙伴上岸，前段时间，**刚指导一个 40 岁 + 被裁小伙伴，拿到了一个年薪 100W 的 offer。**

## 部分历史案例

*   [被裁不慌，9 年小伙 1 个月喜提年薪 60W offer，做中间件架构，爽歪了](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485994&idx=1&sn=50b95d3ae3009d395458c8740595977f&scene=21#wechat_redirect)
    
*   [12 年小伙转架构，1 小时蜕变收年薪 55 万 offer，秘诀：GO+Java 双栖架构](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485972&idx=1&sn=9f2b5059aaad70c8018b7c6c95a08994&scene=21#wechat_redirect)
    
*   [被裁 6 个月，40 岁小伙猛卷 3 月，100W 年薪逆袭 ，上岸秘诀：首席架构 / 总架构](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485942&idx=1&sn=faa999fc9508435db7a88af2716492d9&scene=21#wechat_redirect)
    
*   [不怕裁员，8 年小伙去中年危机，秘诀：换架构师赛道，上得厅堂，下得厨房，越早越好](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485914&idx=1&sn=7832d0b63155f12eb8036321a99991c8&scene=21#wechat_redirect)
    
*   [被裁员后因祸得福，拿 N+1 后，逆涨 30%，如何实现？](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485881&idx=1&sn=97aae4d5edf8ea30d363e22ca86b5955&scene=21#wechat_redirect)
    
*   [逆天啦：4 年 crud 小伙收 shein + 银行两优质 offer，狠卷 1 月收年薪 43 万大涨 30%](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485855&idx=1&sn=e9f6b1de7352f09d686c1481632fc454&scene=21#wechat_redirect)
    
*   [原来，拿大厂 offer 有捷径......](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485838&idx=1&sn=72d9733544a5310ee5068fc4533173ea&scene=21#wechat_redirect)
    
*   [太劲爆.... 被裁 4 个月，38 岁 Android 转 Java，2 个月提架构 offer](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485812&idx=1&sn=d87729bea8a8e22f3c2b3eb207d3b38b&scene=21#wechat_redirect)
    
*   [起死回生：8 年小伙高中毕业 + 频繁跳槽，狠卷 2 月，提 Java 高级 offer](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485780&idx=1&sn=03be191625369d33ac54b5c5066c7a68&scene=21#wechat_redirect)
    
*   [爽翻了：指导 3 轮，5 年小伙收 5 大 offer，涨 50%，领路模式太牛](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485663&idx=1&sn=357ab76bc6695485b7941e8eb24a0b42&scene=21#wechat_redirect)
    
*   [降维攻击，37 年大龄老伙喜提 60W 年薪 offer，1 个月顺利上岸](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485643&idx=1&sn=ce71b19e7c9745ab2fe8bfc75f54d993&chksm=97b57943a0c2f055e5c085eec48586e9ba05ae661acabe4cfb7e984c5aeb03d6beeafd204eba&scene=21#wechat_redirect)
    
*   [架构速成：从一面杀到一次过，记 7 年女架构的速成经历](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485571&idx=1&sn=c0096484fd2e1202c14a4bd9d37427c2&chksm=97b5790ba0c2f01da764b8b7b5e3c8a5b02f42d2ed30809f9d7b2d8e73a5bebc9102ef502bc8&scene=21#wechat_redirect)
    
*   [极速拿 offer：阿里 P6 被裁后极速上岸，1 个月内喜提 2 优质 offer(含滴滴)](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485452&idx=1&sn=68102559298564895b1496e2230dddb2&chksm=97b57984a0c2f0927d80f8719a7dc56f43ff13e373eb3886d39be98538180f919225b5ac05f2&scene=21#wechat_redirect)
    
*   [大逆袭：做化工 12 年，35 岁转 Java，被裁 4 个月后，1 个月收架构 Offer，大龄跨行，超级牛](http://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247496226&idx=1&sn=459a566cfa5915a720e80e1880fd730d&chksm=c14148a6f636c1b08a321d08384c5d76e184c22f8f4457f980452c45c163bdc5ab91ca17b919&scene=21#wechat_redirect)
    
*   [被裁后炸爽：10 年小伙 12 天火速上岸，反涨 20%，爽暴了](http://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247492928&idx=1&sn=82fd1c530d0ad8ad798b5c35c040641b&chksm=c1415fc4f636d6d25f16e12a73a2635073994ef742efdddcc1185c5d7727785c98fac14e8de8&scene=21#wechat_redirect)
    
*   [裁就裁，6 年小伙 60W 极速上岸，白拿 20W 还游一圈拉萨，真香](http://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247492621&idx=1&sn=f6ada03c596baedcf995e5cb60d255b2&chksm=c1415e89f636d79f13b25d7f3691fb79931515f15b421d65fab1c5e34d4cce4165b6d1772737&scene=21#wechat_redirect)
    
*   [极速拿 offer：被毕业 3 个月，11 年经验小伙 0.5 个月极速拿 offer，领路模式的巨大威力](http://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247491450&idx=1&sn=98379c07e430562a8a244121e63f47d7&chksm=c142a5fef6352ce8e3ee64090a6d0619a9d40484f358302d96990e7655ec16f8d8e7624ca88e&scene=21#wechat_redirect)
    
*   [惊天大逆袭：8 年小伙 20 天时间提 75W 年薪 offer，逆涨 50%，秘诀在这](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247491496&idx=1&sn=cb31f7510a7c2efb7daf6cad793860ad&scene=21#wechat_redirect)
    

**职业救助站**

实现职业转型，极速上岸

![](https://mmbiz.qpic.cn/mmbiz_jpg/9qY2Gt3PBs3icE6WcM4nBOdRRcczOW4wkNDgCMZhBfqeet9suwGHfgQduP33YYqqZLOppecMI72Pv5ibNkPeIk6g/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1#imgIndex=12)

关注**职业救助站**公众号，获取每天职业干货  
助您实现**职业转型、职业升级、极速上岸**  
---------------------------------

**技术自由圈**

实现架构转型，再无中年危机

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/xlgvgPaib7WMtyFcJx45oIeWL1d6AENcdoAvuTPuq46E9n8pPe4SHXoSicuEVvhibHRSIzmEBPlC7POcDaqD6ID7w/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1#imgIndex=13)

关注**技术自由圈**公众号，获取每天技术千货  
一起成为牛逼的**未来超级架构师**

**几十篇架构笔记、5000 页面试宝典、20 个技术圣经  
请加尼恩个人微信 免费拿走**

**暗号，请在 公众号后台 发送消息：领电子书**

如有收获，请点击底部的 " 在看 "和" 赞 "，谢谢