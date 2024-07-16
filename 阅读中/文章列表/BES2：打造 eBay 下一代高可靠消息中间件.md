---
source: https://mp.weixin.qq.com/s?__biz=MzA3MDMyNDUzOQ==&mid=2650516484&idx=1&sn=6dfb7bcfc160a06253defda1f6728bd6&chksm=87319450b0461d469a04d0c8458a50657e1128b47cd203e83645b51686d581e9c0487a44ee61&mpshare=1&scene=1&srcid=10204WIBuS9kYkBPxtwNi2FX&sharer_shareinfo=2c9e3bce88259be656a4e09847b62a04&sharer_shareinfo_first=2c9e3bce88259be656a4e09847b62a04#rd
create: 2023-11-13 17:11
---

# BES2：打造 eBay 下一代高可靠消息中间件

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/nwwClDeS1mNr8cjDibicO5hHVnIMZic2Ij1tetSF56QibrZlicPATsFnKNiaOSx0nOuNR3HTXqQgswn6HcWO77aibltpg/640?wx_fmt=jpeg)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/nwwClDeS1mNibx8WtVdiacPDib98cJYXBSFlkJUTY9PdHOxGIibtRDWLXKV2zUUtP7MKwkHQQiac5UUPYmBK4QDfEzQ/640?wx_fmt=png)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/nwwClDeS1mNibx8WtVdiacPDib98cJYXBSFSkcaTXibHS1WPmEbbmsnPoRiaKHJx3FwLOAu6QaFuk0dNIeOOzibWCYRA/640?wx_fmt=png)

**前言**

Forewords

消息系统作为大型分布式系统中的重要组成部分，在企业中广泛应用于异步处理、应用解耦、流量削峰以及事件驱动等多种关键场景。过去的 10 多年中，eBay 内部消息系统 BES（Business Event Streams）已被广泛应用于支付、风控、搜索等核心业务领域。  

随着业务规模的不断发展和数据的日益增长，eBay 现有的消息系统逐渐在消费延时和大流量可靠性等方面无法满足需求。为此，**我们构建了下一代高可靠消息中间件 BES2，其具有如下特点：**

1）**可靠消息保证**：消息被成功写入后，BES2 将确保消息可靠地传递给消费者。

2）**极低的端到端投递延迟**：提供毫秒级的端到端消息传递延迟保证。

3）**支持大流量的重试消息和延时消息**：支持大流量下的消息重试以及延时消费，且用户可设置任意的重试 / 延迟时间，以满足不同的业务需要。BES2 可保证秒级投递延迟。

4）**自动跨数据中心容灾**：数据备份覆盖三个数据中心，支持消息生产端和消费端的自动无感故障转移。

5）**完善的生态集成**：与 eBay 内部各种生态系统紧密集成，显著扩展了消息系统的应用范围。

6）**向后兼容与平滑迁移**：支持与 BES1 完全兼容的 API，使得应用端能够保持代码逻辑不变，只需通过自助配置即可实现平滑迁移，同时确保在迁移过程中不会丢失消息。

本文将从这些特点出发深入探讨 BES2 如何打造低延时、高可用、高可靠的消息中间件平台。

**架构设计**

Architechture

**01**

**BES2 构建于 Kafka 之上，提供丰富的消息语义，包括消息发布订阅、延迟消息 [1]、延迟订阅 [2]、消息重试、死信队列等。与此同时，BES2 API 完全兼容 BES1，以便 BES1 用户能够无缝迁移到 BES2。**BES2 的核心架构如下图所示：

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/nwwClDeS1mNibx8WtVdiacPDib98cJYXBSFlLlEXZvap1xLEVTUBClydHicy4WibYnp7jupmxNtPwsKWC0nxkyC3HKA/640?wx_fmt=png)

图 1. BES2 核心架构图

其中的关键组件有：

*   **Messaging Service**：无状态高可用的消息服务计算层，提供消息发布订阅、 延迟消息、延迟订阅、消息重试、死信队列等功能的实现。
    
*   **Kafka**：BES2 使用 Kafka 作为消息存储层，利用 Kafka 提供的多副本存储，确保消息数据的高可用和持久性。
    
*   **BES2 Bridge**：BES2 内置的消息同步组件，提供将 BES1 和 NuDocument BES 消息从 Oracle/NuDocument CDC[3] 实时同步到 BES2 持久化存储的能力。
    
*   **Producer**：BES 支持三种类型的生产者。BES1 Producer 生产消息并将其持久化到基于 Oracle 的 BES1 存储组件，同时通过 BES2 Bridge 同步到 BES2 存储。BES2 Producer 生产消息并直接持久化到基于 Kafka 的 BES2 存储组件。NuDocument Producer 支持事务性消息发送，以确保 BES 消息的发送与用户业务文档写入操作的原子性，这些消息同样会通过 BES2 Bridge 同步到 BES2 存储。
    
*   **Consumer**：BES 支持三种类型的消费者。BES1 Consumer 仅能消费 BES1 消息。BES2 Consumer 具备同时消费 BES1 和 BES2 消息的能力。此外，BES2 还可与 Flink 无缝集成，支持以流式处理的方式消费 BES2 消息。

本章将重点介绍 **BES2 的内部实现机制**，包括消息可靠性、大容量消息重试以及跨数据中心容灾的原理，以帮助读者深入了解 BES2 的内部工作原理。

**1-1**

**可靠消息保证**

在大规模分布式系统中，消息系统作为基础网络通讯组件，常常面临网络不可靠性和系统崩溃等问题，这些因素可能导致消息丢失等不可靠情况。下面，我们将从生产端发送、服务端消息持久化存储和消费端可靠投递三个方面来详细探讨 BES2 如何确保消息的可靠性。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/nwwClDeS1mNibx8WtVdiacPDib98cJYXBSFjuMdFJiaBOa6kjdNpWs3ysesxjrG3MRj7ukV7LLb6w1SF4HY7m8n4wg/640?wx_fmt=png)

图 2. 可能影响消息可靠性的路径

**生产端可靠发送**

在消息生产端，BES2 Producer 使用同步的 gRPC 与 Messaging Service 进行交互。Messaging Service 内部的 Kafka Producer 将消息写入 Kafka 主题。在 BES2 中，我们采取了以下措施来确保消息的可靠性：

1）所有存储消息的主题副本数为 3，并且配置为 `min.insync.isr=2`，以确保至少有两个副本保持同步，才能将消息成功写入。

2）Messaging Service 内部的 Kafka Producer 配置为 `ack=all`，确保消息必须在所有同步的副本上成功写入后才返回成功。

基于上述两点，BES2 能够保证消息发送要么成功，要么失败。此外，生产端内置了重试机制，保障消息发送的成功率。

**服务端持久存储**

消息的持久化存储确保了在消息堆积时不会因宕机、磁盘故障等情况而丢失。当 BES2 接收到消息时，通过 Kafka 将消息持久化存储。借助 Kafka 的多副本机制，我们可以确保消息在 Broker 宕机时不会丢失。此外，通过禁用 Kafka 的 `unclean.leader.election.enable` 功能，我们可以阻止非同步的副本被选为 Leader，从而避免由此带来的数据丢失问题。

**消费端可靠投递**

消费端可靠投递确保消息最终一定能够被消费者获取，并且能够成功被消费。这有助于防止消费者或 Messaging Service 的宕机、网络抖动等异常情况导致消息消费失败，最终导致消息丢失。为了确保消息的可靠投递，BES2 引入了消费端的消息确认机制以及服务端超时重传机制，以确保消息能够被消费者接收并成功处理。

BES2 消息生产消费流程如下图所示，消息消费过程可分为消息拉取、消息处理和消息确认三个阶段，接下来先逐个分析这些阶段的数据流程，然后探讨如何保证消息投递的可靠性。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nwwClDeS1mNr8cjDibicO5hHVnIMZic2Ij1ibtBYTkIyQibFxeB4cNc60Bnt0n99RicEAD2Fia4RBjh1icCSXTLVBuOHpQ/640?wx_fmt=png)

图 3. BES2 消息生产消费流程图

**消息拉取阶段**负责拉取消息投递给消费者，主要流程如下：

**①**

BES2 消费者后台拉取线程发起消费请求。

**②**

Messaging Service 收到请求后从 Kafka 消息存储中拉取消息，这里除了拉取原始消息队列，还会拉取重试消息队列。

**③**

拉取消息成功后，首先将消息的元数据持久化至 “未确认消息存储队列”，如果 Messaging Service 在消息确认超时时间内没有收到消费者的确认，则认为该批次消息可能投递失败，会将其重新投递给消费者。

**④**

Messaging Service 更新内部消费者消费位点。

**⑤**

最后将消息返回给 BES2 消费者。

另外值得注意的是，消息拉取阶段 BES2 消费者与内部消费者之间是多对一的映射关系，这使得单个分区消息可被多个 BES2 消费者实例无重复地消费，同时这消除了 Kafka 单个分区消息只能由消费组内一个消费者实例消费的限制。

**消息处理阶段**是指 BES2 消费者接收到消息后，对接收到的消息进行处理并更新每条消息的消费状态（成功、失败、延时重试）。

**消息确认阶段**负责根据消费状态进行下一步处理，主要流程如下：

**⑥**

BES2 消费者对本批次中所有消息都处理完成并且更新消费状态后，会自动向 Messaging Service 发送消息确认请求。另外如果到达最大消息处理时间（batch.runaway.timeout）也会自动发送消息确认请求，在此种情况下，未被处理消息将被视为待重试消息。

**⑦**

Messaging Service 在接收到消费者消息确认后，如果有重试消息，则将重试消息发送至相应重试延时队列；下一节会具体探讨如何通过多级重试延时队列实现大容量消息重试。

**⑧**

对于消息消费成功的场景，从未确认消息存储队列中删除未确认消息记录。

总结上述数据流程，BES2 消费端在处理时，消息状态的转换如下图 4 所示。接下来我们探讨如何保证 BES2 的消息投递可靠性。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/nwwClDeS1mNibx8WtVdiacPDib98cJYXBSF07JBsxTBwQvcFR1QzxSibEI7o6WxTSBa8bWCJSCwGLSE9WI4bEhdGRw/640?wx_fmt=png)

图 4. BES 消息状态流转图

可能导致消息投递失败的第一种情况是 Messaging Service 宕机，这可能发生在消息拉取成功、持久化未确认消息、提交消息消费位点以及消息返回给消费者的任一时刻。

1）如果 Messaging Service 在拉取消息成功或者持久化未确认消息后宕机，此时由于消费位点未更新，Messaging Service 启动后将会从已提交的消费位点之后开始拉取消息。此种情况，未确认的消息将被重复投递。

2）如果 Messaging Service 在成功更新消费位点之后宕机，此时消息未返回给消费者。这种情况下，未确认消息已持久化存储。确认超时时间到期后，消息将重新投递给消费者。

另一种情况是消费者在接收消息后崩溃。由于消息未被确认，该情况同上面第二种场景相同，消息将在确认超时时间到达后重新投递给消费者。

**无论哪种情况，消息一定会被投递给消费者，直至被处理完成并被显式地设置消费状态，这就保证了消费端的可靠投递。**

**1-2**

**大容量消息重试和延时消费**

在分布式系统中，消息消费可能因瞬时的网络问题或资源不足而导致处理失败。消息重试机制使得消费者可以在稍后某个时间内再次重新处理这些消息而不必丢弃它们，从而提高系统的稳定性和可用性。

在 BES1 中，消息重试的数量限制在消费者接收到的消息数量的 5% 以内，而且无法保证消息的重试投递延迟。对于需要进行大容量消息重试的用户，他们必须使用复杂的机制来自定义实现消息重试。而 BES2 采用了创新性的方法，引入了多级重试延时队列来解决大容量消息重试以及延时消费问题，同时保证了极低的投递延迟。下图（图 5）展示了大容量消息重试以及延时消费的关键组件。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/nwwClDeS1mNibx8WtVdiacPDib98cJYXBSFfVmPFjRC064NIMJJQqye2ibvCq8L6xvGrQCIAhvLl4mib5D5ThcIto9w/640?wx_fmt=png)

图 5. 大容量消息重试关键组件

**多级重试延时队列**：用于存储待重试消息和延时消息，由多个消费者共享。Messaging Service 会根据消费者指定的延迟时间 T，将重试消息或者延时消息投递至相应延时队列中。例如，重试延时不大于 T0 的重试消息将被投递至 Level-0 队列，重试延时时间不大于 T1 的消息被投递至 Level-1 队列，依此类推。这确保了消息按照不同的延迟时间被分级处理。

同时，多级重试延时队列组成一个逻辑 Group，允许消费者在多个 Group 之间无缝切换。例如，消费者可以将延时重试消息发送至 Group-0，也可将延时重试消息发送至任何其他 Group。同时逻辑 Group 个数支持无限水平扩展，由此可保证 BES2 延时重试消息水平扩容能力及可靠性，也是 BES2 支持大容量重试消息的基石之一。

**重试延时消息同步服务**：用于在不同级别的队列之间转储消息，并将已经满足投递时间的消息转入待投递重试消息队列。对于多级重试延时队列中的每一级队列，重试延时消息同步服务将运行多个任务实例，任务实例的数量取决于该级队列的分区数。Messaging 分布式任务调度平台 [4] 负责管理任务实例的分发、运行状态监控以及异常处理，以确保延时消息被及时投递。

**待投递重试消息队列**：用于存储延时到期将被投递给消费者的重试消息，该队列由单个消费者独占。该队列在 BES2 消费者启动时被隐式地订阅，其中存储的消息会在消息拉取阶段被 Messaging Service 返回给 BES2 消费者。

多级重试延时队列的引入和设计，使 BES2 能够高效、可靠地处理大容量的重试消息。且在与延时消息投递算法 [5] 的配合使用下，用户能够根据不同的业务场景，对每一条重试 / 延时消息，设置不同的、任意的延时时间，且能够保证极低的消息投递延迟。

**1-3**

**跨数据中心灾备**

在 eBay 内部，消息系统广泛应用于支付、风控、搜索等关键业务场景。因此，保障业务连续性对于 BES2 至关重要，特别是在数据中心出现故障或不可用情况时。eBay 的数据中心主要分布在美国的三个区域：LVS（Las Vegas）、SLC（Salt Lake City）和 RNO（Reno）。如下图 6 所示，BES2 采用了存储与计算分离的架构，Messaging Service 节点部署在这三个数据中心。接下来，我们将分别从计算层和存储层来探讨 BES2 的跨数据中心高可用。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/nwwClDeS1mNibx8WtVdiacPDib98cJYXBSFOWMVF08cgh6vTwBH5JCiawNo7Hrh6yljib2VI4rPhoP1aPgvGic85ibljg/640?wx_fmt=png)

图 6. 跨数据中心灾备

**计算层跨数据中心高可用**

BES2 的计算层由 Load Balancer 和 Message Service 节点组成，其中 Messaging Service 节点分布在三个数据中心，而 Load Balancer 则跨数据中心可用。Messaging Service 节点是无状态的，它们通过 Zookeeper 感知其他节点的存活状态。BES2 客户端通过服务发现协议连接到 Messaging Service 集群，其工作流程如下：

**①**

BES2 客户端实例通过 Load Balancer 连接到 Messaging Service 集群。

**②**

Messaging Service 集群选择集负载最小的节点返回给 BES2 客户端。

**③**

BES2 客户端直接连接上一步返回的节点，进行后续消息发送和消费。消息的发送和消费无需再经过 Load Balancer。

**④**

如果 BES2 客户端所连接 Messaging Service 节点不可用，重复上述 1-3 步。

当单个数据中心不可用时，该数据中心节点会从 Zookeeper 中移除。Messaging Service 的服务发现协议确保连接到不可用数据中心的 BES2 客户端将重新连接到其他数据中心的 Messaging Service 节点，以确保跨数据中心高可用性。

**存储层跨数据中心高可用**

BES2 存储层采用了 eBay 自主开发的多地互备 Kafka Stream（详情参考技术荟：[揭秘 eBay Kafka 跨数据中心高可用方案](http://mp.weixin.qq.com/s?__biz=MzA3MDMyNDUzOQ==&mid=2650516368&idx=1&sn=8c29370cf3aee4b0ece5fa393e309ec7&chksm=873193c4b0461ad2576c2ce62c3794dc41749333348366e5e4f8aa86d6d2fb9e47e56ccc7615&scene=21#wechat_redirect)）。多地互备 Kafka Stream 包含 Local 和 Aggregation 两层 Kafka 集群，Local 层是每个数据中心的本地 Kafka 集群，其中的数据通过 MirrorMaker 组件实时复制到 Aggregation 层的每个 Kafka 集群；每个数据中心的 Aggregation 层都包含了全量数据。

对于 BES2 Producer，每个数据中心的写入消息请求将被发往本数据中心 Local Kafka 集群。当本数据中心 Local Kafka 集群不可用时，写入消息请求将自动灾备到其他数据中心。

而 BES2 Consumer 则只从单个数据中心的 Aggregation 层 Kafka 集群消费数据。当单个数据中心不可用时，对于未消费消息，可以通过自研增强的 Kafka SDK 将消费者自动切换至可用数据中心，并通过数据中心之间消费位点映射组件管理切换之后消费起始位点，以确保切换之后消息不会丢失。

需要特别关注的是 “未确认消息存储队列” 中的消息，这些消息的存储内容是消息元数据（分区 + 偏移量 + 消息 ID）。当消息被传递到另一个数据中心后，相同的偏移量在不同数据中心可能对应不同的消息内容。数据中心切换后如果根据元数据消息偏移量去查找消息内容，则很可能将错误的消息投递给消费者，而原始消息可能被丢失，从而引起业务异常。

为了解决这个问题，BES2 引入了消息恢复组件来处理未确认的消息。

**①**

Message Service 消息处理组件从 “未确认消息存储队列” 读取到待处理消息记录时，如果记录数据中心和当前读取数据中心不一致，则将消息发送至“待恢复消息队列”，交由消息恢复组件处理。

**②**

消息恢复组件读取到待恢复消息记录后，请求 Kafka 消费位点管理服务 [6] 获取该偏移量在其他数据中心的可能位点。

**③**

消息恢复组件连接到其他数据中心，从上一步中返回的位点开始遍历记录，直到找到与 messageId 匹配的消息。

**④**

消息恢复组件将消息投递给消费者并从 “待恢复消息队列” 中删除记录。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/nwwClDeS1mNibx8WtVdiacPDib98cJYXBSFgVgdUM3E0qFc0U3lzIPulibibaWlmOH2QsUxKiciblXibeRibaUSJc77WC7Q/640?wx_fmt=png)

图 8. 消息恢复组件

消息恢复组件确保了消费者在切换数据中心时，未确认的消息在切换后的数据中心也能够被正确地投递给消费者，从而保证数据中心切换时消息的完整性。

跨数据中心灾备的设计使得 BES2 能够在数据中心级别的故障或不可用情况下保持高可用性，确保业务的连续性。

**完善的生态集成**

Ecosystem

**02**

BES2 还实现了与 eBay 内部生态系统的广泛集成，包括支持 NuDocument 类型的事务性消息发送以及使用 Flink SQL/Connector 进行流式消息处理。

**NuDocument 事务消息发送**

NuDocument 是 eBay 自主研发的文档数据库，在公司内部被广泛使用。BES2 支持通过 NuDocument 实现事务性消息发送，即在写入业务文档时同时发送 BES 消息，并确保业务文档和 BES 消息同时成功或失败。

在下图中，上半部分是通过 BES 库进行正常的生产和消费，在生产者应用中，用户可以选择使用 BES 库来发送消息。然而，还存在另一种常见情况，例如用户通过 NuDocument API 写入业务数据，同时需要生成相关的事务消息。此时如果通过 BES 库发送消息是无法保证事务性的。

在这种情况下，用户可以在原始 NuDocument API 请求中同时包含一条 BES 消息，NuDocument 可以确保在同一写入请求中，只要业务数据或 BES 数据中的任何一个失败，两者都不会成功写入，否则业务数据和 BES 数据都会被持久化存储。写入成功后 BES 数据会被 Change Data Capture（CDC）流程同步到相应的 Kafka 主题中。由于 BES 数据具有 "Append only" 属性，这些 CDC 数据都是 INSERT 类型。CDC 数据将由 BES NuDocument Bridge 同步到 BES2 存储中，从而可以由 BES2 消费者正常消费。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/nwwClDeS1mNibx8WtVdiacPDib98cJYXBSFJfHNRsQ2524CuCC8LNfc1ibLnwRUx4DHz12hMBEBOHqGkMYib1xCicJAw/640?wx_fmt=png)

图 9. NuDocument 事务消息发送流程图

接下来，我们讨论 BES NuDocument Bridge 是如何同步数据的。首先，控制面在创建每个 NuDocument 类型的 BES 事件元数据时，会同时创建相应的数据同步任务，并将其分发给 Messaging 分布式任务调度平台 [4]。该平台创建适量的 Bridge Worker 以进行数据同步，以确保可扩展性并适应不同的数据量。每个 Bridge Worker 的逻辑大致分为以下几个步骤：首先，从 CDC 主题中轮询数据，解析为 ChangeEvent 对象，并构建成 BES2 数据结构；然后，将构建好的 BES2 数据批量发送到 BES2 存储；最后，提交消费位点信息。这确保了在同步过程中不会丢失消息。

在消息系统中，这种数据同步的需求非常常见，因此我们将 BES Bridge 设计成一个轻量级且可配置的通用数据同步平台，数据同步任务的定义、分发和扩展性都由框架管理，具体的数据同步任务只需实现不同的 Bridge Worker 即可。例如，在本节中，我们使用的是 NuDocumentCdcWorker，用于从 NuDocument CDC 主题中同步到 BES2；而在下文 BES2 迁移中也会涉及到数据同步，使用的则是 OracleBridgeWorker，用于从 BES1 Oracle 数据源同步到 BES2。

**Flink Connector 流式消息消费**

Apache Flink 是 eBay 内部广泛使用的流式数据处理平台，支持从 Kafka、MySQL 等数据源读取数据并进行处理，然后将结果 Sink 到其他数据存储中。**BES2 与 Flink 进行了集成，支持使用 BES Flink Connector 读取和处理 BES 消息，从而显著扩展了 BES 的使用场景。**  

为了达到这个目标，我们基于 FLIP-27 开发了 BES Flink Connector，其主要组件如下图。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/nwwClDeS1mNibx8WtVdiacPDib98cJYXBSFUOuKwKQUbdXJVNWIKjgoOeuDrQKSegLdIlhJE7rsAghAbmUiaFxqvaQ/640?wx_fmt=png)

图 10. BES Flink Connector 关键组件

*   **SplitEnumerator**：在 Job Manager 上运行，负责发现和分配 Split（即 BES 事件）。用户可以配置要消费的事件列表或模式，SplitEnumerator 基于此列表或模式查询管理 API 以获取事件元数据，然后将事件分配给 SourceReader 处理。我们会根据用户配置的并行度灵活启动多个 SourceReader，实现了比 Kafka Connector 更好的扩展性，这得益于 BES 突破了 Kafka 消费者数量受主题分区数的限制。
    
*   **SplitFetcherManager**：在 Job Manager 上运行，负责 SplitFetcher 线程的管理。根据当前消费者订阅的事件数量，SplitFetcherManager 会为每个订阅分配一个单独的线程。
    
*   **SplitReader**：在 Job Manager 上运行，是实际进行消息拉取的组件。每当 SplitEnumerator 发现新的 Split 后，SplitReader 将创建 BES 消费者实例以拉取对应的 BES 队列中的数据，并将其放入 elementQueue 队列。同时，SplitReader 需要处理消息的确认提交，具体是在快照时将当前已消费但尚未提交的 batchId 作为状态进行持久化，然后在 Checkpoint 完成后提交这些 batchId，以确保数据不会丢失。
    
*   **RecordEmitter**：在 Job Manager 上运行，负责将接收到的消息序列化为目标类型，并发送给下游算子。消息序列化方式是可配置的，用户可以使用 Flink 原生的 DeserializationSchema 或 TypeInformation 配置目标类型，还可以配置为标准的 BES 事件 POJO，这使得下游处理逻辑可以与普通 BES 消费者完全一致。

在接收到序列化后的消息流后，下游 Flink 算子可以对其进行各种操作，例如聚合、连接等。目前，BES Flink Connector 已经开始支持 eBay 多个业务的实时计算需求，例如 Ads 团队通过 Flink 应用程序消费 BES 中的用户行为数据以发现可能的用户欺诈行为。

**向后兼容和平滑迁移**

Migration

**03**

由于 BES2 具备更佳的延迟和可靠性，并支持更广泛的上下游生态系统，因此许多用户希望能够无缝地将其从 BES1 迁移到 BES2。BES2 保持了与 BES1 完全兼容的 API，因此对于用户而言，BES2 和 BES1 的代码逻辑可以完全相同。这为平稳的迁移提供了坚实的基础，应用端无需更改代码逻辑，只需通过控制面修改配置即可轻松完成迁移。

**生产者消费者的独立迁移**

在消息系统中，一项重要的使用场景是实现应用解耦。因此，BES 迁移的一个关键目标是确保生产者和消费者可以独立进行迁移，允许用户自由选择是先迁移生产者还是先迁移消费者，否则就背离了应用解耦的初衷。  

如下图所示，橙色路径代表原有的 BES1 流程，生产者通过 BES 库生产消息，BES 将消息持久化到基于 Oracle 的 BES1 存储中，消费者连接到 BES1 存储拉取消息。而 BES2 使用了全新的基于 Kafka 的存储，因此对于生产者来说，需要切换到新的存储进行消息发送；对于消费者来说，需要切换到新的存储进行消息拉取。我们提供了两种不同的策略来完成迁移，如图中的蓝色路径所示。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/nwwClDeS1mNibx8WtVdiacPDib98cJYXBSFnhAiaSEtwZgOClYAZJZe5tUTum4v1ze00eqL47UjqBPkkD1J2DuNLqg/640?wx_fmt=png)

图 11. BES 迁移的两种策略

对于先迁移生产者的场景，我们提供了 "双写"（dual write）策略来实现迁移。生产者通过控制面触发迁移后，底层工作流会自动创建与 BES2 相关的存储资源，利用 BES 库将数据同时写入 BES1 和 BES2 的底层存储中。然后消费者可以选择开始迁移并最终从 BES2 读取数据。当所有下游消费者都完成迁移后，生产者可以停止向 BES1 存储写入数据，只写入 BES2，从而完成迁移过程。

然而在许多情况下，消息系统需要先迁移消费者。例如，消息生产者可能位于第三方应用程序中或者存在于旧的遗留系统中。在这种情况下，生产者将继续将数据写入 BES1 存储，但不会写入 BES2 存储。为了解决这个问题，我们使用 BES Bridge 组件，将数据从 BES1 存储同步到 BES2 存储。这里的 BES Bridge Worker 实际上是通过一个内置的 BES1 Consumer，读取 BES1 的数据并将其写入 BES2 存储，同时依赖 BES 的特性确保数据不会丢失。一旦消费者通过控制面触发迁移，底层工作流将自动创建与 BES2 相关的存储资源，并自动创建 BES Bridge Woker 实时同步相关消息。需要注意的是，在这种情况下，端到端延迟可能会比先迁移生产者要高一些，这是因为增加了 BES Bridge 数据同步的额外步骤，所以我们需要确保 BES Bridge 的延迟尽可能小，这可以通过调整 Bridge 消费者的参数来实现。

通过同时支持双写和 BES Bridge，我们提供了不同的策略，以适应不同迁移需求的情景，从而确保在升级到 BES2 时，生产者和消费者的迁移是相互隔离的、互不依赖的。

**无缝平滑迁移**

消息系统迁移的另一个重要目标是实现无缝平滑迁移，无需重启应用，同时确保在迁移过程中数据不会失，并尽量减少数据重复。一种策略是允许应用端同时配置两个消费者，分别消费 BES1 和 BES2 的消息，然后在应用端进行数据比对，并在合适的时间点进行切换。然而，这种方法需要应用端投入大量的开发工作，同时由于应用端对 BES 的了解程度不同，可能导致比对和切换过程中出现错误。为了降低复杂性，我们将这些任务下沉到 BES 框架中，使得应用端只需通过控制面来触发和确认迁移过程。  

BES 消费者迁移过程包含三个关键时间点：

**①**

**T0：触发迁移**，由用户通过控制面触发。在 T0 之前，BES 库只消费 BES1 的消息，表现为普通的 BES1 消费者。一旦在 T0 时刻触发了迁移，BES 会创建相关 BES2 消费者以及存储资源。如果此时生产者尚未开始迁移，则自动创建 BES Bridge Worker 将数据同步到 BES2 存储。然后，BES 库就可以开始同时消费 BES1 和 BES2 的消息，并将接收到的消息 ID 发送到消息校验模块（下文将讨论该模块）。同时，BES 库会过滤掉接收到的 BES2 数据，因此从用户角度看，业务代码仍然在消费 BES1 的数据。

**②**

**T1：切换到 BES2**，由 BES 框架自动触发。当消息校验模块验证通过后，表示消费者接收到的 BES2 消息与 BES1 消息完全匹配，BES 框架会选择一个时间点 T1 来自动触发切换。BES 库将过滤掉 T1 时刻之后的 BES1 数据，业务代码开始接收 T1 时刻之后的 BES2 数据。

**③**

**T2：完成迁移**，由用户通过控制面触发。在完成切换之后，用户会收到通知，确认无误后可以选择完成当前迁移。之后，BES 库只接收 BES2 的数据，至此迁移过程结束，切换为普通的 BES2 消费者。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/nwwClDeS1mNibx8WtVdiacPDib98cJYXBSFicn1xjH9e8eaJmibykWXAiaWECA42MNAnm5iaX8CUZ616haAABliacZUmAQ/640?wx_fmt=png)

图 12. BES 消费者的平滑迁移与消息校验

由于在 T1 时刻切换到 BES2 时，消息校验模块已经确保 BES2 消息与 BES1 完全匹配，因此不会发生数据丢失。同时，由于消息在指定的时间窗口内完全匹配，BES2 和 BES1 的消费进度是大致相同的，再加上 BES 库的自动过滤，可以尽量减少切换过程中消息重复的机会。

BES 生产者的迁移过程更加直观，用户通过控制面触发迁移后，BES 库即开始同时双写 BES1 和 BES2。同时，自动创建一个后台消费者以模拟消费者迁移，并对其进行消息校验。校验通过后，通知所有相关的消费者可以选择迁移。当所有下游消费者完成迁移后，生产者可以通过控制面确认切换到 BES2。在整个过程中，用户只需要负责触发迁移和确认切换。

**自动消息校验**

接下来，我们将探讨消息校验模块是如何工作的。该模块的目的是确保 BES 库接收到的 BES1 和 BES2 数据是完全匹配的，以避免切换后出现数据丢失，同时尽量减少数据的重复。  

如上一节图所示，在触发迁移（T0）后，每当 BES 库接收到一条消息，就会将消息 ID 发送到一个内部 Kafka 主题，并标明该消息是来自 BES1 还是 BES2。这个 Kafka 主题将被一个 Flink Job 消费，并通过两次窗口聚合来进行数据比对：

**①**

通过 Session Window 聚合来探测每个消息 ID 的匹配情况。每当接收到一个新的 ID，就会开启一个 Session Window，在窗口时间内（例如 20 分钟），如果接收到了对应的另一个数据源的相同 ID，就会认为匹配；否则将视为不匹配。这一步的输出结果是每个消息 ID 的匹配情况，数据量较大。

**②**

通过 Tumbling Window 聚合每小时的数据匹配情况，并生成报告。Tumbling Window 聚合可以有效减少最终报告中的数据量，同时在报告中保留了不匹配数据的样本，以便进行问题排查。

最终，Flink Job 将生成每个正在迁移的订阅的 Metric 和每小时的消息校验报告。如果连续 N 个小时的数据都匹配，那么校验通过，将自动触发切换到 BES2，即触发上图中的 T1。  

然而，如果校验持续未通过，应该怎么办呢？在这种情况下，系统将无缝回滚到 BES1。因为连续校验未通过可能是由于 BES Bridge 错误导致 BES2 数据丢失，或者两个数据源消费时出现了非常大的延迟差异，或者生产者在双写过程中出现了故障等原因。无论出现哪种情况，都不能进行切换，否则可能会导致业务问题。系统在这种情况下会自动切换回 BES1，并将消费位点重置为触发迁移的时刻（T0）。当然，在迁移过程中，用户还可以随时手动触发回滚，并自定义要回滚的消费位点。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/nwwClDeS1mNibx8WtVdiacPDib98cJYXBSFlkJUTY9PdHOxGIibtRDWLXKV2zUUtP7MKwkHQQiac5UUPYmBK4QDfEzQ/640?wx_fmt=png)

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/nwwClDeS1mNibx8WtVdiacPDib98cJYXBSFSkcaTXibHS1WPmEbbmsnPoRiaKHJx3FwLOAu6QaFuk0dNIeOOzibWCYRA/640?wx_fmt=png)

**结 语**

Endings

**BES2 是 eBay 下一代消息中间件，旨在解决原有消息系统的延时和可靠性问题。****它引入了多项关键特性，包括可靠消息保证、极低的消息投递延迟、大容量重试消息支持、自动跨数据中心容灾、更完善的 eBay 生态集成以及向后兼容与平滑迁移。**这些特性使 BES2 成为一个强大的消息中间件平台，适用于广泛的企业应用场景。

**术语**

[1] 延迟消息：在消息发布后，并不立即投递给消费者，而是在一定的时间延迟后再投递。延时时间由发送端指定，对该消息的所有消费者生效。

[2] 延迟订阅：服务端在收到消息后，并不立即投递给消费者，而是在消费端指定的延时后再投递。延时时间由消费端指定，只对指定的订阅关系生效。

[3] NuDocument CDC (Change Data Capture)：NuDocument 为 eBay 自研高可用文档数据库。NuDocument Change Data Capture 为数据库更新日志记录组件，类似于数据库系统 binlog。 

[4] Messaging 任务调度平台：Messaging team 开发的 master-slave 架构任务管理系统，在 Messaging 内部广泛应用于 BES1 到 BES2 数据消息迁移、BES2 NuDocument CDC 消息同步以及 BES2 重试延时消息同步。

[5] 延时消息投递算法：基于多级重试延时队列的消息投递算法，使重试消息和延时消息能够支持设置任意的延时时间，以适配各种业务需求。此算法正在申请专利，本文暂不细述。

[6] Kafka 消息消费位点管理服务：提供 local-agg kafka stream 中不同数据中心之间消息 offset 映射关系。该服务基于稀疏索引建立不同数据中心消息 offset 的映射。详情参考：[揭秘 eBay Kafka 跨数据中心高可用方案](https://mp.weixin.qq.com/s?__biz=MzA3MDMyNDUzOQ==&mid=2650516368&idx=1&sn=8c29370cf3aee4b0ece5fa393e309ec7&scene=21#wechat_redirect)。 

[7] FLIP-27：经过重构的 Flink Source Connector 开发框架，详情参考 https://cwiki.apache.org/confluence/display/FLINK/FLIP-27%3A+Refactor+Source+Interface。