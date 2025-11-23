---
source: https://mp.weixin.qq.com/s/j2dWEk4JVPToZBQ4v7orTg
create: 2025-11-18 21:01
read: true
knowledge: true
knowledge-date: 2025-11-20
tags:
  - 消息队列
summary: "[[消息队列进阶-常见消息队列的核心原理]]"
---
FSAC 未来超级架构师

架构师总动员  
实现架构转型，再无中年危机

## 尼恩说在前面

在 40 岁老架构师 尼恩的**读者交流群** (50+) 中，最近有小伙伴拿到了一线互联网企业如得物、阿里、滴滴、极兔、有赞、希音、百度、网易、美团、蚂蚁、得物的面试资格，遇到很多很重要的面试题：

Rocketmq 消息 0 丢失，如何实现？

Rocketmq 如何保证消息可靠？

**最近有小伙伴在面试滴滴，都到了相关的面试题，可以说是逢面必问。**

小伙伴没有系统的去梳理和总结，所以支支吾吾的说了几句，面试官不满意，面试挂了。

所以，尼恩给大家做一下系统化、体系化的梳理，使得大家内力猛增，可以充分展示一下大家雄厚的 “技术肌肉”，**让面试官爱到 “不能自已、口水直流”**，然后实现”offer 直提”。

当然，这道面试题，以及参考答案，也会收入咱们的 《尼恩 Java 面试宝典 PDF》V175 版本，供后面的小伙伴参考，提升大家的 3 高 架构、设计、开发水平。

《尼恩 架构笔记》《尼恩高并发三部曲》《尼恩 Java 面试宝典》的 PDF，请到文末公号【技术自由圈】获取

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WO8Tic04SfDar9dQ7xVI7b3DQViaxDqZCoV9UJr0X9GjMialiaHuKU6bbbedKcby1OCOTIRxwPExlSibGQ/640?wx_fmt=png&from=appmsg#imgIndex=0)

## 本文目录

****-** 尼恩说在前面**

******-**** 消息的发送流程**

 - 生产阶段如何实现 0 丢失方式

 - 是同步发送还是异步发送

 - 生产端的失败重试策略

********-****** Broker 端保证消息不丢失的方法：**

 - Broker 端第一板斧：设置严格的副本同步机制

 - Broker 端第二板斧：设置严格的消息刷盘机制

#####  - 如何设置 RocketMQ 同步刷盘？

 - Broker 端 0 丢失的配置总结

**********-******** Consumer（消费者）保证消息不丢失的方法：**

**********-******** 1、同步消费发送 CONSUME_SUCCESS**

**********-******** 2、异步消费场景，如何保证 0 丢失**

**********-******** 业务维度的 终极 0 丢失保护措施：本地消息表 + 定时扫描**

#####  - 本地消息表 + 定时扫描 方案，和本地消息表事务机制类似，也是采用 本地消息表 + 定时扫描 相结合的架构方案。

************-************ RocketMQ 的 0 丢失的最佳实践

************-************ 说在最后：有问题找老架构取经

## 消息的发送流程

Rocketmq 和 KafKa 类似（实质上，最早的 Rocketmq 就是 KafKa 的 Java 版本），一条消息从生产到被消费，将会经历三个阶段：

*   生产阶段，Producer 新建消息，而后经过网络将消息投递给 MQ Broker。这个发送可能会发生丢失，比如网络延迟不可达等。
    
*   存储阶段，消息将会存储在 Broker 端磁盘中，Broker 根据刷盘策略持久化到硬盘中，刚收到 Producer 的消息在内存中了，但是如果 Broker 异常宕机了，导致消息丢失。
    
*   消费阶段， Consumer 将会从 Broker 拉取消息
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WO8Tic04SfDar9dQ7xVI7b3DB71jYzm7puyMWFzQSvGCV8tXF9nf2icfk2g6FdXv2dopZ9Gk4HWQwaQ/640?wx_fmt=png&from=appmsg#imgIndex=1)

以上任一阶段, 都可能会丢失消息，只要这三个阶段 0 丢失，就能够完全解决消息丢失的问题。

宏观层面的大的阶段和流程，Rocketmq 和 KafKa 类似的。KafKa 零丢失，具体的文章：

[得物面试：消息 0 丢失，Kafka 如何实现？](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247501596&idx=1&sn=8d006d162646ba085b94564bb19bdfb6&scene=21#wechat_redirect)

### 生产阶段如何实现 0 丢失方式

生产阶段有三种 send 方法:

*   同步发送
    
*   异步发送
    
*   单向发送。
    

三种 send 方法的 客户端 api，具体如下：

```
/**
 * {@link org.apache.rocketmq.client.producer.DefaultMQProducer}
 */

// 同步发送
public SendResult send(Message msg) throws MQClientException, RemotingException,      MQBrokerException, InterruptedException {}

// 异步发送，sendCallback作为回调
public void send(Message msg,SendCallback sendCallback) throws MQClientException, RemotingException, InterruptedException {}

// 单向发送，不关心发送结果，最不靠谱
public void sendOneway(Message msg) throws MQClientException, RemotingException, InterruptedException {}


```

produce 要想发消息时保证消息不丢失，可以采用同步发送的方式去发消息，send 消息方法只要不抛出异常，就代表发送成功。

发送成功会有多个 SendResult 状态，以下对每个状态进行说明：

*   SEND_OK：消息发送成功，Broker 刷盘、主从同步成功
    
*   FLUSH_DISK_TIMEOUT：消息发送成功，但是服务器同步刷盘（默认为异步刷盘）超时（默认超时时间 5 秒）
    
*   FlUSH_SLAVE_TIMEOUT：消息发送成功，但是服务器同步复制（默认为异步复制）到 Slave 时超时（默认超时时间 5 秒）
    
*   SLAVE_NOT_AVAILABLE：Broker 从节点不存在
    

注意：同步发送只要返回以上四种状态，就代表该消息在生产阶段消息正确的投递到了 RocketMq，生产阶段没有丢失。

**如果业务要求严格，可以使用同步发送，并且只取 SEND_OK 标识消息发送成功，**

其他返回值类型的数据，采用 40 岁老架构师尼恩给大家设计的，在业务维度的 终极 0 丢失保护措施：本地消息表 + 定时扫描 （具体参见本文末尾）

### 是同步发送还是异步发送

根据尼恩的架构设计 40 个黄金法则 ，AP 和 CP 是天然的矛盾， 到底是 CP 还是 AP 的 需要权衡，

[一张图总结架构设计的 40 个黄金法则](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247501074&idx=1&sn=24378954dcfa7ede84394892e7f5bb38&scene=21#wechat_redirect)

*   同步发送的方式 是 CP ，高可靠，但是性能低。
    
*   异步发送的方式 是 AP ，低可靠，但是性能高。
    

为了高可靠（CP），可以采取同步发送的方式进行发送消息，发消息的时候会同步阻塞等待 broker 返回的结果，如果没成功，则不会收到 SendResult，这种是最可靠的。

其次是异步发送，再回调方法里可以得知是否发送成功。

最后，单向发送（OneWay）是最不靠谱的一种发送方式，我们无法保证消息真正可达。

当然，具体的如何选择高可用方案，还是要看业务。

为了确保万无一失，可以选择异步发送 + 业务维度的 终极 0 丢失保护措施 ， 实现消息的 0 丢失。

### 生产端的失败重试策略

发送消息如果失败或者超时了，则会自动重试。

同步发送默认是重试三次，可以根据 api 进行更改，比如改为 10 次：

```
producer.setRetryTimesWhenSendFailed(10);


```

其他模式是重试 1 次，具体请参见源码

```java
/**
 * {@link org.apache.rocketmq.client.producer.DefaultMQProducer#sendDefaultImpl(Message, CommunicationMode, SendCallback, long)}
 */

// 自动重试次数，this.defaultMQProducer.getRetryTimesWhenSendFailed()默认为2，如果是同步发送，默认重试3次，否则重试1次
int timesTotal = communicationMode == CommunicationMode.SYNC ? 1 + this.defaultMQProducer.getRetryTimesWhenSendFailed() : 1;

int times = 0;
for (; times < timesTotal; times++) {
      // 选择发送的消息queue
    MessageQueue mqSelected = this.selectOneMessageQueue(topicPublishInfo, lastBrokerName);
    if (mqSelected != null) {
        try {
            // 真正的发送逻辑，sendKernelImpl。
            sendResult = this.sendKernelImpl(msg, mq, communicationMode, sendCallback, topicPublishInfo, timeout - costTime);
            switch (communicationMode) {
                case ASYNC:
                    return null;
                case ONEWAY:
                    return null;
                case SYNC:
                    // 如果发送失败了，则continue，意味着还会再次进入for，继续重试发送
                    if (sendResult.getSendStatus() != SendStatus.SEND_OK) {
                        if (this.defaultMQProducer.isRetryAnotherBrokerWhenNotStoreOK()) {
                            continue;
                        }
                    }
                    // 发送成功的话，将发送结果返回给调用者
                    return sendResult;
                default:
                    break;
            }
        } catch (RemotingException e) {
            continue;
        } catch (...) {
            continue;
        }
    }
}


```

上面的核心逻辑中，调用 sendKernelImpl 真正的去发送消息

通过核心的发送逻辑，可以看出如下：

*   同步发送场景的重试次数是 1 + this.defaultMQProducer.getRetryTimesWhenSendFailed() =3，其他方式默认 1 次。
    
*   this.defaultMQProducer.getRetryTimesWhenSendFailed() 默认是 2，我们可以手动设置`producer.setRetryTimesWhenSendFailed(10);`
    
*   如果是同步发送 sync，且发送失败了，则 continue，意味着还会再次进入 for，继续重试发送
    

同步模式下，可以设置严格的消息重试机制，比如设置 RetryTimes 为一个较大的值如 10。当出现网络的瞬时抖动时，消息发送可能会失败，retries 较大，能够自动重试消息发送，避免消息丢失。

## Broker 端保证消息不丢失的方法：

首先，尼恩想说正常情况下，只要 Broker 在正常运行，就不会出现丢失消息的问题。

但是如果 Broker 出现了故障，比如进程死掉了或者服务器宕机了，还是可能会丢失消息的。

如果确保万无一失，实现 Broker 端保证消息不丢失，有两板斧：

*   **Broker 端第一板斧：设置严格的副本同步机制**
    
*   **Broker 端第二板斧：设置严格的消息刷盘机制**
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WO8Tic04SfDar9dQ7xVI7b3DouicQVSLdt7qDqf7fMlDCQog1dxeNbo5GAKxEd2Q987y976kVnxE3Zg/640?wx_fmt=png&from=appmsg#imgIndex=2)

### Broker 端第一板斧：设置严格的副本同步机制

RocketMQ 通过多副本机制来解决的高可用，核心思想也挺简单的：如果数据保存在一台机器上你觉得可靠性不够，那么我就把相同的数据保存到多台机器上，某台机器宕机了可以由其它机器提供相同的服务和数据。

首先，Broker 需要集群部署，通过主从模式包括 topic 数据的高可用。

为了消息 0 丢失，可以配置设置严格的副本同步机制，等 Master 把消息同步给 Slave 后，才去通知 Producer 说消息 ok。

设置严格的副本同步机制 ， RocketMQ 修改 broker 刷盘配置如下：

所以我们还可以配置不仅是等 Master 刷完盘就通知 Producer，而是等 Master 和 Slave 都刷完盘后才去通知 Producer 说消息 ok 了。

```
## 默认为 ASYNC_MASTER
brokerRole=SYNC_MASTER


```

### Broker 端第二板斧：设置严格的消息刷盘机制

RocketMQ 持久化消息分为两种：同步刷盘和异步刷盘。

RocketMQ 和 kafka 一样的，刷盘的方式有**同步刷盘和异步刷盘**两种。

*   同步刷盘指的是：生产者消息发过来时，只有持久化到磁盘，RocketMQ、kafka 的存储端 Broker 才返回一个成功的 ACK 响应，这就是同步刷盘。它保证消息不丢失，但是影响了性能。
    
*   异步刷盘指的是：消息写入 PageCache 缓存，就返回一个成功的 ACK 响应，不管消息有没有落盘，就返回一个成功的 ACK 响应。这样提高了 MQ 的性能，但是如果这时候机器断电了，就会丢失消息。
    

同步刷盘和异步刷盘的区别如下:

*   同步刷盘: 当数据写如到内存中之后立刻刷盘 (同步)，在保证刷盘成功的前提下响应 client。
    
*   异步刷盘: 数据写入内存后，直接响应 client。异步将内存中的数据持久化到磁盘上。
    

同步刷盘和异步输盘的优劣:

*   同步刷盘保证了数据的可靠性, 保证数据不会丢失。
    
*   同步刷盘效率较低, 因为 client 获取响应需要等待刷盘时间，为了提升效率，通常采用批量输盘的方式，每次刷盘将会 flush 内存中的所有数据。(若底层的存储为 mmap，则每次刷盘将刷新所有的 dirty 页)
    
*   异步刷盘不能保证数据的可靠性.
    
*   异步刷盘可以提高系统的吞吐量.
    
*   常见的异步刷盘方式有两种, 分别是定时刷盘和触发式刷盘。定时刷盘可设置为如每 1s 刷新一次内存. 触发刷盘为当内存中数据到达一定的值，会触发异步刷盘程序进行刷盘。
    

Broker 端第二板斧：设置严格的消息刷盘机制，设置为 Kafka 同步刷盘。

RocketMQ 默认情况是异步刷盘，Broker 收到消息后会先存到 cache 里，然后通知 Producer 说消息我收到且存储成功。Broker 起个线程异步的去持久化到磁盘中，但是 Broker 还没持久化到磁盘就宕机的话，消息就丢失了。

同步刷盘的话是收到消息存到 cache 后并不会通知 Producer 说消息已经 ok 了，而是会等到持久化到磁盘中后才会通知 Producer 说消息完事了。

*   同步刷盘的方式 是 CP ，高可靠，但是性能低。
    
*   异步刷盘的方式 是 AP ，低可靠，但是性能高。
    

根据尼恩的架构设计 40 个黄金法则 ，AP 和 CP 是天然的矛盾， 到底是 CP 还是 AP 的 需要权衡，

[一张图总结架构设计的 40 个黄金法则](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247501074&idx=1&sn=24378954dcfa7ede84394892e7f5bb38&scene=21#wechat_redirect)

为了高可靠（CP），可以采取同步刷盘保障了消息不会丢失，但是性能不如异步高。

#### 如何设置 RocketMQ 同步刷盘？

RocketMQ 修改 broker 刷盘配置如下：

```
## 默认情况为 ASYNC_FLUSH，修改为同步刷盘：SYNC_FLUSH，实际场景看业务，同步刷盘效率肯定不如异步刷盘高。
flushDiskType = SYNC_FLUSH


```

对应的 RocketMQ 源码类如下：

```
package org.apache.rocketmq.store.config;

public enum FlushDiskType {
    // 同步刷盘
    SYNC_FLUSH,
    // 异步刷盘（默认）
    ASYNC_FLUSH
}


```

异步刷盘默认 10s 执行一次，源码如下：

```
/*
 * {@link org.apache.rocketmq.store.CommitLog#run()}
 */

while (!this.isStopped()) {
    try {
        // 等待10s
        this.waitForRunning(10);
        // 刷盘
        this.doCommit();
    } catch (Exception e) {
        CommitLog.log.warn(this.getServiceName() + " service has exception. ", e);
    }
}


```

### Broker 端 0 丢失的配置总结

Broker 端的配置，若想很严格的保证 Broker 存储消息阶段消息不丢失，则需要如下配置

```
# master 节点配置
flushDiskType = SYNC_FLUSH
brokerRole=SYNC_MASTER

# slave 节点配置
brokerRole=slave
flushDiskType = SYNC_FLUSH


```

上面这个配置含义是：

Producer 发消息到 Broker 后，Broker 的 Master 节点先持久化到磁盘中，然后同步数据给 Slave 节点，Slave 节点同步完且落盘完成后才会返回给 Producer 说消息 ok 了。

严格的消息刷盘机制 + 严格的消息同步机制，能够确保 Broker 端保证消息不丢失

当然，根据尼恩的架构设计 40 个黄金法则 ，AP 和 CP 是天然的矛盾， 到底是 CP 还是 AP 的 需要权衡，

[一张图总结架构设计的 40 个黄金法则](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247501074&idx=1&sn=24378954dcfa7ede84394892e7f5bb38&scene=21#wechat_redirect)

## Consumer（消费者）保证消息不丢失的方法：

如果要保证 Consumer（消费者）0 丢失， Consumer 端的策略是啥呢？

普通的情况下，rocketMq 拉取消息后，执行业务逻辑。

一旦 Consumer 执行成功，将会返回一个 ACK 响应给 Broker，这时 MQ 就会修改 offset，将该消息标记为已消费，不再往其他消费者推送消息。

如果出现消费超时 (默认 15 分钟)、拉取消息后消费者服务宕机等消费失败的情况，此时的 Broker 由于没有等到消费者返回的 ACK，会向同一个消费者组中的其他消费者间隔性的重发消息，直到消息返回成功（默认是重复发送 16 次，若 16 次还是没有消费成功，那么该消息会转移到死信队列, 人工处理或是单独写服务处理这些死信消息）

但是 消费者，也有两种消费模式：

*   同步消费，消费线程完成业务操作
    
*   异步消息 ，  独立业务线程池 完成业务操作
    

一般大家在 rocketMq 在并发消费模式下，这个模式，默认有 20 个消费线程：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WO8Tic04SfDar9dQ7xVI7b3Dv9mOXG6EODtfmpmCSrv9EsUgd09aSrVcibUtyY61ECFvia1aGeUFKJKA/640?wx_fmt=png&from=appmsg#imgIndex=3)

关于 Rocketmq 的消息消费的核心架构，请参见尼恩的硬核文章：

[阿里面试：说说 Rocketmq 推模式、拉模式？](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247500993&idx=1&sn=2683f34077a268a9d8d3a3e239a09b7b&scene=21#wechat_redirect)

如何保证客户端的高可用，两种场景：

*   同步消费场景，业务代码手动发送 CONSUME_SUCCESS ，保证 消息者的 0 丢失
    
*   异步消费场景，需要通过业务维度的 终极 0 丢失保护措施：本地消息表 + 定时扫描 ，保证 消息者的 0 丢失
    

## 1、同步消费发送 CONSUME_SUCCESS

同步消费指的是拉取消息的线程，先把消息拉取到本地，然后进行业务逻辑，业务逻辑完成后手动进行 ack 确认，这时候才会真正的代表消费完成。举个例子

```
 consumer.registerMessageListener(new MessageListenerConcurrently() {
     @Override
     public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext consumeConcurrentlyContext) {
         for (MessageExt msg : msgs) {
             String str = new String(msg.getBody());
             
             // 消费者 线程 同步进行  业务处理
             System.out.println(str);
         }
         // ack，只有等上面一系列逻辑都处理完后，
         // 发 CONSUME_SUCCESS才会通知broker说消息消费完成，
         // 如果上面发生异常没有走到这步ack，则消息还是未消费状态。
         return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
     }
 });


```

## 2、异步消费场景，如何保证 0 丢失

rocketMq 在并发消费模式下，默认有 20 个消费线程，但是这个还是有限制的。

如果实现高性能呢？

*   可以一方面去进行线程扩容， 比如通过修改配置，扩容到 100 个 rocketMq 同步消费线程，但是这个会在没有 活儿干的场景，浪费宝贵的 CPU 资源。
    
*   可以另一方便通过异步的 可动态扩容的业务专用线程池，去完成 消费的业务处理。那么，如果是设置了业务的专用线程池，则需要通过业务维度的 终极 0 丢失保护措施：本地消息表 + 定时扫描 ，保证 消息者的 0 丢失
    

关于可动态扩容的业务专用线程池，请参考尼恩的文章：

[阿里面试：系统的最佳线程数，怎么确定？](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247501663&idx=1&sn=e5bb9970efe3e0a69d79c8c976b4bbc7&scene=21#wechat_redirect)

## 业务维度的 终极 0 丢失保护措施：本地消息表 + 定时扫描

40 岁老架构师尼恩慎重提示：前面三板斧，并不能保证 100% 的 0 丢失。

因为百密一疏，任何环节的异常，都有可能导致数据丢失。

有没有业务维度的 终极保护措施呢？有：本地消息表 + 定时扫描

##### 本地消息表 + 定时扫描 方案，和本地消息表事务机制类似，也是采用 本地消息表 + 定时扫描 相结合的架构方案。

大概流程如下图

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WO8Tic04SfDar9dQ7xVI7b3DbSgVBBzyLIiam49k9xNyJf4CKnibibiaicd00o4tKUKicNsFibp9FIcmTib8NA/640?wx_fmt=png&from=appmsg#imgIndex=4)

1、设计一个本地消息表，可以存储在 DB 里，或者其它存储引擎里，用户保存消息的消费状态

2、Producer 发送消息之前，首先保证消息的发生状态，并且初始化为待发送；

3、如果消费者（如库存服务）完成的消费，则通过 RPC，调用 Producer 去更新一下消息状态；

4、Producer 利用定时任务扫描 过期的消息（比如 10 分钟到期），再次进行发送。

在这里尼恩想说的是：本地消息表 + 定时扫描 的架构方案 ，是业务层通过额外的机制来保证消息数据发送的完整性，是一种很重的方案。这个方案的两个特点：

*   CP 不是 AP，性能低
    
*   需要 做好幂等性设计
    

如果降低业务维度的 终极 0 丢失保护措施带来的性能耗损？

可以减少本地消息表的规模，对于正常投递的消息不做跟踪，只把生产端发送失败的消息、消费端消费失败的消息记录到数据库，并启动一个定时任务，扫描发送失败的消息，重新发送直到超过阈值（如 10 次），超过之后，发送邮件或短信通知人工介入处理。

CP 不是 AP 的 需要权衡，请参见全网最好的架构设计个黄金法则，尼恩的 专门文章具体如下：

[一张图总结架构设计的 40 个黄金法则](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247501074&idx=1&sn=24378954dcfa7ede84394892e7f5bb38&scene=21#wechat_redirect)

全网最好的幂等性 方案，请参见尼恩的 专门文章， 具体如下：

[最系统的幂等性方案：一锁二判三更新](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247501086&idx=1&sn=ab6f7ca63f418b96bcd0d486a6834127&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WO8Tic04SfDar9dQ7xVI7b3D6YOcRlXicXYKRPyc4TdVsIbuGic1tSUo2213Ubtrk2b5akR15SE01VXQ/640?wx_fmt=png&from=appmsg#imgIndex=5)

## RocketMQ 的 0 丢失的最佳实践

1.  Producer 端：使用同步发送方式发送消息，可以提高可靠性。
    
    记住，如果使用带有回调通知的 send 异步 方法发送去提高性能，则需要结合 本地消息表 + 定时扫描 的架构，去实现业务维度的 高可靠。
    
2.  Producer 端：同步模式下，可以设置严格的消息重试机制，比如设置 RetryTimes 为一个较大的值如 10。当出现网络的瞬时抖动时，消息发送可能会失败，retries 较大，能够自动重试消息发送，避免消息丢失。。
    
3.  Broker 端：设置严格的副本同步机制 。
    
4.  Broker 端：设置严格的消息刷盘机制。
    
5.  Consumer 端：确保消息消费完成再提交。可以使用同步消费，并发送 CONSUME_SUCCESS。
    
6.  业务维度的的 0 丢失架构：采用 本地消息表 + 定时扫描 架构方案，实现业务维度的 0 丢失，100% 可靠性。
    

如上，就是尼恩为大家梳理的，史上最牛掰的 答案， 全网最为爆表的方案。按照尼恩的套取去回到， 面试官一定惊到掉下巴。offer 直接奉上。此答案大家可以收藏一起，有时间看看。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WO8Tic04SfDar9dQ7xVI7b3DzQ9E27Pp4UtBcF9UDYkibtGxFeLmibiajjSdOqKNkQPDibYfUTGpMxcZKg/640?wx_fmt=png&from=appmsg#imgIndex=6)

## 说在最后：有问题找老架构取经

Rocketmq 消息 0 丢失，如何实现？，如果大家能对答如流，如数家珍，基本上 面试官会被你 震惊到、吸引到。

最终，**让面试官爱到 “不能自已、口水直流”**。offer， 也就来了。

在面试之前，建议大家系统化的刷一波 5000 页《[尼恩 Java 面试宝典 PDF](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247497474&idx=1&sn=54a7b194a72162e9f13695443eabe186&scene=21#wechat_redirect)》，里边有大量的大厂真题、面试难题、架构难题。很多小伙伴刷完后， 吊打面试官， 大厂横着走。

在刷题过程中，如果有啥问题，大家可以来 找 40 岁老架构师尼恩交流。

另外，如果没有面试机会，**可以找尼恩来改简历、做帮扶。**

遇到职业难题，找老架构取经， 可以省去太多的折腾，省去太多的弯路。

尼恩指导了大量的小伙伴上岸，前段时间，**刚指导一个 40 岁 + 被裁小伙伴，拿到了一个年薪 100W 的 offer。**

狠狠卷，实现 “offer 自由” 很容易的， 前段时间一个武汉的跟着尼恩卷了 2 年的小伙伴， 在极度严寒 / 痛苦被裁的环境下， offer 拿到手软， 实现真正的 “offer 自由” 。

部分历史案例

*   [35 岁被裁 6 个月 职业绝望，转架构急救上岸，DDD 和 3 高项目太重要了](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486098&idx=1&sn=bbc5732b071477573bfab8a259d208d3&chksm=97b57b1aa0c2f20c27dd74b490b6062c9eec262a3ac25548534ff70290b172dcc51c6eafe532&scene=21#wechat_redirect)
    
*   [逆天改命，3 年经验 2 本，卷 3 个月涨薪 60%，进准大厂 (得物等)，年薪 36W](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486077&idx=1&sn=090ad8d19559c727b80e321cd70e3c32&chksm=97b57bf5a0c2f2e32a982b62cfeb2db8744260d7e2d80efdf11ad7c1c5257057ea9b1f7419ec&scene=21#wechat_redirect)
    
*   [女 35 岁 CRUD 程序媛，12 月被裁后，拿 30 个机会 2 个 offer 上岸，年底大裁员太不容易了](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486059&idx=1&sn=23fa04e6ea781525732f021ab1afaeeb&scene=21#wechat_redirect)
    
*   [38 还有救吗？有。42 岁被裁 2 年天快塌了，急救 1 个月上岸，成开发经理 offer，起死回生](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486028&idx=1&sn=cf10ecda7a986b26e0359eba7a9cfeff&scene=21#wechat_redirect)
    
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

![](https://mmbiz.qpic.cn/mmbiz_jpg/9qY2Gt3PBs3icE6WcM4nBOdRRcczOW4wkNDgCMZhBfqeet9suwGHfgQduP33YYqqZLOppecMI72Pv5ibNkPeIk6g/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1#imgIndex=7)

关注**职业救助站**公众号，获取每天职业干货  
助您实现**职业转型、职业升级、极速上岸**  
---------------------------------

**技术自由圈**

实现架构转型，再无中年危机

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/xlgvgPaib7WMtyFcJx45oIeWL1d6AENcdoAvuTPuq46E9n8pPe4SHXoSicuEVvhibHRSIzmEBPlC7POcDaqD6ID7w/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1#imgIndex=8)

关注**技术自由圈**公众号，获取每天技术千货  
一起成为牛逼的**未来超级架构师**

**几十篇架构笔记、5000 页面试宝典、20 个技术圣经  
请加尼恩个人微信 免费拿走**

**暗号，请在 公众号后台 发送消息：领电子书**

如有收获，请点击底部的 " 在看 "和" 赞 "，谢谢