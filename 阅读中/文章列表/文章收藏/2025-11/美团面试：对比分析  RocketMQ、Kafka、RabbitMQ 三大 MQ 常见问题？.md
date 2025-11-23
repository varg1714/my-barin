---
source: https://mp.weixin.qq.com/s/UjoEW-cKn_zbZzbp3K5Mhg
create: 2025-11-18 20:55
read: true
knowledge: true
knowledge-date: 2025-11-19
tags:
  - 消息队列
summary: "[[消息队列进阶-常见消息队列的核心原理]]"
---
FSAC 未来超级架构师

架构师总动员  
实现架构转型，再无中年危机

## 尼恩说在前面

在 40 岁老架构师 尼恩的**读者交流群** (50+) 中，最近有小伙伴拿到了一线互联网企业如得物、阿里、滴滴、极兔、有赞、希音、百度、网易、美团、蚂蚁、得物的面试资格，遇到很多很重要的面试题：

如何根据应用场景选择合适的消息中间件?

Rocketmq 消息 0 丢失，如何实现？

Rocketmq 如何保证消息可靠？

对比分析  RocketMQ、Kafka、RabbitMQ 三大 MQ 常见问题?

最近有小伙伴在面试美团，遇到了相关的面试题， 小伙伴没有系统的去梳理和总结，所以支支吾吾的说了几句，面试官不满意，面试挂了。

所以，尼恩给大家做一下系统化、体系化的梳理，使得大家内力猛增，可以充分展示一下大家雄厚的 “技术肌肉”，**让面试官爱到 “不能自已、口水直流”**，然后实现”offer 直提”。

当然，这道面试题，以及参考答案，也会收入咱们的 《尼恩 Java 面试宝典 PDF》V175 版本，供后面的小伙伴参考，提升大家的 3 高 架构、设计、开发水平。

# 三大 MQ 指标对比

分布式、微服务、高并发架构中，消息队列（Message Queue，简称 MQ）扮演着至关重要的角色。

消息队列用于实现系统间的异步通信、解耦、削峰填谷等功能。

<table><thead><tr><td><span><strong><span leaf="">对比指标</span></strong></span></td><td><span><strong><span leaf="">RabbitMQ</span></strong></span></td><td><span><strong><span leaf="">RocketMQ</span></strong></span></td><td><span><strong><span leaf="">Kafka</span></strong></span></td></tr></thead><tbody><tr><td><section><span><span leaf="">应用场景</span></span></section></td><td><section><span><span leaf="">中小规模应用场景</span></span></section></td><td><section><span><span leaf="">分布式事务、实时日志处理</span></span></section></td><td><section><span><span leaf="">大规模数据处理、实时流处理</span></span></section></td></tr><tr><td><section><span><span leaf="">开发语言</span></span></section></td><td><section><span><span leaf="">Erlang</span></span></section></td><td><section><span><span leaf="">Java</span></span></section></td><td><section><span><span leaf="">Scala &amp; Java</span></span></section></td></tr><tr><td><section><span><span leaf="">消息可靠性</span></span></section></td><td><section><span><span leaf="">最高 (AMQP 协议保证)</span></span></section></td><td><section><span><span leaf="">较高 (基于事务保证)</span></span></section></td><td><section><span><span leaf="">中等 (基于副本机制保证)</span></span></section></td></tr><tr><td><section><span><span leaf="">消息吞吐量</span></span></section></td><td><section><span><span leaf="">低 万级到十万级</span></span></section></td><td><section><span><span leaf="">中等 十万级到百万级</span></span></section></td><td><section><span><span leaf="">高 百万级或更高</span></span></section></td></tr><tr><td><section><span><span leaf="">时效性</span></span></section></td><td><section><span><span leaf="">毫秒级</span></span></section></td><td><section><span><span leaf="">毫秒级</span></span></section></td><td><section><span><span leaf="">毫秒级</span></span></section></td></tr><tr><td><section><span><span leaf="">支持的语言和平台</span></span></section></td><td><section><span><span leaf="">Java、C++、Python 等</span></span></section></td><td><section><span><span leaf="">Java、C++、Go 等</span></span></section></td><td><section><span><span leaf="">Java、Scala、Python 等</span></span></section></td></tr><tr><td><section><span><span leaf="">架构模型</span></span></section></td><td><section><span><span leaf="">virtual host、broker、exchange、queue</span></span></section></td><td><section><span><span leaf="">nameserver、controller、broker</span></span></section></td><td><section><span><span leaf="">broker、topic、partition、zookeeper/Kraft</span></span></section></td></tr><tr><td><section><span><span leaf="">社区活跃度和生态建设</span></span></section></td><td><section><span><span leaf="">中等 活跃的开源社区和丰富的插件生态系统</span></span></section></td><td><section><span><span leaf="">较高 阿里巴巴开源，稳定的社区支持</span></span></section></td><td><section><span><span leaf="">最高 活跃的开源社区和广泛的应用</span></span></section></td></tr><tr><td><section><span><span leaf="">github star</span></span></section></td><td><section><span><span leaf="">10.8k</span></span></section></td><td><section><span><span leaf="">19.4k</span></span></section></td><td><section><span><span leaf="">25.2k</span></span></section></td></tr></tbody></table>

## RocketMQ、Kafka、RabbitMQ，如何选型？

RocketMQ、Kafka、RabbitMQ，如何选型？

最为详细的方案，请参考尼恩团队的架构方案： [招行面试：RocketMQ、Kafka、RabbitMQ，如何选型？](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247504454&idx=1&sn=e6feb94879bf970a85e1ccf386d8e525&scene=21#wechat_redirect)

## 对比分析三大 MQ 常见问题

下面， 对比分析三大 MQ 常见问题。

# 消息丢失问题

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WOjdicLFEjwcxOa7fLkHHk9qPQrlomAgPBLib7ibYytXessibZ3oM4T1h9EfcZY5gyT71gtbCoHsxibsiaA/640?from=appmsg#imgIndex=0)

### 1、RocketMQ 解决消息丢失问题：

‌生产端‌：  采用同步发送（等待 Broker 确认）并启用重试机制，结合事务消息（如预提交 half 消息 + 二次确认 commit）确保消息可靠投递。

‌Broker 端‌：配置同步刷盘（消息写入磁盘后返回确认）和多副本同步机制（主从节点数据冗余）防止宕机丢失，同时通过集群容灾保障高可用。

‌消费端‌：消费者需手动 ACK 确认，失败时触发自动重试（默认 16 次），最终失败消息转入死信队列人工处理，避免异常场景下消息丢失。

最为详细的方案，请参考尼恩团队的架构方案： [滴滴面试：Rocketmq 消息 0 丢失，如何实现？](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247501752&idx=1&sn=d22937adeaf3078878483a9e78cb7e2b&scene=21#wechat_redirect)

### 2、Kafka 解决消息丢失问题：

‌生产端‌：设置`acks=all`确保消息被所有副本持久化后才响应，启用生产者重试（`retries`）及幂等性（`enable.idempotence=true`）防止网络抖动或 Broker 异常导致丢失

‌Broker 端‌：配置多副本同步（`min.insync.replicas≥2`）和 ISR（In-Sync Replicas）机制，仅同步成功的副本参与选举；避免`unclean.leader.election.enable=true`（防止数据不全的副本成为 Leader）

‌消费端‌：关闭自动提交位移（`enable.auto.commit=false`），手动同步提交（`commitSync`）确保消息处理完成后再更新位移，结合消费重试及死信队列兜底

最为详细的方案，请参考尼恩团队的架构方案： [得物面试：消息 0 丢失，Kafka 如何实现？](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247501596&idx=1&sn=8d006d162646ba085b94564bb19bdfb6&scene=21#wechat_redirect)

### 3、RabbitMQ 解决消息丢失问题：

‌生产端‌：启用 Publisher Confirm 模式（异步确认消息持久化）并设置`mandatory=true`路由失败回退，结合备份交换机处理无法路由的消息；事务消息因性能损耗仅限关键场景使用。

‌Broker 端‌：消息与队列均需持久化（`durable=true`）防止宕机丢失，部署镜像队列集群实现多节点冗余；同步刷盘策略确保数据落盘后响应。

‌消费端‌：关闭自动 ACK，采用手动 ACK 并在业务处理成功后提交确认；消费失败时重试（重试次数可配置）并最终转入死信队列人工干预，避免消息因异常未处理而丢失。

# 消息积压问题

### 1、RocketMQ 解决消息积压问题：

RocketMQ 通过横向扩展（增加消费者实例、队列数量）、提升消费能力（线程池调优、批量消费）、动态扩容、消息预取、死信队列隔离无效消息，并支持消费限流及监控告警，快速定位处理积压问题。

RocketMQ 还提供了消息拉取和推拉模式，消费者可以根据自身的处理能力主动拉取消息，避免消息积压过多。

最为详细的方案，请参考尼恩团队的架构方案： [阿里面试：如何保证 RocketMQ 消息有序？如何解决 RocketMQ 消息积压？](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247500158&idx=1&sn=483783e762febb12f46b60b99ae1180b&scene=21#wechat_redirect)

### 2、Kafka 解决消息积压问题

Kafka 通过  横向扩展（增加分区及消费者实例）、优化消费者参数（如批量拉取、并发处理）、提升消费逻辑效率（异步化、减少 I/O），并动态监控消费滞后指标。

必要时限流生产者或临时扩容消费组，结合分区再平衡策略快速分发积压消息负载。

Kafka 还提供了消息清理（compaction）和数据保留策略，可以根据时间或者数据大小来自动删除过期的消息，避免消息积压过多。

### 3、RabbitMQ 解决消息积压问题

RabbitMQ 通过调整消费者的消费速率来控制消息积压。

可以使用 QoS（Quality of Service）机制设置每个消费者的预取计数，限制每次从队列中获取的消息数量，以控制消费者的处理速度。

RabbitMQ 还支持消费者端的流量控制，通过设置 basic.qos 或 basic.consume 命令的参数来控制消费者的处理速度，避免消息过多导致积压。

# 消息重复消费问题

### 1、RocketMQ 解决消息重复消费问题

*   使用消息唯一标识符（Message ID）：在消息发送时，为每条消息附加一个唯一标识符。消费者在处理消息时，可以通过判断消息唯一标识符来避免重复消费。可以将消息 ID 记录在数据库或缓存中，用于去重检查。
    
*   消费者端去重处理：消费者在消费消息时，可以通过维护一个已消费消息的列表或缓存，来避免重复消费已经处理过的消息。
    

### 2、Kafka 解决消息重复消费问题

*   幂等性处理：在消费者端实现幂等性逻辑，即多次消费同一条消息所产生的结果与单次消费的结果一致。这可以通过在业务逻辑中引入唯一标识符或记录已处理消息的状态来实现。
    
*   消息确认机制：消费者在处理完消息后，提交已消费的偏移量（Offset）给 Kafka，Kafka 会记录已提交的偏移量，以便在消费者重新启动时从正确的位置继续消费。消费者可以定期提交偏移量，确保消息只被消费一次。
    

### 3、RabbitMQ 解决消息重复消费问题

*   幂等性处理：在消费者端实现幂等性逻辑，即无论消息被消费多少次，最终的结果应该保持一致。这可以通过在消费端进行唯一标识的检查或者记录已经处理过的消息来实现。
    
*   消息确认机制：消费者在处理完消息后，发送确认消息（ACK）给 RabbitMQ，告知消息已经成功处理。RabbitMQ 根据接收到的确认消息来判断是否需要重新投递消息给其他消费者。
    

最为详细的方案，请参考尼恩团队的架构方案： [最系统的幂等性方案：一锁二判三更新](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247501086&idx=1&sn=ab6f7ca63f418b96bcd0d486a6834127&scene=21#wechat_redirect)

# 消息有序性

## 1、Rabbitmq 解决有序性问题

#### 模式一：单队列单消费者模式‌

*   将需要保证顺序的消息全部发送到‌**同一个队列**‌，且消费者设置为单线程处理。
    
*   ‌**原理**‌：RabbitMQ 队列天然支持 FIFO 顺序存储，单消费者避免并发处理导致乱序。
    

示例代：

```
// 生产者发送到同一队列
  rabbitTemplate.convertAndSend("order.queue", "message1");
  rabbitTemplate.convertAndSend("order.queue", "message2");
  // 消费者单线程监听
  @RabbitListener(queues = "order.queue")
  public void processOrder(String message) {
      // 顺序处理逻辑
  }


```

‌**缺点**‌：无法横向扩展消费者，吞吐量受限。

#### 模式二：消息分组策略‌

**按业务标识分区**‌（如订单 ID、用户 ID），相同分组的消息路由到同一队列，每个队列对应一个消费者。

实现方式： 生产者通过哈希算法或自定义路由键将关联的消息分配到特定队列。

*   生产者根据业务标识生成路由键，如 `routingKey = orderId.hashCode() % queueCount`。
    
*   声明多个队列，绑定到同一交换机，并根据路由键规则分发消息。
    

代码示例：

```
// 生产者发送消息时指定路由键
String orderId = "ORDER_1001";
String routingKey = "order." + (orderId.hashCode() % 3);  // 分配到3个队列之一
rabbitTemplate.convertAndSend("order.exchange", routingKey, message);


```

**优势**‌：在保证同分组顺序性的同时，允许不同分组并行处理。

**消费者并发控制**‌ 设置

```
     prefetchCount=1


```

确保每次只处理一个消息，关闭自动应答，手动确认后再获取新消息：

```
spring:
  rabbitmq:
         listener:
           simple:
             prefetch: 1


```

效果：防止消费者同时处理多个消息导致乱序。

## 2、RocketMQ 解决有序性问题

RocketMQ 实现顺序消息的核心是通过生产端和消费端双重保障：

*   全局顺序需单队列（性能受限），分区顺序通过 Sharding Key 哈希分散到不同队列，兼顾吞吐量与局部有序性。需避免异步消费、消息重试乱序，失败时跳过当前消息防止阻塞
    
*   生产者使用‌MessageQueueSelector‌将同一业务标识（如订单 ID）的消息强制路由至同一队列，利用队列 FIFO 特性保序；
    
*   消费端对  同一队列启用‌ 单线程拉取 + 分区锁机制‌（ConsumeOrderlyContext），确保串行处理。
    

最为详细的方案，请参考尼恩团队的架构方案：

*   [惊呆：RocketMQ 顺序消息，是 “4 把锁” 实现的（顺序消费）](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247500939&idx=1&sn=36dd5fa5f5d7d26da67410e96e098968&scene=21#wechat_redirect)
    
*   [阿里面试：如何保证 RocketMQ 消息有序？如何解决 RocketMQ 消息积压？](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247500158&idx=1&sn=483783e762febb12f46b60b99ae1180b&scene=21#wechat_redirect)
    

## 3、Kafka 解决有序性问题

Kafka 实现顺序消息的核心在于‌分区顺序性‌：‌

*   生产端‌：相同业务标识（如订单 ID）的消息通过固定 Key 哈希至同一分区（`Partitioner`），利用分区内消息天然有序性保序；‌
    
*   消费端‌：每个分区仅由同一消费者组的一个线程消费（单线程串行处理），避免并发消费乱序；
    

# 事务消息

## 1、RabbitMQ 的事务消息

*   RabbitMQ 支持事务消息的发送和确认。在发送消息之前，可以通过调用 "channel.txSelect()" 来开启事务，然后将要发送的消息发布到交换机中。如果事务成功提交，消息将被发送到队列，否则事务会回滚，消息不会被发送。
    
*   在消费端，可以通过 "channel.txSelect()" 开启事务，然后使用 "basicAck" 手动确认消息的处理结果。如果事务成功提交，消费端会发送 ACK 确认消息的处理；否则，事务回滚，消息将被重新投递。
    

```
public class RabbitMQTransactionDemo {
    private static final String QUEUE_NAME = "transaction_queue";
    public static void main(String[] args) {
        try {
            // 创建连接工厂
            ConnectionFactory factory = new ConnectionFactory();
            factory.setHost("localhost");
            // 创建连接
            Connection connection = factory.newConnection();
            // 创建信道
            Channel channel = connection.createChannel();
            // 声明队列
            channel.queueDeclare(QUEUE_NAME, false, false, false, null);
            try {
                // 开启事务
                channel.txSelect();
                // 发送消息
                String message = "Hello, RabbitMQ!";
                channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
                // 提交事务
                channel.txCommit();
            } catch (Exception e) {
                // 事务回滚
                channel.txRollback();
                e.printStackTrace();
            }
            // 关闭信道和连接
            channel.close();
            connection.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}


```

## 2、RocketMQ 的事务消息

RocketMQ 提供了事务消息的机制，确保消息的可靠性和一致性。

发送事务消息时，需要将消息发送到半消息队列，然后执行本地事务逻辑。

事务执行成功后，通过调用 "TransactionStatus.CommitTransaction" 提交事务消息；若事务执行失败，则通过调用 "TransactionStatus.RollbackTransaction" 回滚事务消息。

事务消息的最终状态由消息生产者根据事务执行结果进行确认。

```
public class RocketMQTransactionDemo {
    public static void main(String[] args) throws Exception {
        // 创建事务消息生产者
        TransactionMQProducer producer = new TransactionMQProducer("group_name");
        producer.setNamesrvAddr("localhost:9876");
        // 设置事务监听器
        producer.setTransactionListener(new TransactionListener() {
            @Override
            public LocalTransactionState executeLocalTransaction(Message msg, Object arg) {
                // 执行本地事务逻辑，根据业务逻辑结果返回相应的状态
                // 返回 LocalTransactionState.COMMIT_MESSAGE 表示事务提交
                // 返回 LocalTransactionState.ROLLBACK_MESSAGE 表示事务回滚
                // 返回 LocalTransactionState.UNKNOW 表示事务状态未知
            }
            @Override
            public LocalTransactionState checkLocalTransaction(MessageExt msg) {
                // 根据消息的状态，来判断本地事务的最终状态
                // 返回 LocalTransactionState.COMMIT_MESSAGE 表示事务提交
                // 返回 LocalTransactionState.ROLLBACK_MESSAGE 表示事务回滚
                // 返回 LocalTransactionState.UNKNOW 表示事务状态未知
            }
        });
        // 启动事务消息生产者
        producer.start();
        // 构造消息
        Message msg = new Message("topic_name", "tag_name", "Hello, RocketMQ!".getBytes());
        // 发送事务消息
        TransactionSendResult sendResult = producer.sendMessageInTransaction(msg, null);
        System.out.println("Send Result: " + sendResult);
        // 关闭事务消息生产者
        producer.shutdown();
    }
}


```

## 3、Kafka 的事务消息

Kafka 引入了事务功能来确保消息的原子性和一致性。事务消息的发送和确认在生产者端进行。

生产者可以通过初始化事务，将一系列的消息写入事务，然后通过 "commitTransaction()" 提交事务，或者通过 "abortTransaction()" 中止事务。

Kafka 会保证在事务提交之前，写入的所有消息不会被消费者可见，以保持事务的一致性。

```
public class KafkaTransactionDemo {
    public static void main(String[] args) {
        Properties props = new Properties();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        props.put(ProducerConfig.TRANSACTIONAL_ID_CONFIG, "transactional_id");
        Producer<String, String> producer = new KafkaProducer<>(props);
        // 初始化事务
        producer.initTransactions();
        try {
            // 开启事务
            producer.beginTransaction();
            // 发送消息
            ProducerRecord<String, String> record = new ProducerRecord<>("topic_name", "Hello, Kafka!");
            producer.send(record);
            // 提交事务
            producer.commitTransaction();
        } catch (ProducerFencedException e) {
            // 处理异常情况
            producer.close();
        } finally {
            producer.close();
        }
    }
}


```

# 消息确认 ACK 机制

## 1、RabbitMQ 的 ACK 机制

RabbitMQ 使用 ACK（消息确认）机制来确保消息的可靠传递。

消费者收到消息后，需要向 RabbitMQ 发送 ACK 来确认消息的处理状态。

只有在收到 ACK 后，RabbitMQ 才会将消息标记为已成功传递，否则会将消息重新投递给其他消费者或者保留在队列中。

以下是 RabbitMQ ACK 的 Java 示例：

```
public class RabbitMQAckDemo {
    public static void main(String[] args) throws Exception {
        // 创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        // 创建连接
        Connection connection = factory.newConnection();
        // 创建信道
        Channel channel = connection.createChannel();
        // 声明队列
        String queueName = "queue_name";
        channel.queueDeclare(queueName, false, false, false, null);
        // 创建消费者
        String consumerTag = "consumer_tag";
        boolean autoAck = false; // 关闭自动ACK
        // 消费消息
        channel.basicConsume(queueName, autoAck, consumerTag, new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                // 消费消息
                String message = new String(body, "UTF-8");
                System.out.println("Received message: " + message);
                try {
                    // 模拟处理消息的业务逻辑
                    processMessage(message);
                    // 手动发送ACK确认消息
                    long deliveryTag = envelope.getDeliveryTag();
                    channel.basicAck(deliveryTag, false);
                } catch (Exception e) {
                    // 处理消息异常，可以选择重试或者记录日志等操作
                    System.out.println("Failed to process message: " + message);
                    e.printStackTrace();
                    // 手动发送NACK拒绝消息，并可选是否重新投递
                    long deliveryTag = envelope.getDeliveryTag();
                    boolean requeue = true; // 重新投递消息
                    channel.basicNack(deliveryTag, false, requeue);
                }
            }
        });
    }
    private static void processMessage(String message) {
        // 模拟处理消息的业务逻辑
    }
}


```

## 2、RocketMQ 的 ACK 机制

RocketMQ 的 ACK 机制由消费者控制，消费者从消息队列中消费消息后，可以手动发送 ACK 确认消息的处理状态。

只有在收到 ACK 后，RocketMQ 才会将消息标记为已成功消费，否则会将消息重新投递给其他消费者。

```
public class RocketMQAckDemo {
    public static void main(String[] args) throws Exception {
        // 创建消费者
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("group_name");
        consumer.setNamesrvAddr("localhost:9876");
        // 订阅消息
        consumer.subscribe("topic_name", "*");
        // 注册消息监听器
        consumer.registerMessageListener((MessageListenerConcurrently) (msgs, context) -> {
            for (MessageExt message : msgs) {
                try {
                    // 消费消息
                    String msgBody = new String(message.getBody(), "UTF-8");
                    System.out.println("Received message: " + msgBody);
                    // 模拟处理消息的业务逻辑
                    processMessage(msgBody);
                    // 手动发送ACK确认消息
                    return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
                } catch (Exception e) {
                    // 处理消息异常，可以选择重试或者记录日志等操作
                    System.out.println("Failed to process message: " + new String(message.getBody()));
                    e.printStackTrace();
                    return ConsumeConcurrentlyStatus.RECONSUME_LATER;
                }
            }
            return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
        });
        // 启动消费者
        consumer.start();
    }
    private static void processMessage(String message) {
        // 模拟处理消息的业务逻辑
    }
}


```

## 3、Kafka 的 ACK 机制

Kafka 的 ACK 机制用于控制生产者在发送消息后，需要等待多少个副本确认才视为消息发送成功。

这个机制可以通过设置 acks 参数来进行配置。在 Kafka 中，acks 参数有三个可选值：

acks=0：生产者在发送消息后不需要等待任何确认，直接将消息发送给 Kafka 集群。这种方式具有最高的吞吐量，但是也存在数据丢失的风险，因为生产者不会知道消息是否成功发送给任何副本。

acks=1：生产者在发送消息后只需要等待首领副本（leader replica）确认。一旦首领副本成功接收到消息，生产者就会收到确认。这种方式提供了一定的可靠性，但是如果首领副本在接收消息后但在确认之前发生故障，仍然可能会导致数据丢失。

acks=all：生产者在发送消息后需要等待所有副本都确认。只有当所有副本都成功接收到消息后，生产者才会收到确认。这是最安全的确认机制，确保了消息不会丢失，但是需要更多的时间和资源。acks=-1 与 acks=all 是等效的。

```
public classKafkaProducerDemo{
    public static void main(String[]args){
        // 配置Kafka生产者的参数
        Propertiesprops=newProperties();
        props.put("bootstrap.servers","localhost:9092");// Kafka集群的地址和端口
        props.put("key.serializer","org.apache.kafka.common.serialization.StringSerializer");// 键的序列化器
        props.put("value.serializer","org.apache.kafka.common.serialization.StringSerializer");// 值的序列化器
        props.put("acks","all");// 设置ACK机制为所有副本都确认
        // 创建生产者实例
        KafkaProducer<String,String>producer=newKafkaProducer<>(props);
        // 构造消息
        Stringtopic="my_topic";
        Stringkey="my_key";
        Stringvalue="Hello, Kafka!";
        // 创建消息记录
        ProducerRecord<String,String>record=newProducerRecord<>(topic,key,value);
        // 发送消息
        producer.send(record,newCallback(){
            @Override
            publicvoidonCompletion(RecordMetadatametadata,Exceptionexception){
                if(exception!=null){
                    System.err.println("发送消息出现异常："+exception.getMessage());
                }else{
                    System.out.println("消息发送成功！位于分区 "+metadata.partition()+"，偏移量 "+metadata.offset());
                }
            }
        });
        // 关闭生产者
        producer.close();
    }
}


```

# 延迟消息实现

延迟队列在实际项目中有非常多的应用场景，最常见的比如订单未支付，超时取消订单，在创建订单的时候发送一条延迟消息，达到延迟时间之后消费者收到消息，如果订单没有支付的话，那么就取消订单。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WOjdicLFEjwcxOa7fLkHHk9qSJYx5kcd7pG7KcbqcCTOsVrZcEU79E7a4ctf23yuOyVO5clQDibVcTg/640?from=appmsg#imgIndex=1)

## 1、RocketMQ 实现延迟消息

RocketMQ 默认时间间隔分为 18 个级别，基本上也能满足大部分场景的需要了。

默认延迟级别：

```
1s、 5s、 10s、 30s、 1m、 2m、 3m、 4m、 5m、 6m、 7m、 8m、 9m、 10m、 20m、 30m、 1h、 2h


```

使用起来也非常的简单，直接通过`setDelayTimeLevel`设置延迟级别即可。

```
setDelayTimeLevel(level)


```

实现原理说起来比较简单，Broker 会根据不同的延迟级别创建出多个不同级别的队列，当我们发送延迟消息的时候，根据不同的延迟级别发送到不同的队列中，同时在 Broker 内部通过一个定时器去轮询这些队列（RocketMQ 会为每个延迟级别分别创建一个定时任务），如果消息达到发送时间，那么就直接把消息发送到指 topic 队列中。

RocketMQ 这种实现方式是放在服务端去做的，同时有个好处就是相同延迟时间的消息是可以保证有序性的。

谈到这里就顺便提一下关于消息消费重试的原理，这个本质上来说其实是一样的，对于消费失败需要重试的消息实际上都会被丢到延迟队列的 topic 里，到期后再转发到真正的 topic 中

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WOjdicLFEjwcxOa7fLkHHk9qWcQyu195oTcenVnCWvPLFks6O2QLxNdic3zqiarPYicZ3nklv73F0ib0AA/640?from=appmsg#imgIndex=2)

## 2、RabbitMQ 实现延迟消息

RabbitMQ 本身并不存在延迟队列的概念，在 RabbitMQ 中是通过 DLX 死信交换机和 TTL 消息过期来实现延迟队列的。

### TTL（Time to Live）过期时间

有两种方式可以设置 TTL。

**(1) 通过队列属性设置，这样的话队列中的所有消息都会拥有相同的过期时间** **(2) 对消息单独设置过期时间，这样每条消息的过期时间都可以不同**

那么如果同时设置呢？这样将会以两个时间中较小的值为准。

针对队列的方式通过参数`x-message-ttl`来设置。

```
Map<String, Object> args = new HashMap<String, Object>();
args.put("x-message-ttl", 6000);
channel.queueDeclare(queueName, durable, exclusive, autoDelete, args);


```

针对消息的方式通过`setExpiration`来设置。

```
AMQP.BasicProperties properties = new AMQP.BasicProperties();
Properties.setDeliveryMode(2);
properties.setExpiration("60000");
channel.basicPublish(exchangeName, routingKey, mandatory, properties, "message".getBytes());


```

### DLX（Dead Letter Exchange）死信交换机

一个消息要成为死信消息有 3 种情况：

**(1) 消息被拒绝，比如调用`reject`方法，并且需要设置`requeue`为`false`**

**(2) 消息过期**

**(3) 队列达到最大长度**

可以通过参数`dead-letter-exchange`设置死信交换机，也可以通过参数`dead-letter- exchange`指定 RoutingKey（未指定则使用原队列的 RoutingKey）。

```
Map<String, Object> args = new HashMap<String, Object>();
args.put("x-dead-letter-exchange", "exchange.dlx");
args.put("x-dead-letter-routing-key", "routingkey");
channel.queueDeclare(queueName, durable, exclusive, autoDelete, args);


```

### 实现原理

当我们对消息设置了 TTL 和 DLX 之后，当消息正常发送，通过 Exchange 到达 Queue 之后，由于设置了 TTL 过期时间，并且消息没有被消费（订阅的是死信队列），达到过期时间之后，消息就转移到与之绑定的 DLX 死信队列之中。

这样的话，就相当于通过 DLX 和 TTL 间接实现了延迟消息的功能，实际使用中我们可以根据不同的延迟级别绑定设置不同延迟时间的队列来达到实现不同延迟时间的效果。

如果队列通过 dead-letter-exchange 属性指定了一个交换机，那么该队列中的死信就会投递到这个交换机中，这个交换机称为死信交换机（Dead Letter Exchange，简称 DLX）

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WOjdicLFEjwcxOa7fLkHHk9qxeYr1g5gareOZuayQg8VoEqzvfaia9Rhr5CpPAkx4XTstyTmFlleaibA/640?from=appmsg#imgIndex=3)

## 3、Kafka 实现延迟消息

对于 Kafka 来说，原生并不支持延迟队列的功能，需要我们手动去实现，这里我根据 RocketMQ 的设计提供一个实现思路。

这个设计，我们也不支持任意时间精度的延迟消息，只支持固定级别的延迟，因为对于大部分延迟消息的场景来说足够使用了。

只创建一个 topic，但是针对该 topic 创建 18 个 partition，每个 partition 对应不同的延迟级别，这样做和 RocketMQ 一样有个好处就是能达到相同延迟时间的消息达到有序性。

### 应用级 Kafka 延迟消息实现原理

首先创建一个单独针对延迟队列的 topic，同时创建 18 个 partition 针对不同的延迟级别

发送消息的时候根据延迟参数发送到延迟 topic 对应的 partition，对应的`key`为延迟时间，同时把原 topic 保存到 header 中

```
ProducerRecord<Object, Object> producerRecord = new ProducerRecord<>("delay_topic", delayPartition, delayTime, data);
producerRecord.headers().add("origin_topic", topic.getBytes(StandardCharsets.UTF_8));


```

内嵌的`consumer`单独设置一个`ConsumerGroup`去消费延迟 topic 消息，消费到消息之后如果没有达到延迟时间那么就进行`pause`，然后`seek`到当前`ConsumerRecord`的`offset`位置，同时使用定时器去轮询延迟的`TopicPartition`，达到延迟时间之后进行`resume`

如果达到了延迟时间，那么就获取到`header`中的真实 topic ，直接转发

这里为什么要进行`pause`和`resume`呢？

因为如果不这样的话，如果超时未消费达到`max.poll.interval.ms`最大时间（默认 300s），那么将会触发 Rebalance。

## 遇到问题，找老架构师取经

借助此文，尼恩给解密了一个高薪的 秘诀，大家可以 放手一试。**保证  屡试不爽，涨薪  100%-200%。**

后面，尼恩 java 面试宝典回录成视频， 给大家打造一套进大厂的塔尖视频。

通过这个问题的深度回答，可以充分展示一下大家雄厚的 “技术肌肉”，**让面试官爱到 “不能自已、口水直流”**，然后实现”offer 直提”。

在面试之前，建议大家系统化的刷一波 5000 页《[尼恩 Java 面试宝典 PDF](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247497474&idx=1&sn=54a7b194a72162e9f13695443eabe186&scene=21#wechat_redirect)》，里边有大量的大厂真题、面试难题、架构难题。

很多小伙伴刷完后， 吊打面试官， 大厂横着走。

在刷题过程中，如果有啥问题，大家可以来 找 40 岁老架构师尼恩交流。

另外，如果没有面试机会，**可以找尼恩来改简历、做帮扶。**

遇到职业难题，找老架构取经， 可以省去太多的折腾，省去太多的弯路。

尼恩指导了大量的小伙伴上岸，前段时间，**[刚指导一个  32 岁 高中生，冲大厂成功 , 年薪 50W，逆天改命 ！！！。](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486808&idx=1&sn=3fe07ce65a7e2fb26c432b0d71e86bf1&scene=21#wechat_redirect)**

狠狠卷，实现 “offer 自由” 很容易的， 前段时间一个武汉的跟着尼恩卷了 2 年的小伙伴， 在极度严寒 / 痛苦被裁的环境下， offer 拿到手软， 实现真正的 “offer 自由” 。

## 冲大厂 案例：  全网顶尖、高薪案例， 进大厂拿高薪， 实现薪酬腾飞、人生逆袭 

[阿里 + 美团 offer：25 岁 屡战屡败 绝望至极。找尼恩转架构升级，1 个月拿到阿里 + 美团 offer，逆天改命年薪 50W](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486794&idx=1&sn=fc1a1f843b0a91b61f7124ac0c2b51e2&scene=21#wechat_redirect)

  

[阿里 offer：6 年一本 不想 混小厂了。狠卷 1 年  拿到 得物 + 阿里 offer ， 彻底上岸 ，逆天改命](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486782&idx=1&sn=52ed05742c8afe7abf3b50405e78c853&scene=21#wechat_redirect)

[字节 offer：3 年经验 ， 足足折腾 1 年后绝望了，找尼恩陪跑， 2 个月 逆天改命 ，拿到 字节 & 携程  offer](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486772&idx=1&sn=35e2a2a3be26da241229332e44337f7c&scene=21#wechat_redirect)

小米 offer：[7 年 普通小二本，冲 小米 成功，年薪 60W， 吊打一大票 985/211，狠卷 1 年，足足涨了 50%，](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486743&idx=1&sn=e45c86053f87fbf3c0a80b24f44eaf36&scene=21#wechat_redirect)人生逆袭

[拼多多 offer：2 年经验 60W 破全网记录，顶尖案例](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486581&idx=1&sn=00e8d8d91978f993e0d5d5d757fd555c&scene=21#wechat_redirect)，**[2 年经验年薪 60W](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486581&idx=1&sn=00e8d8d91978f993e0d5d5d757fd555c&scene=21#wechat_redirect)**[，3 大厂 offer， 双非一本 秒杀 985、秒杀 211，逆天改命](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486581&idx=1&sn=00e8d8d91978f993e0d5d5d757fd555c&scene=21#wechat_redirect)

# [美团 offer：被裁后涨 80%，大赚了。从小厂被裁，拿 4 大厂 offer（申通、顺丰、携程、美团），大涨 80%，26 岁小伙被裁赚翻了](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486469&idx=1&sn=02fa589a575551c427de3b277629bf4b&scene=21#wechat_redirect)

[逆天大涨：暴涨 200%，29 岁 / 7 年 / 双非一本 ， 从 13K 涨到 37K ，如何做到的？](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486433&idx=1&sn=0f630e8fc7620b3bf0e054827b48d91e&scene=21#wechat_redirect)

[逆天改命：27 岁被裁 2 月，转 P6 降维攻击，2 个月提 JD/PDD 两大 offer，时来运转，人生翻盘!! 大逆袭!!](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486421&idx=1&sn=02a88c23c8fe689a662214d5297f88a0&scene=21#wechat_redirect)

[急救上岸：29 岁（golang）被裁 3 月，转架构降维打击，收 3 个大厂 offer， 年薪 60W，逆天改命](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486392&idx=1&sn=16c34d7f960805f659adfb27676ee2de&scene=21#wechat_redirect)

## 大龄逆袭的案例： 大龄被裁，快速上岸的，远离没有 offer 的焦虑、恐慌 

[47 岁超级大龄，被裁员后 找尼恩辅导收  2 个 offer，一个 40 多 W。 35 岁之后，只要 技术好，还是有饭吃，关键是找对方向，找对路子](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486323&idx=1&sn=a088584f97b657add9a058c149b72067&scene=21#wechat_redirect)

[大龄不难：39 岁 / 15 年老码农，15 天时间 40W 上岸，管一个 team，不用去 铁人三项了！](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486310&idx=1&sn=505e479da54cf1365afd4d39bddc18bd&scene=21#wechat_redirect)

# [武汉收 5 个 offer：34 岁的 CRUD 小伙被裁， 尼恩陪跑 12 天， 收 5 个 offer，拿到 30K，逆涨 7K，逆天改命](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486568&idx=1&sn=7de41b66dbd116600679a6927f37b7f4&scene=21#wechat_redirect)

# [上岸奇迹：中厂大龄 34 岁，被裁 8 月收一大厂 offer， 年薪 65W，转架构后逆天改命!](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486405&idx=1&sn=02336456cf4a09268c142c0b13327682&scene=21#wechat_redirect)

## 100W 年薪 天花板 案例 ，他们 如何 实现薪酬腾飞、人生逆袭？ 

# **[年薪 100W 的底层逻辑：](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486315&idx=1&sn=52cc0a640c1499115f5a45b82097d857&chksm=97b57ae3a0c2f3f502293e84cde4bc892c45d06dee3579471a522ab497fb53e32640fb00c339&scene=21#wechat_redirect)** [大厂被裁，他们两个，如何实现年薪百万？](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486315&idx=1&sn=52cc0a640c1499115f5a45b82097d857&chksm=97b57ae3a0c2f3f502293e84cde4bc892c45d06dee3579471a522ab497fb53e32640fb00c339&scene=21#wechat_redirect) 

**[年薪 100W](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485942&idx=1&sn=faa999fc9508435db7a88af2716492d9&chksm=97b5787ea0c2f168ac80d344670df16be12614d37161a893ebed88d4c915384fcbc024996403&scene=21#wechat_redirect)****[：](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485942&idx=1&sn=faa999fc9508435db7a88af2716492d9&chksm=97b5787ea0c2f168ac80d344670df16be12614d37161a893ebed88d4c915384fcbc024996403&scene=21#wechat_redirect)**[40](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485942&idx=1&sn=faa999fc9508435db7a88af2716492d9&chksm=97b5787ea0c2f168ac80d344670df16be12614d37161a893ebed88d4c915384fcbc024996403&scene=21#wechat_redirect)[岁小伙，被裁 6 个月，猛卷 3 月，100W 逆袭 ，秘诀：升级首席架构 / 总架构](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485942&idx=1&sn=faa999fc9508435db7a88af2716492d9&chksm=97b5787ea0c2f168ac80d344670df16be12614d37161a893ebed88d4c915384fcbc024996403&scene=21#wechat_redirect)

[最新的 100W 案例：环境太糟，如何升 P8 级，年入 100W？](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486369&idx=1&sn=0b32e8907dbcf26eaf33ea3f37ddd099&chksm=97b57a29a0c2f33f468d29dafe739204543873efb459f324871c3c68a3ecc178b1a9ab08a2fa&scene=21#wechat_redirect)

**职业救助站**

实现职业转型，极速上岸

![](https://mmbiz.qpic.cn/mmbiz_jpg/9qY2Gt3PBs3icE6WcM4nBOdRRcczOW4wkNDgCMZhBfqeet9suwGHfgQduP33YYqqZLOppecMI72Pv5ibNkPeIk6g/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1#imgIndex=4)

关注**职业救助站**公众号，获取每天职业干货  
助您实现**职业转型、职业升级、极速上岸**  
---------------------------------

**技术自由圈**

实现架构转型，再无中年危机

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/xlgvgPaib7WMtyFcJx45oIeWL1d6AENcdoAvuTPuq46E9n8pPe4SHXoSicuEVvhibHRSIzmEBPlC7POcDaqD6ID7w/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1#imgIndex=5)

关注**技术自由圈**公众号，获取每天技术千货  
一起成为牛逼的**未来超级架构师**

**几十篇架构笔记、5000 页面试宝典、20 个技术圣经  
请加尼恩个人微信 免费拿走**

**暗号，请在 公众号后台 发送消息：领电子书**

如有收获，请点击底部的 " 在看 "和" 赞 "，谢谢