![](https://mmbiz.qpic.cn/mmbiz_gif/j3gficicyOvasIjZpiaTNIPReJVWEJf7UGpmokI3LL4NbQDb8fO48fYROmYPXUhXFN8IdDqPcI1gA6OfSLsQHxB4w/640?wx_fmt=gif)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvauCTKmIUakdm23FkH2QA8wNyhWQkhLdrEXxOJslKicflGTqKdANHNXOcibGhcJ5fWVic0UbTNNhFJ3Vw/640?wx_fmt=png)

作者：koka

最近在 Discord 的技术 blog 看到 Discord 的底层数据存储的演进过程，从最开始的 2015 初用的单个副本集的 MongoDB，2015 年底迁移到 Cassandra，2022 年消息量达到了万亿的级别，他们将存储迁移到 ScyllaDB。本文会介绍 ScyllaDB 的基本架构和原理，分析下 ScyllaDB 能够支持万亿级存储的原因。

Discord 在创建之初采用的是一个单副本集的 MongoDB，[没有使用 MongoDB 的分片](https://discord.com/blog/how-discord-stores-billions-of-messages)，他们给出的理由是当时 MongoDB 分片很难用，而且不够稳定（这里就不去深究了）。消息数到达一亿条时，RAM 里已经存不下这么数据和索引，MongoDB 的延时开始变得不可控。

### 1、Discord 存储迁移之路

#### 1.1、从 MongoDB 到 Cassandra

开始选择新的存储（Cassandra）进行数据迁移，他们认为 Cassndra 是当时（2015 年底）唯一能满足他们要求的数据库（后面也打脸了）。他们对数据库的要求如下：

1.  线性可扩展性——不需要手动进行数据的分片
    
2.  自动故障转移——尽可能的进行自我修复
    
3.  维护成本低——设置好后就能工作，以后数据量增加后只需要增加节点即可。
    
4.  已经被证明有效——他们喜欢采用新技术，但又不是太新
    
5.  可预测的性能——当 API 的响应时间的 P95 超过 80ms 时就会告警，他们也不希望在 Redis 或者在 Memcache 中缓存数据
    
6.  不是 Blob 存储——如果必须不断地反序列化 Blob 并附加到它们，那么每秒写入数千条消息的效果并不好。
    
7.  开源——掌控自己的命运，不想依赖第三方公司
    

理想很丰满现实很骨感，随着业务场景和消息规模的增长，2022 年初 Cassandra 有 177 个节点，拥有数万亿条消息，Cassandra 也出现了严重的性能问题。

在 Cassandra 中，读取比写入更昂贵。写入会附加到提交日志并写入称为内存表的内存结构，最终刷新到磁盘。然而，读取需要查询 memtable 和可能的多个 SSTable（磁盘文件），这是一个更昂贵的操作。用户与服务器交互时的大量并发读取可以使分区成为热点，称之为 “热分区”。当数据集的大小与这些访问模式相结合时，导致 Cassandra 的集群陷入困境。

当遇到热分区时，它经常会影响整个数据库集群的延迟。一个通道和存储桶对接收了大量流量，并且随着节点越来越努力地服务流量并且越来越落后，节点中的延迟将会增加。由于该节点无法跟上，对该节点的其他查询受到影响。由于我们以仲裁一致性级别执行读取和写入，因此对服务热分区的节点的所有查询都会遭受延迟增加，从而导致更广泛的最终用户影响。

集群维护任务也经常造成麻烦。他们很容易在压缩方面落后，Cassandra 会压缩磁盘上的 SSTable 以提高读取性能。不仅的读取成本更高，而且当节点试图压缩时，还会看到级联延迟。

由于 Cassandra 是 Java 开发的，他们还花费了大量时间调整 JVM 的垃圾收集器和堆设置，因为 GC 暂停会导致显着的延迟峰值。

#### 1.2、从 Cassandra 到 ScyllaDB

他们选取的方案是 ScyllaDB，这是一个用 C++ 编写的与 Cassandra 兼容的数据库。它承诺提供更好的性能、更快的修复、通过每核分片架构实现更强的工作负载隔离，以及无垃圾收集器，听起来相当吸引人。它采用 C++ 编译而不是 Java 所以没有垃圾收集器的 GC 暂停问题。

ScyllaDB 也并不是完全没有问题，当以与表排序相反的顺序扫描数据库时，有反向查询性能不足的问题，现在 ScyllaDB 已经优先解决了这个问题。ScyllaDB 同样也存在 “热分区” 的问题，当前还是需要业务通过其他方式去解决。

##### 1.2.1、 “热分区” 问题

Discord 采用的方案是：在 ScyllaDB 和业务服务之间加了一个中介服务（Rust 语言编写），它不包含任何业务逻辑，主要功能就是合并请求。

**合并请求**

如果多个用户同时请求数据库的同一行，那么只会查询数据库一次。第一个发出请求的用户会导致该服务中启动工作任务，后续请求将检查该任务是否存在并订阅它，该工作任务将查询数据库并将该行返回给所有订阅者。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasqwgS0VsKNADeQ6qsde2rxh3iaMLjEkW5NauauxaiaNMzQInruCemKy2BZ9P1LrCico1V1SAbWPNdcQ/640?wx_fmt=png)

**收敛请求**

同时根据一致性 hash 将同类查询请求，比如同一个频道的请求，进一步收敛到中介服务，这个请求合并的效果更好。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/j3gficicyOvasqwgS0VsKNADeQ6qsde2rx9gWecIdTOEFHYiaiant7X7eGc8svLXYWLSZIK1SDFa5QsQfp3aQEGsuQ/640?wx_fmt=other)

##### 1.2.2、迁移效果

将运行 177 个 Cassandra 节点减少到仅运行 72 个 ScyllaDB 节点。每个 ScyllaDB 节点拥有 9TB 磁盘空间，高于每个 Cassandra 节点平均 4TB 的存储空间。177_4-72_9=60T，这么看的话他们的存储空间也节省了一些。在 Cassandra 上获取历史消息的 p99 为 40-125 毫秒，而 ScyllaDB 的延迟为 15 毫秒，消息插入性能从 Cassandra 上的 5-70 毫秒 p99 到 ScyllaDB 上稳定的 5 毫秒 p99。

### 2、ScyllaDB

ScyllaDB 号称是下一代 NoSQL 数据库，采用 C++ 编写充分利用 Linux 底层原语优势，利用现代多核、多处理器 NUMA 服务器硬件，有卓越的性能表现，API 兼容 Cassandra 和 DynamoDB。支持和 Cassandra 一样的 CQL 查询语言和驱动，一样的 SSTable 存储格式。同样也支持和 DynamoDB 一样的 JSON-style 查询和驱动。下列章节的图片来自 ScyllaDB 官方。

#### 2.1 ScyllaDB 架构

##### **2.1.1 ScyllaDB 服务架构**

ScyllaDB 服务架构图如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/j3gficicyOvasqwgS0VsKNADeQ6qsde2rxc84re2HWRNd71UrqCqkZcMmibsv6XdZAB340ibv6Dj2s8ZWNicc1WjTrA/640?wx_fmt=png)

**Cluster (集群)**：Cluster 是一组相互连接的 Node (节点) 组成，这些节点组织成虚拟的环架构。节点间都是平等的关系，没有定义领导者。这样就不会出现单点故障，可以使用多数据中心将数据复制到地理位置上分散的各集群中。

**Node (节点)**：Node 可以是本地服务器，或者是公有云的虚拟机等。整个集群的数据尽可能均匀的分布在这些节点上。此外，ScyllaDB 使用称为虚拟节点 (vNode) 的逻辑单元来更好地分布数据以获得更均匀的性能。集群可以在不同节点上存储相同数据的多个副本以确保可靠性。

**Shard (分片)**：ScyllaDB 进一步划分数据，通过将节点中总数据的片段分配给特定 CPU 及其关联的内存 (RAM) 和持久存储（例如 NVMe SSD）来创建分片。分片主要作为独立运行的单元运行，称为 “无共享” 设计。这大大减少了争用以及对昂贵的处理锁的需求。

##### 2.2 ScyllaDB 数据架构

ScyllaDB 根据其数据模型，我们一般将其称为 “宽列” 数据库，有时也被称为“key-key-value” 数据库反映其分区键和集群键，其数据架构图如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/j3gficicyOvasqwgS0VsKNADeQ6qsde2rx9TJdpd5pXWXdZVlMrSUkEp7dOX5bGicWFAiawW06Cl2EWaiaJMbCJr5OA/640?wx_fmt=other)

**Keyspace (键空间)**: 数据的顶级容器（表的集合）：定义 ScyllaDB 中保存的数据的复制策略和复制因子 (RF)。例如，用户可能希望存储相同数据的两个、三个甚至更多副本，以确保在一个或多个节点丢失时其数据仍然安全。

**Table (表)**：在键空间内，数据存储在单独的表中。表是由列和行组成的二维数据结构。与 SQL RDBMS 系统不同，ScyllaDB 中的表是独立的，不能跨表进行 JOIN。

**Partition (分区)**：ScyllaDB 中的表可能非常大，通常以 TB 为单位。因此，表被分为更小的块（称为分区），以便尽可能均匀地分布在分片上。

**Rows (行)**：每个分区包含按特定顺序排序的一行或多行数据。并非每一列都出现在每一行中。这使得 ScyllaDB 能够更有效地存储所谓的 “稀疏数据”。

**Colums (列)**：表行中的数据将分为列。特定的行和列条目将被称为单元格。某些列将用于定义数据的索引和排序方式，称为分区键和聚类键

ScyllaDB 包含查找可能导致性能问题的特别大分区和大行的方法。ScyllaDB 还将最常用的行缓存在基于内存的缓存中，以避免在所谓的 “热分区” 上进行昂贵的基于磁盘的查找。

##### **2.3 环架构**

Ring Architecture 的示意图如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/j3gficicyOvasqwgS0VsKNADeQ6qsde2rx9mufia3q0SY4nibic1myCw3H5e3TKYXicobAzEEiaKHKtYyJOD8ma2ia6gNQ/640?wx_fmt=other)

**Ring (环)**：ScyllaDB 中的所有数据都可以可视化为令牌范围环，每个分区映射到单个散列令牌（相反：一个令牌可以与一个或多个分区关联）。这些令牌用于在集群中分发数据，在节点和分片之间尽可能均匀地平衡数据。

**vNode (虚拟节点)**：该环被分成 vNode（虚拟节点），其中包含分配给物理节点或分片的一系列令牌。根据为键空间设置的复制因子 (RF)，这些 vNode 在物理节点上复制多次。

##### 2.4 存储 5 架构

存储架构的示意图如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/j3gficicyOvasqwgS0VsKNADeQ6qsde2rxbRmbUncDMOQjwfALvV9HLtEDueh5ZicnIlCVjibj3FeZZtcPBRjzmIjw/640?wx_fmt=other)

**Memtable**：在 ScyllaDB 的写入路径中，数据首先放入内存表中，存储在 RAM 中。这些数据会及时刷新到磁盘以进行持久化。

**Commitlog**：本地节点操作的仅附加日志，在数据发送到内存表时同时写入。这在节点关闭的情况下提供持久性（数据持久性）；当服务器重新启动时，提交日志可用于恢复内存表。

**SSTables**：在 ScyllaDB 中使用排序字符串表（SSTables）形式对每个分片的数据永久存储。SSTables 采用 LSM 格式，只读且不可更改。一旦数据从内存表刷新到 SSTable，内存表（以及关联的提交日志段）就可以被删除。对记录的更新不会写入原始 SSTable，而是记录在新的 SSTable 中。ScyllaDB 具有了解特定记录的哪个版本是最新版本的机制。

**Tombstones (墓碑)**：当从 SSTable 中删除一行时，ScyllaDB 会将一个称为墓碑的标记放入新的 SSTable 中。这可以提醒数据库忽略被删除的原始数据。

**Compactions**：将多个 SSTable 写入磁盘后，ScyllaDB 知道要运行压缩，这是一个仅存储记录的最新副本的过程，并删除任何标有墓碑的记录。一旦新的压缩的 SSTable 被写入，旧的、过时的 SSTable 就会被删除，并释放磁盘上的空间。

**Compaction Strategy**：ScyllaDB 使用不同的算法（称为策略）来确定何时以及如何最好地运行压缩。该策略决定了写入、读取和空间放大之间的权衡。ScyllaDB Enterprise 甚至支持一种称为增量压缩策略的独特方法，该方法可以显着节省磁盘开销。

#### 2.2 Shard-per-Core Architecture

ScyllaDB 是实时大数据 NoSQL 数据库，采用 C++ 从头开始构建，具有如何利用现代多核、多处理器 NUMA 服务器硬件和 Linux 操作系统基本功能的完整知识和经验。ScyllaDB 是一个大规模并行数据库引擎，它在服务器的每个核心上分片运行，跨集群中的所有服务器。其设计使 ScyllaDB 能够以亚毫秒级的平均延迟每秒运行数百万次操作。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/j3gficicyOvasqwgS0VsKNADeQ6qsde2rxlVvUjdzE13GSCFN97hA0oJKDvibNAzG88ficStQInjW1fFC0w1Y2m54Q/640?wx_fmt=other)

**Modern Shared-Nothing Architecture**

ScyllaDB 基于其底层 [Seastar](https://seastar.io/) 框架，采用高度异步、无共享设计。每个数据分片都分配有 CPU、RAM、持久存储和网络资源，并尽可能高效地使用这些资源。凭借其自己的用于 CPU 和 I/O 处理的自定义调度程序，ScyllaDB 知道如何从大数据基础设施中获得最大效率。

#### 2.3 高可用

##### **2.3.1 peer-to-peer 架构**

当 ScyllaDB 启动时，节点使用 gossip 协议来发现对等节点以建立集群 (进行拓扑和模式更新)。没有领导者也没有追随者，底层架构是无领导者，没有初选，也没有副本。事实上，在 ScyllaDB 中甚至删除了其他 gossip 实现中的种子节点的概念。它完全是点对点的。这种八卦机制还可以在拓扑发生变化的情况下使用，例如添加或删除节点，或者在节点意外中断的情况下，为 ScyllaDB 集群提供强大的弹性。这里新版本 5.2 以后版本有点不一样，引入了 raft 算法来保证拓扑更新的一致性。

##### **2.3.2 Automatic Data Replication**

ScyllaDB 允许用户设置复制因子（RF），这意味着相同数据的多个副本可以存储在集群中的多个节点上。这样，即使某个节点丢失，数据仍然驻留在集群的某个地方。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/j3gficicyOvasqwgS0VsKNADeQ6qsde2rxvN2z4UNjUajQmjwF78pDVGKkJYA90Qb8nj6aYzj72CsyDJz3LKxdXA/640?wx_fmt=other)

对于许多高可用性用例，将复制因子设置为三 (3) 就足够了。在这种情况下，即使三个数据副本中的两个不可用，数据也会驻留在集群中的某个位置。

通过正确设置复制因子，可以实现零停机。用户可以根据自己的用例确定自己的复制因子。有时，复制因子为 2 就足够了，而有时，复制因子可能需要为 5。ScyllaDB 自动负责在后台复制数据。您只需设置复制因子，集群就会处理其余的事情。

##### **2.3.3 ScyllaDB 与 CAP 理论**

CAP 定理基于这样的假设：系统可以选择提供一致性、可用性或分区容错性，并且数据库设计者必须选择这三个特征中的两个。任何分布式数据库都需要分区容错性——即使系统的一部分由于网络或服务器故障而脱机，也能够继续运行。因此，当今有两种流行的数据库模式：CP 或者 AP。

ScyllaDB 一般来说属于 AP，更加侧重于可用性和分区容错性，**但是 ScyllaDB 的一致性级别是可以调整的**。

##### **2.3.4 Tunable Consistency（可调一致性）**

ScyllaDB 中的一致性是可调的——用户可以允许他们的事务具有不同程度的一致性。其中的一些策略如下：

**ONE**：写入任何一个节点成功就算成功

**QUORUM**：写入大多数节点成功才算成功

**ALL**：写入所有节点成功才算成功

**实现零停机：**

节点可能会失败。机架可能会发生故障。甚至整个数据中心。然而您的应用程序却不能。它们始终保持在线状态。这就是高可用性数据库系统的目标。ScyllaDB 实现零停机的方式是通过一些机制，包括机架和数据中心感知以及多数据中心复制。

ScyllaDB 集群可以跨越分散在任何地理空间的数据中心。ScyllaDB 中的数据以最终一致的方式跨数据中心自动同步，无需用户创建任何类型的流或批处理来确保集群传达更改。

机架和数据中心意识 ScyllaDB 具有拓扑意识。它使用告密者来了解节点属于哪个机架和哪个数据中心。这些允许您将数据分布在数据中心不同机架中的节点上，或者跨公共云中的不同数据中心、可用区和区域。这样，如果一个机架甚至整个数据中心出现故障，您的数据仍然可用。

多数据中心复制跨不同数据中心的 ScyllaDB 集群可以采用 NetworkTopologyStrategy 并为每个数据中心设置不同的复制因子。例如，主数据中心的 RF 可能为 3，而单独的卫星数据中心的 RF 可能设置为 2。这使您可以确定每个站点数据的弹性。

##### **2.3.5 反熵**

ScyllaDB 设计为即使在节点临时不可用（当它最终重新加入集群时）或节点故障（当它必须更换时）的情况下也能运行。但当这些情况发生时，系统必须与熵作斗争，并使集群恢复全面运行。以下流程和功能旨在缓解这种情况：

hinted handoff：在节点临时中断（少于三个小时）的情况下，ScyllaDB 使用称为 “提示切换” 的功能来跟踪节点不可用时发生的事务。当节点恢复服务时，提示切换允许节点赶上离线时发生的情况。（你可以把它想象成一个同学，他会为你做笔记，以防你错过一两节课。）

Row-level Repair：如果您的节点可用性出现更严重的损失，ScyllaDB 有一个后台修复过程，可让您让新节点加快速度。这个过程可以使用命令行界面（称为 nodetool 修复）进行管理，也可以在 ScyllaDB Manager 中进行管理，ScyllaDB Manager 还可以从备份中恢复数据。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/j3gficicyOvasqwgS0VsKNADeQ6qsde2rxuibrwL2GXhWAiaia8gSHdwyIib2je1oOkpl7Ylvl272Ficz2tq3YSL9b0Zw/640?wx_fmt=other)

#### 2.4 ScyllaDB 中的网络通信

Client-to-Server

ScyllaDB 支持多种网络协议，作用于客户端—服务器之间的 RPC Streaming 通信。Cassandra Query Language (CQL)、Apache Thrift、HTTP/HTTPS RESTful API。

Server-to-Server

服务器—服务器使用时 RPC Streaming 进行通信。在 ScyllaDB 本身内，服务器到服务器的通信使用高效的 Seastar RPC 流，并使用暗示切换等反熵机制保持彼此同步。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/j3gficicyOvasqwgS0VsKNADeQ6qsde2rxB1XB3TSSDdYwd7v514gMofbZ1bQDYRiaBLM7Py4jqkVgG7K81EPcauQ/640?wx_fmt=other)

Secure Networking

ScyllaDB 认真对待网络威胁，并应用强大的安全方法和协议，包括身份验证、基于角色的访问控制 (RBAC)、授权和加密。用户可以对客户端和服务器节点之间以及服务器节点之间传输的数据应用加密。

#### 2.5 内存管理

在启动过程中，ScyllaDB 会检查节点的硬件，并尝试为自己申请所有可用内存（除了保留给操作系统的内存），因为内存是任何 NoSQL 数据库最关键的资源。分配的内存被划分并分配给节点中运行的每个单线程分片，每个分片固定到不同的 CPU 核心。这种方法允许 ScyllaDB 以无共享 NUMA 友好的方式有效地将内存分配给每个 CPU 的核心，并避免任何典型的阻塞操作或内存锁定，被称为无锁内存管理。

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/j3gficicyOvasqwgS0VsKNADeQ6qsde2rxnMrsibcYSfzjyF9iaKJmE3LUINtxkuialncSu7tKicObsgeZq0RNG24DiaA/640?wx_fmt=other)

**Memtable and Row-Based Cache**

ScyllaDB 中分配的内存的一个主要部分是用于内存表（Memtable），这是一种在写入路径上使用的内存结构，用于在将传入的写入和更新刷新到磁盘上的持久 SSTable 之前对它们进行排队。请注意，相同的数据会立即写入提交日志以确保持久性。

第二部分用于基于行的缓存。通常在基于 Linux 的操作系统中，数据以 4KB 块的形式从存储中获取。然而，实际上，许多数据库读取获取的数据要少得多。这会导致 Linux 缓存的读取放大和低效率。相比之下，ScyllaDB 在读取过程中完全绕过 Linux 缓存，并利用其自己的高效基于行的缓存。

ScyllaDB 中的内存分配是动态的且按需分配。没有静态块或预留空间。例如，在只读工作负载中，缓存将消耗所有内存。如果写入开始，内存将从缓存中回收并用于创建内存表。

**In-Memory Tables**

ScyllaDB Enterprise 和 ScyllaDB Cloud 允许用户利用一种更独特的数据结构类型：内存表（In-Memory）。这些是存储在 RAM 中的标准 CQL 可查询 SSTable。虽然 NVMe SSD 的 ScyllaDB 已经提供了始终如一的低毫秒级延迟，但在内存中运行表的速度要快一个数量级，p99 延迟以数百微秒（微秒）为单位测量。为了实现弹性，ScyllaDB 的内存中 NoSQL 表也会随着时间的推移持久保存到磁盘上的 SSTable。

#### 2.6 ScyllaDB 性能

ScyllaDB 有着卓越的性能表现，具体见[测试数据](https://www.scylladb.com/product/benchmarks/aws-c3-2xlarge-benchmark/)。得益于其**优秀的架构设计**外，还有上面提到的采用 C++ 编写充分利用 Linux 底层原语优势，利用现代多核、多处理器 NUMA 服务器硬件。

**“无共享” 设计**

ScyllaDB 采用分片（Shard）设计，每个分片分配给特定 CPU 及其关联的内存 (RAM) 和持久存储（例如 NVMe SSD）。分片作为独立运行的单元运行，ScyllaDB 底层基于 [Seastar](https://seastar.io/) 框架，采用高度异步、无共享设计。每个数据分片都分配有 CPU、RAM、持久存储和网络资源，并尽可能高效地使用这些资源。这大大减少了竞争以及对昂贵的处理锁的需求。在无法避免内核之间通信的情况下，Seastar 提供高度可扩展的异步无锁内核间通信。

**避免用户态内核态切换**

当在 SSTable 中找到一行时，需要通过网络将其发送到客户端。这涉及将数据从用户空间复制到内核空间。ScyllaDB 通过使用 Seastar 的网络堆栈来处理这个问题。Seastar 的网络堆栈在用户空间中运行，并利用 [DPDK](https://www.dpdk.org/) 实现更快的数据包处理。DPDK 绕过内核将数据直接复制到 NIC 缓冲区，并使其在最短的 CPU 周期得到处理。

**卓越的内存管理**

当您有顺序 I/O 并且数据以有线格式存储在磁盘中时，页面缓存非常有用。然而，在 ScyllaDB 中，有 SSTable 形式的数据，页缓存以相同的格式存储数据，小数据会占用大量内存，并且在传输时需要序列化 / 反序列化。ScyllaDB 不依赖页缓存，而是将大部分内存分配给行缓存。Row-Cache 以优化的内存格式存储数据，占用空间更少，并且不需要序列化 / 反序列化使用行缓存的另一个优点是，当页面缓存受到冲击时发生压缩时，行缓存不会被删除。

此外，ScyllaDB Enterprise 和 ScyllaDB Cloud 中提供了一个内存表（In-Memory Tables）的数据结构用来存储标准 CQL 可查询 SSTable，使得查询更快延迟更低。

**其他**

ScyllaDB 还有许多其他的底层优化，这里有不全部展开了。

#### 2.7 ScyllaDB 其他问题

ScyllaDB 并不是一点问题都没有，他还是存在一些问题，例如上面提到的反向查询性能问题（已优化），有些已经在迭代中解决，有些也问题还未解决。

##### 2.7.1 大行和热行

上面已经提到，ScyllaDB 还将最常用的行缓存在基于内存的缓存中，以避免在所谓的 “热分区” 上进行昂贵的基于磁盘的查找。但 ScyllaDB 对大行（`system.large_rows`）、大单元格（`system.large_cells`）问题目前还没有优化，不过提供观察和告警手段去监测这类问题。

##### 2.7.2 大分区和热分区

当某个分区包行大量的行时，这个分区就称为大分区。当对它进行读取和查询时，速度就可能变慢。当某个分区对访问次数特别多时，该分区就成了热分区。最为严重的时，某个分区既是大分区又是热分区时候，问题就变得格外严重。大分区的解决办法是根据自己的数据模型选择合适的分区键（单列或者多列组合的形式），使得分区更小更容易管理。为了跟踪大分区，SycallDB 提供了一个名为 system. large_partitions 的系统表。每次将大分区写入磁盘时（这意味着在将其从内存表中刷新后），都会向该表添加一个条目。可以检测随着时间的推移生成了多少大分区，以便了解数据的行为方式并根据需要改进数据分布。

针对热分区，ScyllaDB 支持二级索引（本地二级索引和全局二级索引），可以通过创建二级索引的形式提升查询的效率。即使这样可能有时候也不能满足业务的需求，这就需要在业务做一些类似合并请求的方法去给分区 “消热”。

### **3、总结**

本文介绍了 Discord 将数据迁移到 ScyllaDB 的过程，以及 SycllaDB 实现原理，使用 ScyllaDB 一些使用问题的解法。接下来有时间的话会解读下 [ScyllaDB 的源码](https://github.com/scylladb/scylladb)以及最佳实践之类。

### 4、参考资料

https://discord.com/blog/how-discord-stores-billions-of-messages

https://discord.com/blog/how-discord-stores-trillions-of-messages

https://www.scylladb.com/product/technology/

![](https://mmbiz.qpic.cn/sz_mmbiz_gif/j3gficicyOvasOHYuGEic5USJwXGgWfxiaIXz2uxcnbFsTuYvRdoias1ajkEa7Qd81r31CJCMQ8OK6YfMYjw2OXz3EA/640?wx_fmt=gif)