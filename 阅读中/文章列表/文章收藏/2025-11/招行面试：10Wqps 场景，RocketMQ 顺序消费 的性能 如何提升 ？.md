---
source: https://mp.weixin.qq.com/s/zIntV5H-DDcFjvUCzmE1LA
create: 2025-11-18 20:55
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

在 45 岁老架构师 尼恩的**读者交流群** (50+) 中，最近有小伙伴拿到了一线互联网企业如得物、阿里、滴滴、极兔、有赞、希音、百度、网易、美团、蚂蚁、得物的面试资格，遇到很多很重要的相关面试题：

问题 ：10Wqps 高并发，如何提升 RocketMQ 顺序消费性能? 该考虑哪些问题?

**最近有小伙伴面试招行， 问到了相关的面试题。**

小伙伴没有系统的去梳理和总结，所以支支吾吾的说了几句，面试官不满意，面试挂了。

所以，尼恩给大家做一下系统化、体系化的梳理，使得大家内力猛增，可以充分展示一下大家雄厚的 “技术肌肉”，**让面试官爱到 “不能自已、口水直流”**，然后实现”offer 直提”。

当然，这道面试题，以及参考答案，也会收入咱们的 《尼恩 Java 面试宝典 PDF》V175 版本，供后面的小伙伴参考，提升大家的 3 高 架构、设计、开发水平。

《尼恩 架构笔记》《尼恩高并发三部曲》《尼恩 Java 面试宝典》的 PDF，请到文末公号【技术自由圈】获取

## 招商银行的高阶 Java 后端面试真题

被狠狠拷打了，问的人都懵了。项目场景题太难了，不好好准备，真的答不出!

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/xlgvgPaib7WOALBYxl8uPWFhqia5ulFB0zrwwy7U0VOSQAD97ral8hJInficR8YID8KDPicIcovNzkibLAx8PYfb1AQ/640?wx_fmt=other&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1#imgIndex=0)

## 尼恩将给出全部答案：

**1. 如何让系统抗住双十一的预约抢购活动?**

[45 岁资深老架构师尼恩的参考答案，点此查看](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247502814&idx=1&sn=722d69d8eae6a9de60cf0dc3acaadd93&scene=21#wechat_redirect)

**2. 如何从零搭建 10 万级 QPS 大流量、高并发优惠券系统?**

[45 岁资深老架构师尼恩的参考答案，点此查看](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247502893&idx=1&sn=b39b320390580b04c0261083e84dbb11&scene=21#wechat_redirect)

**3. 百万级别数据的 Excel 如何快速导入到数据**

[45 岁资深老架构师尼恩的参考答案，点此查看](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247504374&idx=1&sn=1a542e126b5388fd90e20930ba2ecb88&scene=21#wechat_redirect)

**4. 如何设计一个支持万亿 GB 网盘实现秒传与限速的系统?**

[45 岁资深老架构师尼恩的参考答案，点此查看](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247504394&idx=1&sn=97b9e628d05147dc9b1bfed9a6e986b8&scene=21#wechat_redirect)

**5. 如何根据应用场景选择合适的消息中间件?**

[45 岁资深老架构师尼恩的参考答案，点此查看](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247504454&idx=1&sn=e6feb94879bf970a85e1ccf386d8e525&scene=21#wechat_redirect)

**6. 如何提升 RocketMQ 顺序消费性能?**

本文

**7. 设计分布式调度框架，该考虑哪些问题?**

[45 岁资深老架构师尼恩的参考答案，点此查看](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247504465&idx=1&sn=2bb789691a64c4de1100669ca89136a9&scene=21#wechat_redirect)

**9. 如何让系统抗住双十一的预约抢购活动?**

[45 岁资深老架构师尼恩的参考答案，点此查看](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247504421&idx=1&sn=75a7a954dd3aa59315d0186778d8bd93&scene=21#wechat_redirect)

**10. 问 ：如何解决高并发下的库存抢购超卖少买？**

[45 岁资深老架构师尼恩的参考答案，点此查看](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247504476&idx=1&sn=7829d7ddf5010b55322589d83848b803&scene=21#wechat_redirect)

**11. 为什么高并发下数据写入不推荐关系数据？**

[45 岁资深老架构师尼恩的参考答案，点此查看](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247504446&idx=1&sn=419d0cd86f6a08c083f04f0ab555b29d&scene=21#wechat_redirect)

**12. 如果让你设计一个分布式链路跟踪系统?**

即将发布。

前几天 尼恩给一个 小伙伴改造过一个 100wtps 链路跟踪平台简历， 非常 NB， 牛到暴表。

## 本文目录

****-**** **尼恩说在前面**

****-** 招商银行的高阶 Java 后端面试真题**

****-** 尼恩将给出全部答案：**

****-** 本文目录**

****-** 1：专用方案：顺序消息 的 专用调优方案**

 - 1.1 简介：顺序消息 的 2 大专用的高并发调优方案

 - 1.2 详解调优方案 1：如何进行 顺序消息 的 topic  优化？

 - 1.3 分区有序消息：

 - 1.4 如何设置更多 ConsumeQueue？

****-**** **2：通用方案：RocketMQ 的通用调优方案** 

****-** 2.1 硬件配置调优** 

 - 2.1.1：  CPU 的调优

 - 2.1.2：  内存 的调优

 - 2.1.3：  磁盘 的调优

 - 2.1.4：  网络 的调优

****-**** **2.2 操作系统调优**

 - 2.2.1 内存与交换优化

 - 2.2.2 文件系统与磁盘 I/O 优化

 - 2.2.3 优化磁盘 I/O 调度算法：

 - 2.2.4 网络参数调优：

****-**** **3：RocketMQ 配置优化**

****-** 3.1 Broker 配置优化**

 - 3.1.1  调整文件存储配置：

 - 3.1.2  调整 `mappedFileSizeCommitLog`， 减少文件切换频率

****-**** **3.2 Broker 线程池大小调整**

 - 3.2.1 调整发送消息线程池（SendMessageThreadPool）

 - 3.2.2 拉取消息线程池（PullMessageThreadPool）

****-**** **3.3 消息写入与刷盘策略优化**

****-** 3.4 NameServer 配置优化**

****-** 4：客户端配置优化**

 - 4.1 调整客户端重试策略：

 - 4.2 使用异步发送模式：

****-**** **5：JVM 调优：**

****-** 6：监控与调优**

****-** 7：日志分析**

****-** 尼恩架构团队的塔尖 MQ 面试题，熟读 10 遍进大厂**

******-**** 说在最后：有问题找老架构取经‍**

## 1：专用方案：顺序消息 的 专用调优方案

首先，讲讲 顺序消息专用 的高并发调优方案。

### 1.1 简介：顺序消息 的 2 大专用的高并发调优方案

**调优方案 1：topic 设计优化**：

rocketmq 为 分区内有序， 一个顺序的 topic， 设置更多的 consumequeue。

consumequeue 越多， 顺序消息 的 并发量越高。

**调优方案 2：message 设计优化**：

*   精简消息属性，只保留必要的信息，减少消息大小。
    
*   在应用层使用 Gzip 或 Snappy 等压缩算法对消息体进行压缩，减少消息体占用空间。
    

### 1.2 详解调优方案 1：如何进行 顺序消息 的 topic 优化？

**首先回顾一下 ，什么是 顺序消息？**

RocketMQ 中的顺序消息是指对于同一个消息分区（通常是同一个 Topic 下的一个逻辑分区），消息的消费顺序与发送顺序相同。

**然后回顾一下 ，什么是 顺序消息 的 分区 有序？**

对于顺序消息，RocketMQ 保证了同一分区内的消息顺序，不同分区的消息则不保证顺序。

这样可以在一定程度上兼顾顺序性和高并发性能。

例如，对于订单业务，同一个订单的创建、支付、发货等消息可以发送到同一个分区，以保证这些消息的顺序消费，而不同订单的消息可以在不同分区并行处理。

### 1.3 分区有序消息：

对于指定的一个 Topic ，所有消息根据 Sharding Key 进行区块分区，同一个分区内的消息按照严格的先进先出（FIFO）原则进行发布和消费。

同一分区内的消息保证顺序，不同分区之间的消息顺序不做要求。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WNQKU5IsmOsI8bhacEWrMBmA9LEf5vqD862VZynyosxF7R430Lmzy72mkW0uyr1Aaib1bxXBqSld2Q/640?wx_fmt=png&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1#imgIndex=1)

**分区有序适用场景**：适用于性能要求高，以 Sharding Key 作为分区字段，在同一个区块中严格地按照先进先出（FIFO）原则进行消息发布和消费的场景。

**示例**：电商的订单创建，以订单 ID 作为 Sharding Key ，那么同一个订单相关的创建订单消息、订单支付消息、订单退款消息、订单物流消息都会按照发布的先后顺序来消费。

**最后回顾一下 ，什么是 RocketMQ 中 的 ConsumeQueue ？**

ConsumeQueue 是 RocketMQ 中的消息索引文件，它存储了消息在 CommitLog 中的位置信息，是消息消费的关键索引。

当消费者拉取消息时，会先访问 ConsumeQueue，根据 ConsumeQueue 中的索引信息到 CommitLog 中获取消息。

每个 Topic 下可以有多个 ConsumeQueue，不同的 ConsumeQueue 可以映射到不同的分区，这样可以将消息的存储和消费负载分散，提高并发处理能力。

设置更多的 ConsumeQueue 提高并发量的思路， 大致如下：

**顺序消息的专用优化手段：增加 ConsumeQueue。ConsumeQueue 越多， 顺序消息 的 并发量越高。**

1.  **设置更多的 ConsumeQueue 提高并行度**：
    
    通过增加一个顺序 Topic 下的 ConsumeQueue 数量，可以提高该 Topic 的并行消费能力。
    
    因为每个 ConsumeQueue 可以由不同的消费者组中的消费者并行消费，当有更多的 ConsumeQueue 时，理论上可以分配给更多的消费者进行并行消费，从而提高整个 Topic 的并发处理能力。
    
2.  **设置更多的 ConsumeQueue, 实现更好的 负载均衡**：
    
    更多的 ConsumeQueue 可以更精细地进行负载均衡。
    
    在 RocketMQ 的集群环境中，当消费者组中的多个消费者订阅一个 Topic 时，它们会根据分配策略分配到不同的 ConsumeQueue 进行消费，更多的 ConsumeQueue 意味着可以更均匀地分配负载，避免部分消费者过载而部分消费者空闲的情况。
    

### 1.4 如何设置更多 ConsumeQueue？

在 RocketMQ 中设置更多 ConsumeQueue 的具体操作步骤：

创建 Topic 时设置更多的 ConsumeQueue 数量

*   方式一：可以 代码来创建 Topic 并指定 ConsumeQueue 数量。
    
*   方式二：使用命令行工具（mqadmin）指定 ConsumeQueue 数量。
    

**方式一：使用命令行工具（mqadmin）指定 ConsumeQueue 数量， 具体的方法如下：**

```
sh mqadmin updateTopic -n localhost:9876 -t OrderTopic -c DefaultCluster -r 8 -w 8


```

上述命令中，

`-n localhost:9876` 指定 NameServer 的地址，

`-t OrderTopic` 是要创建或更新的 Topic 名称，

`-r 8` 表示读队列数（通常等同于 ConsumeQueue 数量），

`-w 8` 表示写队列数。

这里将读队列数和写队列数都设置为 8，即设置了 8 个 ConsumeQueue。

**方式二：通过 Java 代码创建 Topic 并设置 ConsumeQueue 数量：**

```
import org.apache.rocketmq.client.exception.MQClientException;
import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.common.admin.TopicConfig;
import org.apache.rocketmq.remoting.RPCHook;
import org.apache.rocketmq.tools.admin.DefaultMQAdminExt;
import org.apache.rocketmq.tools.admin.MQAdminExt;
import org.apache.rocketmq.tools.command.CommandUtil;

import java.util.Set;

public class CreateTopicWithMoreConsumeQueue {
    public static void main(String[] args) throws Exception {
        MQAdminExt admin = new DefaultMQAdminExt();
        admin.start();

        String topicName = "OrderTopic";
        int queueNum = 8; // 设置 ConsumeQueue 数量为 8

        TopicConfig topicConfig = new TopicConfig(topicName);
        topicConfig.setReadQueueNums(queueNum);
        topicConfig.setWriteQueueNums(queueNum);

        Set<String> masterSet = CommandUtil.fetchMasterAddrByClusterName(admin, "DefaultCluster");
        for (String master : masterSet) {
            admin.createAndUpdateTopicConfig(master, topicConfig);
        }

        admin.shutdown();
    }
}


```

上述代码使用 `DefaultMQAdminExt` 创建一个名为 `OrderTopic` 的 Topic，并将读队列数和写队列数都设置为 8，即设置了 8 个 ConsumeQueue。

## 2：通用方案：RocketMQ 的通用调优方案

然后 ，讲讲 RocketMQ 通用 高并发调优方案。

RocketMQ 是一款高性能、高可靠的分布式消息中间件，在高并发场景下，通过多方面的调优，可以显著提升其性能和稳定性。

可以从硬件配置、操作系统、RocketMQ 配置、消息存储设计、 监控与调优等方面进行详细阐述 。

## 2.1 硬件配置调优

### 2.1.1：CPU 的调优

**选择高性能的多核 CPU**：

选用高主频、8 核或以上的 CPU，例如 Intel Xeon Platinum 系列。

在实际应用中，8 核 CPU 相较于 4 核 CPU，在高并发消息处理场景下，吞吐量可提升 30% - 50% 。

**避免 CPU 过度争用**：

通过 CPU 绑定技术，将 RocketMQ 的 Broker 和 NameServer 进程固定到特定的 CPU 核心上，减少线程调度和上下文切换开销。

可以使用`taskset`命令进行 CPU 绑定，如`taskset -p 0x1 <pid>`将进程`<pid>`绑定到 CPU 0 。

### 2.1.2：内存 的调优

配置 32GB 或以上的内存，如在一个消息量较大的电商订单处理系统中，将内存从 16GB 提升到 32GB 后，消息处理延迟降低了约 20% 。

### 2.1.3：磁盘 的调优

**使用高性能的 SSD 磁盘**：

采用 SSD 磁盘，其随机读写性能远高于传统机械硬盘。

在一个实时数据采集系统中，使用 SSD 后，消息写入延迟降低了约 70% 。

**合理配置 RAID**：

对于数据可靠性要求高的场景，使用 RAID 1，兼顾性能和可靠性；

对于追求极致性能的场景，可使用 RAID 0，但需做好数据备份。

### 2.1.4：网络 的调优

**高带宽低延迟网络**：

配置 10Gb 或更高带宽的网络，减少网络传输时延。

在一个跨地域的分布式系统中，将网络带宽从 1Gb 提升到 10Gb 后，消息传输性能提升了约 80% 。

## 2.2 操作系统调优

### 2.2.1 内存与交换优化

**锁定 RocketMQ 进程内存**：

在`/etc/security/limits.conf`中设置：

```
* soft memlock unlimited
* hard memlock unlimited


```

**禁用交换（swap）**：

在`/etc/sysctl.conf`中设置`vm.swappiness = 0`，然后执行`sysctl -p` 。

### 2.2.2 文件系统与磁盘 I/O 优化

**选择合适的文件系统**：

推荐使用`xfs`文件系统，在大文件写入和高并发写入场景下性能更优。

**调整文件描述符限制**：

在`/etc/security/limits.conf`中设置：

```
* soft nofile 65536
* hard nofile 65536


```

### 2.2.3 优化磁盘 I/O 调度算法：

使用`deadline` 调度算法，如`echo deadline > /sys/block/sdX/queue/scheduler`（`sdX`为磁盘设备名）。

在操作系统中，磁盘 I/O 调度算法用于管理对磁盘设备的读写请求顺序，以提高磁盘的性能和效率。

不同的调度算法适用于不同的场景，对于 RocketMQ 这种依赖磁盘存储的系统，选择合适的调度算法可以优化磁盘 I/O 性能。

**deadline 调度算法**：

*   `deadline` 调度算法旨在避免请求饥饿现象。它为每个请求设置了两个队列：读请求队列和写请求队列，同时为每个队列设置了超时时间。
    
*   读请求的超时时间通常比写请求短，因为读操作往往需要更快的响应时间，以避免阻塞进程。
    
*   当一个请求在其超时时间内未被处理时，会优先处理该请求，而不管其他请求的顺序，这样可以防止某些请求因大量其他请求的存在而被长时间延迟，保证了一定的响应性和公平性。
    

**deadline 适用场景**：

适用于对读写延迟有一定要求的场景，尤其是读操作需要及时响应的情况，如实时数据处理系统，像 RocketMQ 这种需要频繁读写消息的系统，使用 `deadline` 可以避免某些读写请求被长时间阻塞，提高系统的响应性能。

**noop 调度算法**：

*   `noop` 是一种简单的调度算法，它对请求进行简单的先入先出（FIFO）处理，基本不做额外的排序或优化操作。
    
*   它将请求直接发送到磁盘，适合于某些特定的存储设备，如使用了设备自身硬件缓存和优化机制的 SSD 或其他高级存储设备。
    

**noop 调度算法适用场景**：

对于使用了高级存储设备（如 SSD 或 RAID 控制器带有自己的智能调度和缓存机制）的系统，`noop` 调度算法可以避免操作系统额外的调度操作，让存储设备自身的优化机制发挥最大作用，提高性能。

### 2.2.4 网络参数调优：

在 Linux 系统的`/etc/sysctl.conf`中配置 TCP 参数，如：

```
net.core.somaxconn = 1024
net.ipv4.tcp_max_syn_backlog = 2048
net.ipv4.tcp_fin_timeout = 15
net.ipv4.tcp_keepalive_time = 300



```

然后执行`sysctl -p`使配置生效。

这些参数的调整有助于优化网络性能，提高 RocketMQ 集群在高并发场景下的网络传输性能和稳定性。

它们可以减少网络延迟、提高网络资源的利用效率以及避免因网络连接处理能力不足而导致的各种性能问题。

```
net.core.somaxconn = 1024


```

`net.core.somaxconn` 是 Linux 内核参数，它定义了系统中每一个端口监听队列的最大长度。

这个参数影响着服务器接收新的 TCP 连接请求的能力。当服务器接收到客户端的连接请求时，这些请求会被放入一个监听队列中等待服务器的处理。

如果这个队列已满，新的连接请求可能会被拒绝。将其设置为 1024 意味着监听队列最多可以容纳 1024 个未处理的连接请求。

对于像 RocketMQ 这样的分布式系统，在高并发情况下，可能会有大量的客户端同时尝试连接 Broker 或 NameServer，适当增大该值可以避免连接请求被拒绝，从而提高系统的并发连接处理能力。

```
net.ipv4.tcp_max_syn_backlog = 2048


```

`net.ipv4.tcp_max_syn_backlog` 主要与 TCP 三次握手的过程相关。

在 TCP 连接建立的初始阶段，客户端发送一个 `SYN` 包给服务器，服务器会将该请求存储在 `SYN` 队列中，直到完成三次握手。这个参数指定了 `SYN` 队列的最大长度。

将其设置为 2048 可以允许服务器存储更多的 `SYN` 请求，防止在高并发环境下，服务器无法接收新的 `SYN` 请求而导致客户端连接失败。

例如，当大量客户端同时发起连接请求时，如果 `SYN` 队列过小，部分客户端可能会收到 `ECONNREFUSED` 错误。增大这个值可以提高服务器在高并发连接场景下的抗压力，避免出现连接超时或失败的情况。

```
net.ipv4.tcp_fin_timeout = 15


```

`net.ipv4.tcp_fin_timeout` 表示在 TCP 连接中，当一端发起关闭连接请求（发送 `FIN` 包）后，等待对方确认关闭的超时时间，单位是秒。

设置为 15 秒意味着，如果在这个时间内没有收到对方的 `ACK` 确认关闭消息，系统将强制关闭连接。缩短这个时间可以更快地释放 TCP 连接资源，减少连接处于 `FIN_WAIT_2` 状态的时间，从而使服务器能够更快地处理新的连接。

在高并发环境下，大量的连接可能处于 `FIN_WAIT_2` 状态，这会占用系统资源，通过缩短 `tcp_fin_timeout` 可以提高系统资源的利用率和处理新连接的速度。

```
net.ipv4.tcp_keepalive_time = 300


```

`net.ipv4.tcp_keepalive_time` 用于控制 TCP 连接的保活时间。当一个 TCP 连接处于空闲状态（没有数据传输）时，服务器会定期发送 `keep-alive` 包来检测连接是否仍然有效，这个参数指定了发送 `keep-alive` 包的时间间隔，单位是秒。

将其设置为 300 秒表示如果一个 TCP 连接在 300 秒内没有数据传输，服务器会发送 `keep-alive` 包。

这样可以及时发现并关闭失效的连接，释放系统资源，避免因长时间的空闲连接占用资源而影响系统性能，特别是在分布式消息中间件中，有助于清理长时间未使用的连接，确保系统资源得到有效利用。

注意: 参数的具体设置需要根据系统的实际情况和性能需求进行调整，并非越大越好，过大的参数值可能会导致系统资源的浪费或其他潜在问题。

例如，如果 `net.core.somaxconn` 设置过大，可能会占用过多的系统内存，而 `tcp_keepalive_time` 过短可能会导致频繁发送 `keep-alive`包，增加网络开销。

## 3：RocketMQ 配置优化

## 3.1 Broker 配置优化

### 3.1.1 调整文件存储配置：

将`storePathRootDir`和`storePathCommitLog`设置为不同的磁盘路径，减少磁盘 I/O 争用。

配置方法：在 `broker.conf` 中添加或修改以下行：

```
storePathRootDir=/path/to/root/dir
storePathCommitLog=/path/to/commitlog/dir


```

确保 `/path/to/root/dir` 和 `/path/to/commitlog/dir` 是不同的磁盘路径。

### 3.1.2 调整 mappedFileSizeCommitLog， 减少文件切换频率。

根据磁盘性能和数据量调整`mappedFileSizeCommitLog`，如设置为 1GB 或 2GB，减少文件切换频率。

配置项 `mappedFileSizeCommitLog` 决定了 CommitLog 文件的大小。

CommitLog 文件 越大， 可以减少文件切换的频率，为啥呢 ？文件切换涉及文件的创建和关闭等操作，会带来额外的 I/O 开销。

根据磁盘性能和消息量，可以将其设置为 1GB 或 2GB 等大小。

例如，

*   对于高性能的 SSD 且消息量较大的场景，设置为 2GB 可以减少文件切换次数，提升写入性能；
    
*   对于磁盘性能稍弱的场景，设置为 1GB 可能更合适。
    

配置方法, 在 `broker.conf` 中添加或修改以下行：

```
mappedFileSizeCommitLog=1073741824  // 1GB


```

或者

```
mappedFileSizeCommitLog=2147483648  // 2GB


```

### 3.2 Broker 线程池大小调整

**调整线程池大小**：

根据硬件资源和业务负载，合理调整线程池大小，避免线程过多导致的 CPU 争用和上下文切换开销。

以下是 Broker 线程池大小调整的详细步骤和解释：

**评估硬件资源：**

首先需要了解服务器的硬件资源，特别是 CPU 核心数和内存大小。一般来说，线程池大小应该根据 CPU 核心数来合理设置，避免过多的线程导致 CPU 资源的过度竞争和上下文切换开销。通常，线程池大小可以设置为 CPU 核心数的 1 到 2 倍，但这只是一个大致的参考，具体数值需要根据实际性能测试来确定。

**分析业务负载：**

考虑消息的发送和接收频率，以及 Broker 处理的任务类型和复杂度。如果业务负载较高，可能需要更多的线程来处理请求；如果处理的任务相对简单，可以适当减少线程池大小。

**监控性能指标：**

在调整线程池大小前后，需要监控 CPU 使用率、线程上下文切换频率、消息处理延迟、吞吐量等性能指标，以便确定调整是否有效。

### 3.2.1 调整发送消息线程池（SendMessageThreadPool）

调整发送消息线程池大小：可以在 `broker.conf` 中添加或修改以下参数来调整发送消息的线程池大小：

```
sendMessageThreadPoolNums=200


```

这里将发送消息的线程池大小设置为 200， 可以根据实际情况调整该值。

sendMessageThreadPool 负责将接收到的消息存储到 CommitLog 中，包括消息的序列化、存储文件的分配和写入等操作。

sendMessageThreadPool 主要用于处理生产者发送消息的请求。当生产者向 Broker 发送消息时，这些请求会被分配到发送消息线程池中的线程进行处理。

### 3.2.2 拉取消息线程池（PullMessageThreadPool）

调整拉取消息线程池大小, 可修改：

```
pullMessageThreadPoolNums=150


```

pullMessageThreadPool 线程池负责处理消费者的拉取消息请求。当消费者从 Broker 拉取消息时，拉取请求会被分配到该线程池中的线程进行处理。

它会根据消费者的拉取请求，从存储（如 CommitLog 和 ConsumeQueue）中查找并提取消息，然后发送给消费者。

## 3.3 消息写入与刷盘策略优化

`flushDiskType` 的两大刷盘模式：

*   同步刷盘（SYNC_FLUSH）
    

同步刷盘意味着每次消息写入内存缓冲区后，会立即将数据刷盘到磁盘，确保消息存储的可靠性。在消息存储成功之前，会阻塞消息发送操作，这样可以保证数据不会因系统崩溃而丢失，但会严重影响写入性能，适用于对数据可靠性要求极高的场景，如金融系统中的重要交易数据。

*   异步刷盘（ASYNC_FLUSH）：
    

如前面所述，异步刷盘将消息先存储在内存缓冲区，然后在适当的时候批量刷盘到磁盘，不阻塞消息发送操作，提高了写入性能，但在系统崩溃时可能会丢失部分未刷盘的数据。

将`flushDiskType`设置为`ASYNC_FLUSH`，提高写入性能，但数据可靠性降低，适用于对数据可靠性要求不高的场景。

将`flushDiskType`设置为`ASYNC_FLUSH`的配置方法 ：

在 `broker.conf` 中修改 `flushDiskType` 来选择刷盘策略 ，在 Broker 的配置文件（通常是 broker.conf）中添加或修改以下行：

配置方法：在 Broker 的配置文件（通常是 `broker.conf`）中添加或修改以下行：

```
flushDiskType=SYNC_FLUSH  // 同步刷盘


```

改成

```
flushDiskType=ASYNC_FLUSH  // 异步刷盘


```

## 3.4 NameServer 配置优化

**优化 NameServer 的高可用性**：

部署多个 NameServer 实例，分布在不同的物理节点上，确保在单点故障时系统仍然可用。

**合理配置 NameServer 的心跳和路由更新频率**：

在`broker.conf`中调整参数：

```
brokerHeartbeatInterval=10000  // 心跳间隔，单位为毫秒
brokerTimeout=30000           // Broker超时时间，单位为毫秒


```

以下是对这两个 RocketMQ 配置参数的详细解释：

**brokerHeartbeatInterval=10000 含义**：

`brokerHeartbeatInterval` 表示 Broker 向 NameServer 发送心跳消息的时间间隔，单位是毫秒。在 RocketMQ 的架构中，Broker 是消息存储和转发的核心组件，而 NameServer 是路由中心，负责管理 Broker 和客户端的路由信息。

这里将 `brokerHeartbeatInterval` 设置为 10000 毫秒（即 10 秒），意味着 Broker 会每隔 10 秒向 NameServer 发送一次心跳消息，以告知 NameServer 自身仍然存活，并且更新自身的状态信息（如存储的 Topic 信息、队列信息等）。

**brokerHeartbeatInterval 作用和影响**：

*   确保 NameServer 能及时掌握 Broker 的最新状态，使得客户端在获取路由信息时，能得到最新的信息，保证消息的正确路由。如果心跳间隔过长，可能导致 NameServer 上的 Broker 信息更新不及时，当 Broker 发生故障时，客户端可能会将消息发送到已经失效的 Broker 上，造成消息发送失败。
    
*   适当的心跳间隔可以平衡网络和服务器资源的使用。如果心跳间隔过短，会导致频繁的网络请求，增加网络负载和 Broker 以及 NameServer 的处理开销；而如果间隔过长，会影响路由信息的更新及时性。对于大规模的 RocketMQ 集群，合理设置这个值可以避免不必要的网络开销，同时保证路由信息的更新速度能满足系统正常运行的需要。
    

**brokerTimeout=30000 含义**：

`brokerTimeout` 定义了 NameServer 判断 Broker 是否超时的时间，单位是毫秒。当 NameServer 在 `brokerTimeout` 时间内没有收到 Broker 的心跳消息时，会认为该 Broker 已经失效。

这里将 `brokerTimeout` 设置为 30000 毫秒（即 30 秒），如果 NameServer 超过 30 秒没有收到来自某个 Broker 的心跳，就会将该 Broker 标记为不可用，并将其从路由信息中移除。

**brokerTimeout 作用和影响**：

*   此参数有助于提高 RocketMQ 集群的容错性。当 Broker 出现故障或网络故障导致心跳中断时，NameServer 可以在一定时间后发现并将其移除，避免将消息路由到故障的 Broker 上，保证消息发送和消费的可用性。
    
*   设置过长的超时时间可能会导致系统在 Broker 失效后，仍然将消息发送到该 Broker 一段时间，造成消息发送延迟和潜在的消息丢失；设置过短的超时时间可能会导致误判，例如在网络抖动或 Broker 短暂延迟时，将正常的 Broker 误判为失效，影响系统的正常运行。
    

## 4：客户端配置优化

### 4.1 调整客户端重试策略：

在 Java 代码中设置：

```
producer.setRetryTimesWhenSendFailed(3);  // 设置发送失败时的重试次数
producer.setRetryAnotherBrokerWhenNotStoreOK(true);  // 发送失败时尝试其他Broker


```

### 4.2 使用异步发送模式：

```
producer.send(message, new SendCallback() {
    @Override
    public void onSuccess(SendResult sendResult) {
        // 处理发送成功的逻辑
    }

    @Override
    public void onException(Throwable e) {
        // 处理发送失败的逻辑
    }
});


```

## 5：JVM 调优：

调整 JVM 堆大小和垃圾回收器参数，例如：

```
-Xms4g -Xmx4g -XX:+UseG1GC -XX:MaxGCPauseMillis=200 -XX:G1HeapRegionSize=16M


```

`-Xms4g -Xmx4g` 设置堆的初始大小和最大大小为 4GB ；

`-XX:+UseG1GC`使用 G1 垃圾回收器；

`-XX:MaxGCPauseMillis=200` 设置最大垃圾回收停顿时间为 200 毫秒；

`-XX:G1HeapRegionSize=16M` 设置 G1 堆区域大小为 16MB 。

## 6：监控与调优

**使用 RocketMQ Console 监控**：

部署 RocketMQ Console，通过 Web 界面监控集群的运行状态，包括 Broker、NameServer、Topic、Consumer Group 等信息。

**集成第三方监控工具**：

将 RocketMQ 的性能指标集成到 Prometheus、Grafana 等第三方监控工具中，设置报警规则。

## 7：日志分析

**启用慢查询日志**：

在`broker.conf`中启用慢查询日志，如`traceTopicEnable=true`，分析日志找出性能瓶颈。

**分析 Broker 和 Consumer 的日志**：

定期检查日志文件，排查异常和错误信息。

## 尼恩架构团队的塔尖 MQ 面试题，**熟读  10 遍 ** 进大厂 

[面试：RocketMQ、Kafka、RabbitMQ，如何选型？](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247504454&idx=1&sn=e6feb94879bf970a85e1ccf386d8e525&scene=21#wechat_redirect)

[滴滴面试：Rocketmq 消息 0 丢失，如何实现？](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247501752&idx=1&sn=d22937adeaf3078878483a9e78cb7e2b&scene=21#wechat_redirect)

[顺序消息 底层原理：RocketMQ 顺序消息，是 “4 把锁” 实现的](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247500939&idx=1&sn=36dd5fa5f5d7d26da67410e96e098968&scene=21#wechat_redirect)

[阿里面试：canal+MQ，会有乱序的问题吗？](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247502825&idx=1&sn=2314fc816e0774e0c3e622e4c0e92673&scene=21#wechat_redirect)

[网易一面：单节点 2000Wtps，Kafka 怎么做的？](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247496807&idx=1&sn=b18c379fcbb3eed2bcd95adfa19beebf&scene=21#wechat_redirect)

  

## 说在最后：有问题找老架构取经‍

只要按照上面的 尼恩团队梳理的 方案去作答， 你的答案不是 100 分，而是 120 分。面试官一定是 心满意足， 五体投地。

按照尼恩的梳理，进行 深度回答，可以充分展示一下大家雄厚的 “技术肌肉”，**让面试官爱到 “不能自已、口水直流”**，然后实现”offer 直提”。

在面试之前，建议大家系统化的刷一波 5000 页《[尼恩 Java 面试宝典 PDF](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247497474&idx=1&sn=54a7b194a72162e9f13695443eabe186&scene=21#wechat_redirect)》，里边有大量的大厂真题、面试难题、架构难题。

很多小伙伴刷完后， 吊打面试官， 大厂横着走。

在刷题过程中，如果有啥问题，大家可以来 找 40 岁老架构师尼恩交流。

另外，如果没有面试机会， 可以找尼恩来改简历、做帮扶。

前段时间，**[刚指导一个小伙 暴涨 200%（涨 2 倍），29 岁 / 7 年 / 双非一本 ， 从 13K 一次涨到 37K ，逆天改命](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486433&idx=1&sn=0f630e8fc7620b3bf0e054827b48d91e&scene=21#wechat_redirect)。**

另外，**[刚指导一个 40 岁大龄，上岸：转架构，收 3 个外企 offer， 机会多了，不焦虑了，逆天改命](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486604&idx=1&sn=69e1b1b95f50947c2a1bf8601ac77b89&scene=21#wechat_redirect)。**

  

狠狠卷，实现 “offer 自由” 很容易的， 前段时间一个武汉的跟着尼恩卷了 2 年的小伙伴， 在极度严寒 / 痛苦被裁的环境下， offer 拿到手软， 实现真正的 “offer 自由” 。

## 空窗 1 年 / 空窗 2 年，如何通过一份绝世好简历，  起死回生  ？ 

[**空窗 8 月：中厂大龄 34 岁，被裁 8 月收一大厂 offer， 年薪 65W，转架构后逆天改命!**](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486405&idx=1&sn=02336456cf4a09268c142c0b13327682&chksm=97b57a4da0c2f35be50007aee12b8f9e9aa7a74864f720846ae6dc58a5fa4b092e22d1805e1e&scene=21#wechat_redirect)  

[**空窗 2 年：42 岁被裁 2 年，天快塌了，急救 1 个月，拿到开发经理 offer，起死回生**](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486028&idx=1&sn=cf10ecda7a986b26e0359eba7a9cfeff&chksm=97b57bc4a0c2f2d2be01f2656ac65e9bd8854b91c64b01eb55f56e645f3c733525418808b06c&scene=21#wechat_redirect)

[**空窗半年：35 岁被裁 6 个月， 职业绝望，转架构急救上岸，DDD 和 3 高项目太重要了**](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486098&idx=1&sn=bbc5732b071477573bfab8a259d208d3&chksm=97b57b1aa0c2f20c27dd74b490b6062c9eec262a3ac25548534ff70290b172dcc51c6eafe532&scene=21#wechat_redirect)

[**空窗 1.5 年：失业 15 个月，学习 40 天拿 offer， 绝境翻盘，如何实现？**](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486345&idx=1&sn=8978c2c98378e85efe9a089fa08fc9e8&chksm=97b57a01a0c2f317c29c7bedab1fa8a9f6101ab35cf005df447d662daa9c17d56f4e4c3a354a&scene=21#wechat_redirect)

##  100W 年薪  大逆袭,  如何实现  ？ 

# [**100W 案例，100W 年薪的底层逻辑是什么？** 如何实现年薪百万？ 如何远离  中年危机？](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486315&idx=1&sn=52cc0a640c1499115f5a45b82097d857&chksm=97b57ae3a0c2f3f502293e84cde4bc892c45d06dee3579471a522ab497fb53e32640fb00c339&scene=21#wechat_redirect)

[**100W 案例 2****：**40 岁小伙被裁 6 个月，猛卷 3 月拿 100W 年薪 ，秘诀：首席架构 / 总架构](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485942&idx=1&sn=faa999fc9508435db7a88af2716492d9&chksm=97b5787ea0c2f168ac80d344670df16be12614d37161a893ebed88d4c915384fcbc024996403&scene=21#wechat_redirect)

[**环境太糟，如何升 P8 级，年入 100W？**](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486369&idx=1&sn=0b32e8907dbcf26eaf33ea3f37ddd099&chksm=97b57a29a0c2f33f468d29dafe739204543873efb459f324871c3c68a3ecc178b1a9ab08a2fa&scene=21#wechat_redirect)

## 如何  评价一份绝世好简历， 实现逆天改命，包含 AI、大数据、golang、Java  等 

##   

*   ## [逆天大涨：暴涨 200%，29 岁 / 7 年 / 双非一本 ， 从 13K 涨到 37K ，如何做到的？](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486433&idx=1&sn=0f630e8fc7620b3bf0e054827b48d91e&chksm=97b57a69a0c2f37fad55b164ba7afb963e0db0366901da97a1fadc599cb5cf1ef95dfac736f2&scene=21#wechat_redirect)
    
*   ## [逆天改命：27 岁被裁 2 月，转 P6 降维攻击，2 个月提 JD/PDD 两大 offer，时来运转，人生翻盘!!  大逆袭!!](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486421&idx=1&sn=02a88c23c8fe689a662214d5297f88a0&chksm=97b57a5da0c2f34b1f87ed0bb948ddfc9327533bdfc44b88ad25201949bbdb8ae7439904e675&scene=21#wechat_redirect)
    
*   ## [急救上岸：29 岁（golang）被裁 3 月，转架构降维打击，收 3 个大厂 offer， 年薪 60W，逆天改命](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486392&idx=1&sn=16c34d7f960805f659adfb27676ee2de&chksm=97b57a30a0c2f326092cd7f79e125fd8bd5a006ed7459f1a2a81a8d557388297a02acd2cd5ce&scene=21#wechat_redirect) 
    

*   [**绝地逢生：**9 年经验](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486132&idx=1&sn=bb949c992b3a35935bf5fca57f19e2d8&chksm=97b57b3ca0c2f22aeb0e648191801aff933164dcf2b06eb4b6274df6499bffb0bba8e8169205&scene=21#wechat_redirect)[自考](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486132&idx=1&sn=bb949c992b3a35935bf5fca57f19e2d8&chksm=97b57b3ca0c2f22aeb0e648191801aff933164dcf2b06eb4b6274df6499bffb0bba8e8169205&scene=21#wechat_redirect)小伙伴，跟着尼恩狠卷 3 月硬核技术，面试机会爆表，2 周后收 3 个 offer ，满血复活
    

**职业救助站**

实现职业转型，极速上岸

![](https://mmbiz.qpic.cn/mmbiz_jpg/9qY2Gt3PBs3icE6WcM4nBOdRRcczOW4wkNDgCMZhBfqeet9suwGHfgQduP33YYqqZLOppecMI72Pv5ibNkPeIk6g/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1#imgIndex=2)

关注**职业救助站**公众号，获取每天职业干货  
助您实现**职业转型、职业升级、极速上岸**  
---------------------------------

**技术自由圈**

实现架构转型，再无中年危机

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/xlgvgPaib7WMtyFcJx45oIeWL1d6AENcdoAvuTPuq46E9n8pPe4SHXoSicuEVvhibHRSIzmEBPlC7POcDaqD6ID7w/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1#imgIndex=3)

关注**技术自由圈**公众号，获取每天技术千货  
一起成为牛逼的**未来超级架构师**

**几十篇架构笔记、5000 页面试宝典、20 个技术圣经  
请加尼恩个人微信 免费拿走**

**暗号，请在 公众号后台 发送消息：领电子书**

如有收获，请点击底部的 " 在看 "和" 赞 "，谢谢