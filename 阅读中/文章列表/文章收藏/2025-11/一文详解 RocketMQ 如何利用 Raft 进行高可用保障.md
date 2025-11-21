---
source: https://mp.weixin.qq.com/s/TT_NpT3xV9Yhmva73E85Hg
create: 2025-11-18 20:56
read: false
knowledge: false
---
# 

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/qdzZBE73hWvNG9VjIS9sOow1MoYQgibicAhyLCiagyqhMVMBR5LiaLDIwVjiayNbjG4682icRrJOrll5bkJu9kebAQHA/640?wx_fmt=gif&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp#imgIndex=0)

_**前言**_

_Aliware_

# Apache RocketMQ 自诞生以来，因其架构简单、业务功能丰富、具备极强可扩展性等特点被众多企业开发者以及云厂商广泛采用。历经十余年的大规模场景打磨，RocketMQ 已经成为业内共识的金融级可靠业务消息首选方案，被广泛应用于互联网、大数据、移动互联网、物联网等领域的业务场景。由于其业务场景愈加丰富，在工业界的使用率日益提高，开发者们也必须更完善地考虑 RocketMQ 的可靠性、可用性。  

由于 RocketMQ 底层实际上是一种基于日志的存储系统，而前人为了避免这种存储系统中单个机器可能出现的数据丢失、单点故障等问题，已经有了相对成熟的解决方案——例如同时复制数据到多个机器上。在这个过程中，需要解决的问题便被简化了：如何保证多个机器上的数据是一致的，而且这种一致性强大到可以对抗宕机、脑裂等问题。而这些问题，可以通过分布式一致性算法来彻底解决。

**在开源的 Apache RocketMQ 中，我们已经引入了 DLedger**[****2]** 和 SOFAJRaft**[****3]** 来作为 Raft**[****4]** 算法的具体实现，以支撑系统高可用。**本文将介绍 RocketMQ 如何利用 Raft（一种简单有效的分布式一致性算法）进行高可用的保障。

_**分布式一致性算法：Raft**_

_Aliware_

# 共识（Consensus）是分布式系统中实现容错的一个基本问题。它指的是在一个系统中，多个服务器需要就某些值达成一致的意见。一旦它们对一个值做出了决定，这个决定就是不可变更的 **[****1]**。典型的共识算法能够在多数服务器可用的情况下继续运行；比如说，在一个有 5 台服务器的集群中，即使有 2 台服务器宕机，整个集群依然能够正常运作。如果宕机的服务器数量超过半数，集群就无法继续正常运行了（但它也绝不会返回错误的结果）。  

业界比较有名的分布式一致性算法是 paxos**[****12]**，不过可惜的是它比较晦涩难懂，难懂的代价就是很少有人能掌握它然后基于它做出可靠的实现。不过幸好 Raft 及时出现，它易于理解，并且已经有非常多的业界使用先例，比如 tikv、etcd 等。

这是 Raft 的原始论文，详细描述了 Raft：《In Search of an Understandingable Consensus Algorithm》**[****4]**。本文的略短版本在 2014 年 USENIX 年度技术会议上获得了最佳论文奖。有意思的是，在论文的 Abstract 中，第一句话便是：Raft is a consensus algorithm for managing a replicated log.

在英文中，log 不仅有 “日志” 的意思，还有 “木头” 的意思，一组 "replicated log" 便组成了木筏，这和 Raft 的英文原意不谋而合。而且，Raft 的主页中，也采用了三根木头组成的筏作为 logo（如图 1）。当然，对 Raft 更正式的解释还是这些单词的首字母缩写：**R**e{liable | plicated | dundant} **A**nd **F**ault-**T**olerant. 也就是 Raft 被提出时要解决的问题初衷。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/qdzZBE73hWvsVCic3LEmwOjaaFL8hbpKcJAJy2aZOVfgnlxqC5icQYgtUFuC2NWumYhe1IiaFheuMKWXtBaJTibBoA/640?wx_fmt=png&from=appmsg#imgIndex=1)

图 1：Raft 主页标题后附的头图

Raft 算法是一种为了管理复制日志的共识算法，它将整个共识过程分解为几个子问题：领导者选举、日志复制和安全性。整个 Raft 算法因此变得易理解、易论证、易实现，从而让分布式一致性协议可以较为简单地实现。Raft 和 Paxos 一样，只要保证 n/2+1 节点即多数派节点正常就可以对外提供服务。

在 Raft 算法中，集群中的每个节点（服务器）可以处于以下三种状态之一：

a. Follower（跟随者）：这是所有节点的初始状态。它们被动地响应来自领导者和候选者的请求。

b. Candidate（候选者）：当跟随者在一段时间内没有收到来自领导者的消息时，它们会成为候选者，并开始选举过程以成为新的领导者。

c. Leader（领导者）：集群中的管理节点，负责处理所有客户端请求，并将日志条目复制到其他节点。

Raft 算法通过任期的概念来分隔时间，每个任期开始都会进行一次领导者选举。任期是一个递增的数字，每次选举都会增加。如果跟随者在 “选举超时” 之内没有收到领导者的心跳，它会将自己的任期号加一，并转变为候选者状态来发起一次领导者选举。候选者首先给自己投票，并向其他节点发送请求投票的 RPC。如果接收节点在当前任期内还没有投票，它会同意投票给请求者。

如果候选者在一次选举中从集群的大多数节点获得了选票，它就会成为领导者。在此过程中生成的任期号用于节点之间的通信，以防止过时的信息导致错误。例如，如果节点收到任期号比自己小的请求，它会拒绝该请求。

极端情况下集群可能会出现脑裂或网络问题，此时集群可能会被分割成几个互不通信的子集。不过由于 Raft 算法要求一个领导者必须拥有集群大多数节点的支持，这保证了即使在脑裂的情况下，最多只有一个子集能够选出一个有效的领导者。

Raft 通过日志复制来保持节点间的一致性。对于一个无限增长的序列 a[1, 2, 3…]，如果对于任意整数 i，a[i] 的值满足分布式一致性，这个系统就满足一致性状态机的要求 基本上所有的真实系统都会有源源不断的操作，这时候单独对某个特定的值达成一致显然是不够的。为了让真实系统保证所有的副本的一致性，通常会把操作转化为 write-ahead-log(WAL)。然后让系统中所有副本对 WAL 保持一致，这样每个副本按照顺序执行 WAL 里的操作，就能保证最终的状态是一致的，如图 2 所示。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/qdzZBE73hWsAmKbPUcSdBOKcY7tPExdr8AVsprCGnfKcanQ0aFTasMFVsRXmXAOfBVl7SsK2GsCEKfDtQ3Iia1A/640?wx_fmt=other&from=appmsg#imgIndex=2)

图 2：如何通过日志复制来确保节点间数据一致。阶段一为 Client 向 leader 发送写请求，阶段二为 Leader 把‘操作’转化为 WAL 并复制，阶段三为 Leader 收到多数派应答，并将操作应用到状态机

领导者在收到客户端的请求后，会先将请求作为新的日志条目追加到它的日志中，然后并行地将该条目复制到其他节点。只有当大多数节点都写入了这个日志条目，领导者才会将该操作提交，并应用到它的状态机上，同时通知其他节点也提交这个日志条目。因此，Raft 确保了即使在领导者崩溃或网络分区的情况下，也不会有数据丢失。任何被提交的日志条目都保证在后续的任期中也存在于任意新的领导者的日志中。

简单来说，Raft 算法的特点就是 Strong Leader：

a. 系统中必须存在且同一时刻只能有一个 Leader，只有 Leader 可以接受 Clients 发过来的请求；

b. Leader 负责主动与所有 Followers 通信，负责将 “提案” 发送给所有 Followers，同时收集多数派的 Followers 应答；

c. Leader 还需向所有 Followers 主动发送心跳维持领导地位（保持存在感）。

一句话总结 Strong Leader: "你们不要 BB! 按我说的做，做完了向我汇报!"。另外，身为 Leader 必须保持一直 BB（heartbeat）的状态，否则就会有别人跳出来想要 BB 。

为了更直观的感受到 Raft 算法的运行原理，笔者强烈推荐观看下面网站中的演示。它以动画的形式，非常直观地展示了 Raft 算法是如何运行的，以及如何应对脑裂等问题的：_https://thesecretlivesofdata.com/raft/_

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/qdzZBE73hWsAmKbPUcSdBOKcY7tPExdr3AGklzetMQ14p3bWC4eGK5N1uavjhWo9WLbKbvoTdicPBDiauib9xIHdA/640?wx_fmt=gif&from=appmsg#imgIndex=3)

图 3：Raft 算法选举过程图示

相信观看过上面网站中的图解后，读者应该了解了 Raft 的设计思想与具体算法，下面我们直接切入正题，讲解 RocketMQ 与 Raft 的前世今生。

_**RocketMQ 与 Raft 的前世今生**_

_Aliware_

# RocketMQ 尝试融合 Raft 算法已经非常之久，这期间的融合方式也经历过变革。发展至今，Raft 也只是 RocketMQ 高可用机制中的一小部分，RocketMQ 已然发展出了一套适合自身的高可用共识协议。  

本章主要阐述 RocketMQ 为了在系统内实现 Raft 算法作出过哪些尝试，以及当前 Raft 在 RocketMQ 中的存在形态与具体作用。

**Raft 在 RocketMQ 中的初期形态**

RocketMQ 引入 Raft 协议的主要原因是为了增强系统的高可用性和故障自动恢复能力。在 4.5 版本之前，RocketMQ 只有 Master/Slave 一种部署方式，即一组 broker 中仅有一个 Master，有零到多个 Slave，这些 Slave 以同步或者异步的方式去复制 Master 中的数据。然而这种方式存在一些限制：

1.  **故障转移不是完全自动的：**当 Master 节点出现故障时，需要人工介入进行手动重启或者切换到 Slave 节点。
    
2.  **对外部依赖较高：**虽然可以通过第三方协调服务（如 ZooKeeper 或 etcd）实现自动选主，但这增加了部署和运维的复杂性，同时第三方服务本身的故障也可能影响到 RocketMQ 的集群。
    

为了解决上述问题，RocketMQ 引入了基于 Raft 协议的 **DLedger 存储库** **[****5]**。DLedger 是一个分布式日志复制技术，它使用 Raft 协议，可以在多个副本之间安全地复制和同步数据。RocketMQ 4.5 版本发布后，可以采用 RocketMQ on DLedger 方式进行部署。DLedger commitlog 代替了原来的 CommitLog，使得 CommitLog 拥有了选举复制能力，然后通过角色透传的方式，raft 角色透传给外部 broker 角色，leader 对应原来的 master，follower 和 candidate 对应原来的 slave：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/qdzZBE73hWsAmKbPUcSdBOKcY7tPExdrTEdmkfGMswI3HoxMOuw3IdkNniauUiaobWwMO6ic93L04PxoiaztBLGQmA/640?wx_fmt=png&from=appmsg#imgIndex=4)

图 4：RocketMQ on DLedger 部署形态，每个 broker 间的角色由 Raft CommitLog 向外透传

因此 RocketMQ 的 broker 拥有了自动故障转移的能力。在一组 broker 中， Master 挂了以后，依靠 DLedger 自动选主能力，会重新选出 leader，然后通过角色透传变成新的 Master。DLedger 还可以构建高可用的嵌入式 KV 存储。我们把对一些数据的操作记录到 DLedger 中，然后根据数据量或者实际需求，恢复到 hashmap 或者 rocksdb 中，从而构建一致的、高可用的 KV 存储系统，应用到元信息管理等场景。

我们测试了各种故障下 Dledger 表现情况，包括随机对称分区，随机杀死节点，随机暂停一些节点的进程模拟慢节点的状况，以及 bridge、partition-majorities-ring 等复杂的非对称网络分区。在这些故障下，DLedger 都保证了一致性，验证了 DLedger 有很好可靠性 **[****6]**。

总结来说，引入 Raft 协议后，RocketMQ 的多副本架构得以增强，提高了系统的可靠性和自我恢复能力，同时也简化了整个系统的架构，降低了运维的复杂性。这种架构通过 Master 故障后短时间内重新选出新的 Master 来解决单主问题，但是由于 Raft 选主和复制能力一同在数据链路（CommitLog）上，因此存在以下问题：

1.  Broker 组内的副本数必须是 3 副本及以上才有切换能力，因此部署的最低成本是有上升的。
    
2.  Raft 多数派限制导致三副本副本必须两副本响应才能返回，五副本需要三副本才能返回，因此 ACK 是不够灵活的，这也导致 “发送延迟低” 和“副本冗余小”两种要求很难做到。
    
3.  由于存储复制链路用的是 OpenMessaging DLedger 库，导致 RocketMQ 原生的一些存储能力没办法利用，包括像 TransientPool、零拷贝的能力，如果要在 Raft 模式下使用的话，就需要移植一遍到 DLedger 库，开发特性以及 bug 修复也需要做两次，这样的维护和开发成本是非常高的。
    

此外，将选举逻辑嵌入数据链路中可能会引发一连串的问题，这直接与我们追求的高可用和稳定性目标背道而驰。以选举发生在数据链路中的假设情景为起点，我们可以设想一个由多个节点构成的存储集群，其中节点需要定期进行选举来决定谁负责数据流的管理任务。在这种设计下，选举不仅是控制面的一部分，而且直接影响数据链路的稳定性。一旦发生选举失败或不一致的情况，整个数据链路可能会受阻，导致写入能力的丧失或数据丢失。

但是我们将目光看向 PolarStore 时就能发现，它的设计思想包含了 “控制面和数据面分离”：数据面操作仅依赖于本地缓存的全量元数据，而对控制面的依赖最小化。这种设计的优势在于即使控制面完全不可用，数据面依然能够依据本地缓存维持正常的读写操作。在这种情况下，控制面的选举机制永远不会影响到数据面的可用性。这种分离架构为存储系统带来了相当强的鲁棒性，即使在遭遇故障时也能够保持业务的连续性。

总而言之，将选举逻辑与数据链路解耦是保障存储系统高可用性和稳定性的关键。通过将控制面的复杂性和潜在故障隔离，可以确保即使在面临控制面故障时，数据面依然能够保持其核心功能，从而为用户提供持续的服务。这种健壮的设计理念在现代分布式存储系统中是至关重要的——在控制面遭遇问题时，数据面能够以最小的影响继续运作。

这个例子告诉我们，数据面的可用性如果和控制面解耦，那么控制面挂掉对数据面的影响很轻微。否则，要么要不断去提高控制面的可用性，要么就要接受故障的级联发生 **[****7]**。这也是我们后文中 RocketMQ 的演进方向。

**现在的 Raft Controller：控制面 / 数据面分离**

上文中提到，我们以 DLedger 的形式将 Raft 引入了 RocketMQ，但这种引入实际上是给了 CommitLog 选举的能力。这种设计固然直接有效，能够直接赋予一致性给最重要的组件。但是这样的设计也让选举、复制的过程被耦合到了一起。当二者耦合时，每次的选举、拷贝便都强依赖 DLedger 的 Raft 实现，这对于未来的扩展性是非常不友好的。

那么，有没有更可靠、更灵活的解决方案呢？我们不妨把目光转向学术界，看看他们的灵感。在 OSDI' 20 上，Meta 公司管控平面元数据存储统一平台 Delos**[****8]** 相关论文获得了的 Best Paper Award。这篇论文提供了一个全新的视角与思路来解决选主过程中 “控制面与数据面耦合” 的问题：虚拟共识（Virtual Consensus）。它在论文中描述了在生产环境中实现在线切换共识协议的工作。这种设计旨在通过虚拟化共享日志 API 来实现虚拟化共识，从而**允许服务在不停机的情况下更改共识协议。**

论文的出发点是，**在生产环境中，系统往往高度集成了共识机制，因此更换共识协议会涉及到非常复杂且深入的系统改动。**以前文提到的 DLedger CommitLog 为例，其分布式共识协议中的数据流（负责容错）和控制流（负责同步共识组配置）是密切相关的。在这种紧密耦合的系统中进行修改是极其困难的，这就使得开发和实施新共识协议的代价变得相当昂贵，甚至单纯的更新微小特性都面临重大挑战。更不用提在这种环境下引入更先进的共识算法（未来如果有的话），这必然花费相当重大的成本。

相比之下，Delos 提出了一种新方法，通过分离控制层和数据层来克服这些挑战。在这个架构中，控制层负责领导选举，而数据层则由 VirtualLog 和下层的 Loglets 组成，用于管理数据。VirtualLog 提供了一个共享日志的抽象层，将不同 Loglets（代表不同共识算法实例）串联起来。这种抽象层在 VirtualLog 和各个 Loglet 之间建立日志条目的映射关系。要切换到新的共识协议时，简单地指示 VirtualLog 将新的日志条目交由新 Loglet 处理以达成共识即可。**这种设计实现了共识协议的无缝切换，并显著降低了更换共识协议的复杂性和成本。**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/qdzZBE73hWsAmKbPUcSdBOKcY7tPExdrRNjSEMOeE4WibFSsZmeFoZ7bUcF9LMkhzcwvlicdZ1dC3kyrvib776HqQ/640?wx_fmt=png&from=appmsg#imgIndex=5)

图 5：Delos 设计架构示意图，控制面和数据面被分开，以 VirtualLog 的形式进行协作

这种设计既然已经在学术界开诚布公，那 RocketMQ 也可以在工业界将其落地并作验证。在 RocketMQ 5.0 中，我们提出了 Raft Controller 的概念：**仅在上层协议中使用 Raft 与其它选主算法，而下层数据链路的复制则由 Broker 中的一套数据复制算法负责，用于响应上层的选主结果。**

这个设计理念在行业中其实并不罕见，反而已经成为了一种成熟且广泛被验证的实践。以 Apache Kafka 为例，这个高性能的消息传递系统采用了分层的架构策略，在早期版本中使用了 ZooKeeper 来构建其元数据的控制平面，这一控制平面负责管理集群的状态和元数据信息。随着 Kafka 的发展，新版本引入了自研的 KRaft 模式，进一步内部化并提升了元数据管理。此外，Kafka 的 ISR（In-Sync Replicas）**[****15]** 协议承担了数据传输的重任，提供了高吞吐量和可配置的复制机制，确保了数据平面的灵活性和可靠性。

同样地，Apache BookKeeper，一个低延迟且高吞吐的存储服务，也采用了类似的架构思想。它利用 ZooKeeper 来管理控制平面，包括维护一致的服务状态和元数据信息。在数据平面方面，BookKeeper 利用其自研的 Quorum 协议来处理写操作，并确保读操作的低延迟性能。

类似这种设计，我们的 Broker 不再自己负责自己的选举，而是由上层 Controller 对下层的角色进行指示，下层根据指示结果进行数据的拷贝，从而达到**选举与复制分离**的目的。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/qdzZBE73hWsAmKbPUcSdBOKcY7tPExdrxmu9xwYU82ibcL7GTlgZyQ0AXLAd26SJNTWALVaiaS1Combf8hI0HJtg/640?wx_fmt=png&from=appmsg#imgIndex=6)

图 6：来源于 ATA 文章《全新 RocketMQ 5.0 高可用设计解读》

不过与 Delos 不同的是，我们在这里面其实有三层共识——Controller 间的共识协议（Raft），Controller 对 Broker 选主时的共识协议（Sync-State Set），Broker 间复制时的共识协议（主备确认复制算法）。我们在这个过程中额外增加了 Controller 间的共识，以保证控制节点也是强一致的。这三种共识算法具体实现在这里不加以赘述，有兴趣的可以看我们 RocketMQ 社区中的 RIP-31/32/34/44 几个说明文档。

这个设计也在我们被 ASE 23' 录用的论文 **[****9]** 中得以体现：Controller 组件承担了切换链路中的核心角色，但是又不影响数据链路的正常运行，即便其面临宕机、夯机、网络分区等问题，也不会导致 broker 的数据丢失、不一致。我们在后续测试中甚至模拟了更加严苛的场景，例如 Controller 与 Broker 同时宕机、同时夯机、同时进行随机网络分区等等，我们的设计均有非常好的表现。

下面，我们对 RocketMQ 中的共识协议作展开，深入地剖析 RocketMQ 中的共识是如何实现的。

_**RocketMQ 中的共识协议**_

_Aliware_

# 首先我们在这里放一张大图，用于描述 Raft Controller 具体是如何实现控制面、数据面的共识的。  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/qdzZBE73hWsAmKbPUcSdBOKcY7tPExdrhdv6bdEZ0u4ibW4kbnnam2GajZiclE7YvefwcNSlg47UNS81vYTuGoOA/640?wx_fmt=png&from=appmsg#imgIndex=7)

图 7：Raft Controller 具体设计架构及其运作原理

上图中，绿色部分的是 Controller，也就是 RocketMQ 划分出来的控制面。它自身包含了两种共识算法，分别是保证 Controller 自身共识的 Raft 算法。以及保证 Broker 共识的选主算法，Sync-State Set（后文简称 3S）算法，这个算法是我们参考 PacificA 算法 **[****10]** 提出的一套用于数据面选主的分布式共识协议。红色部分是 Broker，也就是我们最核心的数据面，这里面忽略了 Broker 的其它存储结构，仅保留复制过程中实现共识的核心文件：epoch 文件。我们基于它实现了数据复制过程中的共识协议。

下面我们针对控制面和数据面的共识，分别进行阐述。

**控制面共识**

控制面共识共有两层：

a. Raft Controller 自身的共识——用于保证 Controller 间的数据强一致性

b. 3S 算法——用于保证 Broker 选主结果的强一致。

下面对这两种共识算法分别进行解释。

### Raft In Controller

Raft 在 RocketMQ 5.0 前，只在 CommitLog 中存在，以 DLedger CommitLog 的方式向外透传角色。但是经过我们前文的分析，可以知道这种方式是有不容忽视的弊端在的：选举、复制过程强耦合，复制过程强依赖 DLedger 存储库，迭代难度高。因此，我们考虑将 Raft 的能力向上移动，让其用于在控制面中实现原数据强一致。这样一来，选举的过程便与日志复制的过程区分开了，而且每次选举的成本相对更低，只需要同步非常有限的数据量。

在 Controller 中，我们将 Raft 算法用于选举 Controller 中的 Active Controller，由它来负责处理数据面的选举、同步等任务，其余的 Controller 则只负责同步 Active Controller 的处理结果。这样的设计能够保证 Controller 本身也是高可用的，且保证了仅有一个 Controller 在处理 Broker 的选举事务。

在最新的 RocketMQ 中，在这一层共提供了两种 Controller 内的分布式共识算法的实现：DLedger 与 JRaft。这两种共识算法可以在 Controller 的配置文件中被非常简单地选择。本质上来说，这两者都是 Raft 算法的具体实现，只不过具体的实现方式有些差异：

a. DLedger**[****2]** 是一个基于 raft 协议的 commitlog 存储库，是一个 append only 的日志系统，早期针对 RocketMQ 的诸多场景有过相当多的适配。同时，它是一个轻量级的 java library。它对外提供的 API 非常简单，append 和 get。

b. JRaft 全称为 SOFAJRaft**[****3]**，它是一个基于 Raft 一致性算法的生产级高性能 Java 实现，支持 MULTI-RAFT-GROUP，适用于高负载低延迟的场景。SOFAJRaft 是从百度的 braft**[****11]** 移植而来的，做了一些优化和改进。

这两种 Raft 的具体实现都对外提供了非常简单的 API 接口，所以我们可以把更多的精力放在处理 Active Controller 的事务上。

### 3S Algorithm

抛开 Controller 本身的共识算法，我们将目光聚焦于 Active Controller 在整个过程中起的作用，这也是我们控制面共识的核心——3S 算法。

3S 算法中的 Sync-State Set 概念与 Kafka 的 In-Sync Replica（ISR）**[****17]** 机制类似，都参考了微软的 PacificA 算法。与以分区为维度的 ISR 不同，3S 算法以整个 Broker 的维度发起选举，且针对 RocketMQ 的需要选举场景作了系统的归纳。相比较来说，3S 算法更加简单，选举更加高效，面对大量分区场景能有更加强大的表现。

3S 算法主要作用在 Controller 与 Broker 的交互过程中，Active Controller 会处理每个 Broker 的心跳与选举工作。和 Raft 类似的，3S 算法也有心跳机制来实现类似租约的功效——当 Master Broker 一定时长没有上报心跳，就可能触发重新选举。不过与 Raft 不同的是，3S 算法有一个共识处理的核心：Controller。这种中心化的设计能够让数据面的选主更加简单，达成共识更加迅速。

在这种设计下，Broker 的心跳不再向同级别 (数据面) 发送，而是统一向上 (控制面) 发送。发送选举请求后，由 Controller 来决定哪个 Broker 可以作为 Master 存在，而其它 Broker 自然退化为 Slave。Controller 的选择原则可以是多样的（同步进度、节点资源等指标），也可以是简单有效的（数据同步进度达到一定阈值），只需要这个节点位于 Sync-State Set 中。这也是一种 Strong Leader 的形式，只不过和 Raft 不同的地方在于：

**Raft 像是小组作业，同学们（Broker）互相投票进行小组长的票选，而 3S 算法则由班主任（Controller）根据举手快慢直接任命。**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/qdzZBE73hWsAmKbPUcSdBOKcY7tPExdrHW2N8zhbXhic9NyhHKLoWaMdH9BP8uJdbIUCrdeW4Cwt5vNR8f5Nopg/640?wx_fmt=png&from=appmsg#imgIndex=8)

图 8：多个 Broker 集群向 Active Controller 汇报集群内的主备角色以及同步情况

如上图所示，三个 Broker 集群中的 Leader 都会定期向 Active Controller 上报集群的同步状态：

a. A 集群的所有节点的同步进度都很良好，因此 Leader 上报的 Sync-State Set 是所有节点。

b. B 集群的 Follower-2 可能刚刚启动，仍旧在同步历史消息，因此 Sync-State Set 并不会包含它——当 Leader 宕机时，Controller 自然也不会选择它。

c. C 集群中，虽然 1 号 Leader 已经宕机，但是 Controller 迅速便能决定 Sync-State Set 中的 3 号节点作为替代，提拔为主节点，整个集群便能正常运转，此时，即便 3 号节点又宕机，也能选择 2 号节点为主节点，不影响集群运行状态。

这种设计的好处在哪里呢？Raft 算法的实现原理其实是 “投票”，同学间彼此平等，靠投票结果 “少数服从多数”。因此，对于一个有 2n+1 节点的集群来说，Raft 最多只能容忍 n 个节点失效，至少需要保证有 n+1 个节点是持续运行的。但是 3S 算法有一个选举中心，每次选举的 RPC 都向上发送，它不需要得到其它节点的认可便可选举出一个节点。因此对于之前提到的 2n+1 节点的集群来说，最多能容忍 2n 个节点的失效，即副本的数量不需要超过副本总数的一半，不需要满足 “多数派” 原则。通常，**副本数大于等于 2 即可，**如此，便在可靠性和吞吐量方面取得平衡。

上面的例子比较简明扼要地介绍了 3S 算法和 Raft 的关系与不同，可以说 3S 算法的设计思想来源于 Raft，但是在特定场景下又优于 Raft。

此外，3S 算法的共识以整个 Broker 为维度，因此我们对选主时机作了优化，有如下几种情形可能触发选举，括号内的红色是更加形象的描述，将选主过程具像化为选小组长的过程，以便理解：

a. 控制面，Controller 主动发起**（班主任发起）**：

i.HeartbeatManager 监听到有 broker 心跳失效。(班主任发现有小组同学退学了)

ii.Controller 检测到有一组 Sync-State Set 不存在 master。（班主任发现有组长虽然在名册里，但是旷课了）

b. 数据面，Broker 发起将自己选为 master**（同学毛遂自荐）**：

i. Broker 向 controller 查询元数据时，没找到 master 信息。（同学定期检查小组情况，问班主任为啥没小组长）

ii. Broker 向 controller 注册完后，仍未从 controller 获取到 master 信息。（同学报道后发现没小组长，汇报并自荐）

c. 运维侧，通过 RocketMQ Admin Tools 发起，是运维能力的一部分**（校长直接任命）。**

通过上述两方面优化，3S 算法在 RocketMQ 5.0 中，展露了非常强大的功能性，让 Controller 成为了高可用设计范式中不可或缺的组件。其实 3S 算法在实际使用场景中还有很多细节上的处理优化，能够容忍前文提到的更加严峻的场景：如控制面和数据面同时发生故障，且故障节点超过一半以上的场景。这部分结果会在后文的混沌实验中得以展示。

**数据面共识**

控制面通过 Raft 算法保障了 Controller 间的角色共识，以及通过 3S 算法保障了 Broker 中的角色共识。那么在 Broker 角色被确定后，其数据面该如何根据选举结果保障数据的强一致呢？这里的算法并不复杂，因此笔者从实现角度介绍一下 RocketMQ 的设计，RocketMQ 的数据面共识主要由下面两个组件构成：

*   **HAClient：**每个 Slave 的 HAService 中必备的 client，负责管理同步任务中的读、写操作。
    
*   **HAConnection：**代表在 Master 中的 HA 连接，每个 connection 理论上对应一个 slave。在该 connection 类中存储了传输过程中的诸多内容，包括 channel、传输状态、当前传输位点等等信息。
    

为了更形象地描述清楚 RocketMQ 在这方面的设计结构，笔者绘制了下面这幅图，可以看出核心还是数据的传输过程，分别设计了一个 Reader 与一个 Writer：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/qdzZBE73hWsAmKbPUcSdBOKcY7tPExdrkPia6rL9VUdCxpoJWTludsygyuYCmf6mh4lBLjrg2FqY7icKl4tWC5Vg/640?wx_fmt=png&from=appmsg#imgIndex=9)

图 9：数据面复制过程的具体实现，Master 与 Slave 分别设计，但选举完成后可互相切换

**这么简单的设计，是如何确保数据写入时的强一致的呢？**

核心的共识其实存在于 HAConnection（也就是图中左下角那个深蓝色框）的建立、维护过程中。每个 Broker 集群的主节点都会维护和所有 Slave 的连接关系，并将其存于 Connection 表中，在每次 Slave 来请求代复制数据后，都会反馈复制的最后位点与结果，因此主节点也可以基于此来确定上报给 Controller 的 Sync-State Set。在 HAConnection 的建立过程中，有一个确保数据一致性的 HandShake 阶段。这个阶段能够对 CommitLog 作截断，从而保障复制位点之前的所有数据都是强一致的。这个过程通过 epoch 文件的标记实现：Epoch 文件中包含了每一次选举的状态，每次选举完成后，主节点都会在 epoch 文件上留下自己的痕迹，即当前的选举代数 + 当前的初始位点。

从这里也可以看到，我们数据面的共识算法也有一些 Raft 的影子：Raft 算法在每次选举后也会给任期数自增一，这个任期数的大小决定了后续选主的权威性。而在数据面共识算法中，选主的结果已经认定，任期数被用于多次选主结果的共识表征——当任期数与日志位点一致时，代表这两台 broker 就选主这件事达成过一致，因此可以认为此前的数据是强一致的，只需要保证后续数据的强一致即可。为了方便理解，可以通过下图进行描述：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/qdzZBE73hWsAmKbPUcSdBOKcY7tPExdrh58rDxSdKn2cy3RTGRSQEuibRx0p7XkW63j381sBFNOw4v6AVyyXjgw/640?wx_fmt=png&from=appmsg#imgIndex=10)

图 10：来源于文章《全新 RocketMQ 5.0 高可用设计解读》

类似上面这张图，最上面那个方块长条实际是 RocketMQ 的日志存储形态 MappedFile。下面两条方块组成的长条分别是主和备的 commitlog，备节点会从后向前找到最大的 <epoch, startOffset> 一致的位点，然后截断到这个位点，开始向后复制。这种复制在 RocketMQ 中有单独的一个 Service 去执行，因此主备节点的复制和选举过程其实是彻底解耦开的，只有当一个备节点尽可能跟上主节点时，这个备才会被纳入 Sync-State Set，后续才有资格参加选举。

_**拥抱故障，把故障当作常态**_

_Aliware_

# 俗话说的好，“空谈误国，实干兴邦”，设计究竟是先进还是冗余，需要通过各方面的检验。  

对于 RocketMQ 这种大规模在生产中被使用的系统，我们必须模拟出足够接近现实情况的故障，才能检验其可用性在现实场景中究竟如何。在这里，我们需要引入一个新的概念——混沌工程。

**混沌工程的原始定义 **[1****3]** 为：**“Chaos engineering is the discipline of experimenting on a system in order to build confidence in the system's capability to withstand turbulent conditions in production.”

从原始定义看，混沌工程实际上是一种软件工程方法，旨在通过在软件系统的生产环境中故意引入混乱来验证系统的可靠性。混沌工程的基本假设是，生产环境是复杂且不可预测的，而通过模拟各种失败，可以发现并解决潜在的问题。这种方式有助于确保系统能够在面临真实世界中的各种挑战时，持续并有效地运行。

大家都写过代码，也都深知一个精心设计过的系统总是能够巧妙通过各个测试样例，但是上线后总会遇到各种问题。因此对于一个系统来说，能够出色地通过复杂且不可预知的频繁故障的混沌工程的考验，而不是测试样例，才能说明这个系统是高可用的。

下面我们将详细介绍，我们为了验证 RocketMQ 的高可用，对其作过哪些 “拷打”。

**OpenChaos**

OpenChaos**[****14]** 作为云原生场景量身定制的混沌 “刑具”，位于 OpenMessaging 名下，托管于 Linux 基金会。目前，它支持以下平台的混沌测试：Apache RocketMQ、Apache Kafka、DLedger、Redis、Zookeeper、Etcd、Nacos。

目前 OpenChaos 支持注入多种故障类型，其中最主要的便是：

1.  random-partition (fixed-partition)：随机（固定）隔离节点与网络的其他部分。
    
2.  random-loss：随机选定的节点丢失网络数据包。
    
3.  random-kill (minor-kill, major-kill, fixed-kill)：终止随机（少数、多数、固定）的进程并重启它们。
    
4.  random-suspend (minor-suspend, major-suspend, fixed-suspend)：使用 SIGSTOP/SIGCONT 暂停随机（少数、多数、固定）的节点。
    

在实际场景中，最常见的故障就是这四种：网络分区、丢包、宕机、夯机。此外，OpenChaos 还支持其它更复杂的特定场景，例如 Ring（每个节点能够看到大多数其他节点，但没有节点能看到与任何其他节点相同的多数节点）和 Bridge（网络一分为二，但保留中间的一个节点，该节点与两边的组件保有不间断的双向连通性），形成条件非常严苛，而且它们阻碍共识生效的原理都是 “通过阻碍各节点间的可见性，来避免形成全局多数派”，理论上来说，通过足够久的网络分区、丢包，也能模拟出这些情况，甚至更复杂的情况。

因此，我们注入了大量上面罗列的四种混沌故障，观察集群是否有出现消息丢失的情况，并统计了故障恢复时间。

**具体测试场景**

我们混沌测试的验证实验环境如下：

a. namesrv 一台，内含 namesrv 进程，openchaos 的混沌测试进程也在该机器上启动，向 controller/broker 发出控制指令。

b. controller 三台，内含 controller 进程。

c. broker 三台，同属一个集群，分别为主备，内含 broker 进程。

上述 7 台机器的配置为，处理器：8 vCPU，内存：16 GiB，规格族：ecs.c7.2xlarge，公网带宽：5Mbps，内网带宽：5/ 最高 10 Gbps。

在测试中，我们设置了如下的若干种随机测试场景，每种场景都会持续至少 60 秒，且恢复后会保证 60 秒的时间间隔再注入下一次故障：

a. 机器宕机，这个混沌故障注入通过 kill -9 命令实现，将会杀死范围内的随机进程。

i. Broker 节点，随机宕机一半以上的节点，至少保留一台 Broker 工作。

ii. Controller 节点，随机宕机一半以上的节点，以及全部宕机。

iii. Broker+Controller 节点，随机宕机一半以上的节点。

b. 机器夯机，这个混沌故障注入通过 kill -SIGSTOP 命令实现，模拟进程暂停的情况。

i. Broker 节点，随机夯机一半以上的节点，至少保留一台 Broker 工作。

ii. Controller 节点，随机夯机一半以上的节点，以及全部夯机。

iii. Broker+Controller 节点，随机夯机一半以上的节点。

c. 机器丢包，这个混沌故障注入通过 iptables 命令实现，可以指定机器间特定比例的丢包事件。

i. Broker 间随机丢包 80%。

ii. Controller 间随机丢包 80%。

iii. Broker 和 Controller 间随机丢包 80%。

d. 网络分区，这个混沌故障注入也是通过 iptables 命令实现，能够将节点间完全分区。

i. Broker 间随机网络分区。

ii. Controller 间随机网络分区。

iii. Broker 合 Controller 间随机网络分区。

实际测试场景、组别远多于上述罗列的所有故障场景，但是存在一些包含关系，例如单台 broker/controller 的启停，便不再单独罗列。此外我们均针对 Broker 的重要参数配置进行了交叉测试。测试的开关有：transientStorePoolEnable（是否使用直接内存），slaveReadEnable（备节点是否提供读消息能力）。我们还针对 Controller 的类型（DLedger/JRaft）也进行了分别的测试，每组场景的测试至少重复 5 次，每次至少持续 60 分钟。

**实验结论**

针对上述提出的所有场景，混沌测试的总时长至少有：

12（场景数）* 2（Broker 开关数）* 2 (Controller 类型) * 5（每组测试次数）* 60（单组时长） =**1440****0 分钟**。

由于设置的注入时长一分钟，恢复时长一分钟，因此至少共计注入故障 14400/2 = **7200 次**（实际上注入时长、注入次数远多于上述统计值）。

在这些记录在册的测试结果中，有如下测试结论：

a. RocketMQ 无消息丢失，数据在故障注入前后均保持强一致。

b. 恢复时长基本等于客户端的路由间隔时间，在路由及时的情况下，能够保证恢复 RTO 约等于 3 秒。

c. Controller 任意形式下的故障，包括宕机、夯机、网络故障等等，均不影响 Broker 的正常运转。

_**总结**_

_Aliware_

# 本文总结了 RocketMQ 与 Raft 的前世今生。从最开始的忠实应用 Raft 发展 DLedger CommitLog，到如今的控制面 / 数据面分离，并分别基于 Raft 协议作专属于 RocketMQ 的演化。如今 RocketMQ 的高可用已经逐渐趋于成熟：基于三层共识协议，分别实现 Controller 间、Controller&Broker、Broker 间的共识。  

在这种设计下，RocketMQ 的角色、数据共识被妥善地划分到了多个层次间，并能够彼此有序地协作。当选主与复制不再耦合，我们便能更好地腾出手脚发展各个层次间的共识协议——例如，当出现比 Raft 更加优秀的共识算法时，我们可以直接将其应用于 Controller 中，**且对于我们的数据面无任何影响。**

可以说 Raft 的设计给 RocketMQ 的高可用注入了非常多的养分，能够让 RocketMQ 在其基础上吸纳其设计思想，并作出适合自己的改进。RocketMQ 的共识算法与高可用设计 **[****9]** 在 2023 年也得到了学术界的认可，被 CCF-A 类学术会议 ASE 23' 录用。期待在未来能够出现更加优秀的共识算法，能够在 RocketMQ 的实际场景中被适配、发扬。

* * *

![](https://mmbiz.qpic.cn/mmbiz_png/yvBJb5IiafvmAT560a59RwZ7EH350CHkGN1BHWtNDOabRraMuvxibGcMwCTgfmsYXZEia0EPFuCM77prq1mMM1yBQ/640?wx_fmt=other&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1#imgIndex=11)

若您希望深入了解 RocketMQ，推荐访问并收藏网站：_https://rocketmq.io_

# **参考链接：**

[1] Apache RocketMQ

_https://github.com/apache/rocketmq_

[2] OpenMessaging DLedger

_https://github.com/openmessaging/dledger_

[3] SOFAJRaft

_https://github.com/sofastack/sofa-jraft_

[4] Diego Ongaro and John Ousterhout. 2014. In search of an understandable consensus algorithm. In Proceedings of the 2014 USENIX conference on USENIX Annual Technical Conference (USENIX ATC'14). USENIX Association, USA, 305–320.

[5] OpenMessaging DLedger

_https://github.com/openmessaging/dledger_

[6] DLedger——基于 raft 协议的 commitlog 存储库

_https://developer.aliyun.com/article/713017_

[7] [《从滴滴的故障我们能学到什么》](https://mp.weixin.qq.com/s?__biz=Mzg3MDk3ODA0Mg==&mid=2247488226&idx=1&sn=e029954f869dc030a14b36acf3e19de8&scene=21&token=199006883&lang=zh_CN#wechat_redirect)

[8] Mahesh Balakrishnan, Jason Flinn, Chen Shen, Mihir Dharamshi, Ahmed Jafri, Xiao Shi, Santosh Ghosh, Hazem Hassan, Aaryaman Sagar, Rhed Shi, Jingming Liu, Filip Gruszczynski, Xianan Zhang, Huy Hoang, Ahmed Yossef, Francois Richard, and Yee Jiun Song. 2020. Virtual consensus in delos. In Proceedings of the 14th USENIX Conference on Operating Systems Design and Implementation (OSDI'20). USENIX Association, USA, Article 35, 617–632.

[9] Juntao Ji, Rongtong Jin, Yubao Fu, Yinyou Gu, Tsung-Han Tsai, and Qingshan Lin. 2023. RocketHA: A High Availability Design Paradigm for Distributed Log-Based Storage System. In Proceedings of the 38th IEEE/ACM International Conference on Automated Software Engineering (ASE 2023), Luxembourg, September 11-15, 2023, pp. 1819-1824. IEEE.

[10] PacificA: Replication in Log-Based Distributed Storage Systems. 

_https://www.microsoft.com/en-us/research/wp-content/uploads/2008/02/tr-2008-25.pdf_

[11] braft

_https://github.com/baidu/braft_

[12] Leslie Lamport. 2001. Paxos Made Simple. ACM SIGACT News (Distributed Computing Column) 32, 4 (Whole Number 121, December 2001), 51-58.

[13] Chaos Engineering

_https://en.wikipedia.org/wiki/Chaos_engineering_

[14] OpenMessaging OpenChaos

_https://github.com/openmessaging/openchaos_

[15] What does In-Sync Replicas in Apache Kafka Really Mean?

_https://www.cloudkarafka.com/blog/what-does-in-sync-in-apache-kafka-really-mean.html_