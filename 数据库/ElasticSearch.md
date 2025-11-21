
# 1. ES 的架构

## 1.1. 数据划分

### 1.1.1. 数据组织形式

结构化数据：也称作行数据，主要通过关系型数据库由二维表结构进行存储和管理。

非结构化数据：又可称为全文数据，不定长或无固定格式。

### 1.1.2. 使用场景

ES 在全文检索、日志分析、监控分析等场景具有广泛应用。

#### 1.1.2.1. 日志实时分析

日志处理的基本流程包含：日志采集 -> 数据清洗 -> 存储 -> 可视化分析。Elastic Stack 通过完整的日志解决方案，帮助用户完成对日志处理全链路管理。

![image.png](https://r2.129870.xyz/img/202309110133197.png)

#### 1.1.2.2. 时序分析场景

ES 提供灵活、多维度的统计分析能力，实现查看监控按照地域、业务模块等灵活的进行统计分析。另外，ES 支持列存储、高压缩比、副本数按需调整等能力，可实现较低存储成本。最后时序数据也可通过 Kibana 组件轻松实现可视化。

![image.png](https://r2.129870.xyz/img/202309110135795.png)

#### 1.1.2.3. 搜索场景

搜索服务典型场景有像京东、拼多多、蘑菇街中的商品搜索；应用商店中的应用 APP 搜索；论坛、在线文档等站内搜索。

这类场景用户关注高性能、低延迟、高可靠、搜索质量等。如单个服务最大需达到 10w+ QPS，请求平均响应时间在 20ms 以内，查询毛刺低于 100ms，高可用如搜索场景通常要求 4 个 9 的可用性，支持单机房故障容灾等。

## 1.2. 倒排索引

Elasticsearch 是以目前以 Lucene 为基础建立的开源可用全文搜索引擎，而 Lucene 能实现全文搜索主要是因为它实现了倒排索引的查询结构。

假如现有三份数据文档：

```txt
Java is the best programming language.
PHP is the best programming language.
Javascript is the best programming language.
```

通过分词器将每个文档的内容域拆分成单独的词（我们称它为词条或 Term），创建一个包含所有不重复词条的排序列表，然后列出每个词条出现在哪个文档。这个结构称为倒排索引。

![image.png](https://r2.129870.xyz/img/202309101735990.png)

其中主要有如下几个核心术语需要理解：

- 词条（Term）
    索引里面最小的存储和查询单元。
- 词典（Term Dictionary）
    字典，是词条 Term 的集合。
- 倒排表（Post list）
    倒排表记录的是某个词在哪些文档里出现过以及出现的位置以及词频等信息。
- 倒排文件（Inverted File）
    存储倒排索引的物理文件。

即倒排索引主要由两个部分组成：

- 词典
- 倒排文件

词典和倒排文件是分两部分存储的，词典在内存中而倒排文件存储在磁盘上。

分析器 (Analyzer) 由三个部分组成：

1. 分词 (Tokenization)：将文本拆分成单词。
2. 标准化 (Normalization)：如大小写转换、去除停用词。
3. 过滤 (Filtering)：如移除特殊字符。

## 1.3. 集群

ES 集群由一个或多个 Elasticsearch 节点组成，每个节点配置相同的 `cluster.name` 即可加入集群，默认值为 “elasticsearch”。

一个 Elasticsearch 服务启动实例就是一个节点（Node）。节点通过 `node.name` 来设置节点名称，如果不设置则在启动时给节点分配一个随机通用唯一标识符作为名称。

### 1.3.1. 集群的发现

[Zen Discovery](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery-zen.html) 是 ES 的内置默认发现模块（发现集群中的节点以及选举 Master 节点）。它提供单播和基于文件的发现，并且可以扩展为通过插件支持云环境和其他形式的发现。Zen Discovery 可与其他模块集成，例如，节点之间的所有通信都使用 Transport 模块完成。节点使用发现机制通过 Ping 的方式查找其他节点。

![](https://r2.129870.xyz/img/202309110126695.png)

ES 的发现流程如下：

1.  如果 `discovery.zen.ping.unicast.hosts` 有配置，则尝试 ping 这些节点
2. 如果没有配置，则 ping 本地的默认节点

### 1.3.2. 节点类型

ES 节点分为两种类型：

- 数据节点：负责数据的存储和相关的操作
- 主节点：负责维护集群各种元信息，例如创建索引、删除索引、维护集群节点状态，维护分片信息等

每个节点既可以是候选主节点也可以是数据节点，候选主节点可以被选举为主节点（Master 节点），**集群中只有候选主节点才有选举权和被选举权**，其他节点不参与选举的工作。

![image.png](https://r2.129870.xyz/img/202311100029080.png)

所以如果某个节点既是数据节点又是主节点，那么主节点产生影响时可能会对从而对整个集群的状态产生影响。因此为了提高集群的健康性，应该对 Elasticsearch 集群中的节点做好角色上的划分和隔离。

主节点和其他节点之间通过 Ping 的方式互检查，当节点故障后，会被主节点移出集群，并自动在其他节点上恢复故障节点上的分片。主分片故障时会提升其中一个副本分片为主分片。其他节点也会探活主节点，当主节点故障后，会触发内置的类 Raft 协议选主，并通过设置最少候选主节点数，避免集群脑裂。

![image.png](https://r2.129870.xyz/img/202309110128550.png)

虽然对节点做了角色区分，但是**用户的请求可以发往任何一个节点，并由该节点负责分发请求、收集结果等操作，而不需要主节点转发**。这种节点可称之为协调节点，集群中的任何节点都可以充当协调节点的角色。

### 1.3.3. 节点选举

master 节点的选举过程如下：
1. 选举的时机
    由 master-eligible 节点发起，发起条件为：
    1.  该 master-eligible 节点的当前状态不是 master。
    2.  该 master-eligible 节点通过 ZenDiscovery 模块的 ping 操作询问其已知的集群其他节点，没有任何节点连接到 master。
    3.  包括本节点在内，当前已有超过 minimum_master_nodes 个节点没有连接到 master。
    
    即当一个节点发现包括自己在内的多数派的 master-eligible 节点认为集群没有 master 时，就可以发起 master 选举。
2. 选举的对象
    1. 节点的状态越新，优先级越高
        clusterStateVersion 越大数据状态越新。
    2. 当 clusterStateVersion 相同时，节点的 Id 越小，优先级越高。
        根据 ID 是为了保持选举的稳定性。
2. 选举的流程
    - 假设 Node_A 选 Node_B 当 Master  
        Node_A 会向 Node_B 发送 join 请求，那么此时会有以下几种情况：
        1. 如果 Node_B 已经成为 Master  
            Node_B 就会把 Node_A 加入到集群中，然后发布最新的 cluster_state，最新的 cluster_state 就会包含 Node_A 的信息。
            
            这就相当于一次正常情况的新节点加入。对于 Node_A，等新的 cluster_state 发布到 Node_A 的时候，Node_A 也就完成 join 了。
        2. 如果 Node_B 在竞选 Master  
            此时 Node_B 会把这次 join 当作一张选票。对于这种情况，Node_A 会等待一段时间，看 Node_B 是否能成为真正的 Master，直到超时或者有别的 Master 选成功。
        3. 如果 Node_B 认为自己不是 Master (现在不是，将来也选不上)，那么 Node_B 会拒绝这次 join。对于这种情况，Node_A 会开启下一轮选举。
    - 假设 Node_A 选自己当 Master  
        此时 NodeA 会等别的 node 来 join，即等待别的 node 的选票，当收集到超过半数的选票时，认为自己成为 master，然后变更 cluster_state 中的 master node 为自己，并向集群发布这一消息。

选举的限制条件是节点数超过 `discovery.zen.minimum_master_nodes`，如果**节点数达不到最小值的限制，则循环上述过程，直到节点数足够可以开始选举**。ES 将以上服务发现以及选主的流程叫做 Zen Discovery 。

### 1.3.4. 脑裂现象

如果由于网络或其他原因导致集群中选举出多个 Master 节点，使得数据更新时出现不一致，这种现象称之为脑裂。

> [!tip] 脑裂之多次投票问题的解决
> 在 raft 算法中引入了选举周期 (term) 的概念，保证了每个选举周期中每个成员只能投一票，如果需要再投就会进入下一个选举周期，term+1。
> 
> 假如最后出现两个节点都认为自己是 master，那么 term 较大的节点获胜。由此保证了 term 小的 master 不可能 commit 任何状态变更(commit 需要多数派节点先持久化日志成功，由于有 term 检测，不可能达到多数派持久化条件)。这就保证了集群的状态变更总是一致的。

“脑裂”问题可能有以下几个原因造成：

- 网络问题
    集群间的网络延迟等问题导致节点失联。
- 节点负载
    既为主节点也为数据节点，导致节点负载高。
- 内存回收
    JVM 的大规模内存回收造成 ES 进程失去响应。

为了避免脑裂现象的发生，可以从原因着手通过以下几个方面来做出优化措施：
- 适当调大响应时间，减少误判
    调整节点健康检查状态的响应时间，避免错误的杀死节点。
- 选举触发
    配置合理的选举节点数量，例如一半以上的节点。
- 角色分离
    即候选主节点和数据节点进行角色分离，这样可以减轻主节点的负担，防止主节点的假死状态发生，减少对主节点“已死”的误判。

### 1.3.5. 节点监控

节点监控主要用于检测 master 节点及其他节点的状态，有两类错误检测，一类是 Master 定期检测集群内其他的 Node，另一类是集群内其他的 Node 定期检测当前集群的 Master。

> [!quote] ES 的错误检测
> 
> There are two fault detection processes running. The first is by the master, to ping all the other nodes in the cluster and verify that they are alive. And on the other end, each node pings to master to verify if its still alive or an election process needs to be initiated.

检测的方式如下：
1. Master 节点检测其他节点
    如果 Master 检测到某个 Node 无法连接，会执行 removeNode 的操作将节点从集群中移除。
2. 其他节点检测 Master 节点
    若发现 Master 无法连接，则发起 rejoin，重新加入集群 (如果达到选举条件则触发新 master 选举)。
3. Master 节点检查自身
    若 Master 发现自己已经不满足多数派条件 (>=minimumMasterNodes) 了，需要主动退出 master 状态 (退出 master 状态并执行 rejoin) 以避免脑裂的发生。master 需要 rejoin 的触发条件类似如下：
    
    1. 移除其他节点后判断 minimumMasterNodes 条件
        在移除其他节点后判断当前集群节点数量是否可构成健康的集群。
    2. 发布集群状态时判断响应的节点数量
        例如在 publish 新的 cluster_state 时，分为 send 阶段和 commit 阶段，send 阶段要求多数派必须成功，然后再进行 commit。如果在 send 阶段没有实现多数派返回成功，那么可能是有了新的 master 或者是无法连接到多数派个节点等，则 master 需要执行 rejoin。

### 1.3.6. 集群扩缩容

假设一个 ES 集群存储或者计算资源不够了，我们需要进行扩容。

#### 1.3.6.1. 数据节点

对于数据节点的扩容，只要配置数据节点信息将其加入集群即可。启动节点后，节点会自动加入到集群中，集群会自动进行 rebalance，或者通过 [reroute api](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-reroute.html) 进行手动操作。

对于数据节点的缩容，为了保证数据安全需要先把这个 Node 上的 Shards 迁移到其他节点上。首先设置 allocation [规则](https://www.elastic.co/guide/en/elasticsearch/reference/current/allocation-filtering.html)，禁止分配 Shard 到要缩容的机器上，然后让集群进行 rebalance。

> [!example] 示例
> ```bash
> PUT _cluster/settings
> {
>   "transient" : {
>     "cluster.routing.allocation.exclude._ip" : "10.0.0.1"
>   }
> }
> ```

#### 1.3.6.2. 主节点

假如想扩容一个 MasterNode (master-eligible node)，为了避免脑裂需要先提升 quorum 数量，然后再进行集群的扩容操作。缩容与之相似。

### 1.3.7. 集群元数据管理

#### 1.3.7.1. 元数据管理

Master 节点通过发布 `ClusterState` 来通知其他节点最新的元数据信息，当节点收到新的 ClusterState 后，会把新的 ClusterState 发给相关的各个模块，各个模块根据新的 ClusterState 判断需要触发的操作。即通过 Meta 数据来驱动各个模块工作。

为了保证元数据的一致性，需要让所有节点都执行最新的元数据变更操作。当选举出新的 Master 后，**需要保证所有节点都要按照最新的 Meta 执行操作**。

#### 1.3.7.2. 元数据的恢复

当 ES 集群重启后首先进行 Master 选举，选举完成才会进行故障恢复。恢复时 master 节点会向集群中的 MasterNode 和 DataNode 请求其机器上的 MetaData 然后选择其中 version 最大的版本。对于每个 Index 的 Meta，也选择其中最大的版本。然后将集群的 Meta 和每个 Index 的 Meta 再组合起来，构成当前的最新 Meta。

#### 1.3.7.3. 元数据的更新

1. 并发更新的控制
    首先，master 进程内不同线程更改 ClusterState 时要保证是原子的，采取的方式是**串形化**。每次需要更新 ClusterState 时提交一个 Task 给 MasterService，MasterService 中只使用一个线程来串行处理这些 Task。
2. 两阶段提交
    为了保证元数据信息在所有节点都正常完成，ES 采取了[[分布式大纲#2.4.3. XA 实现之 2PC 协议|两阶段提交]]的方式。
    
    ![image.png](https://r2.129870.xyz/img/202311112240018.png)
    
    采取两阶段提交可以解决一些问题，但并不是完美的：
    1. 第一阶段数据未持久化
        第一阶段 master 节点 send 新的 ClusterState，接收到的节点只是把新的 ClusterState 放入内存一个队列中，就会返回 ack。这个过程没有持久化，所以当 master 接收到超过半数的 ack 后，也不能认为这些节点上都有新的 ClusterState 了。
    2. 节点失效
        如果 master 在 commit 阶段，只 commit 了少数几个节点就出现了网络分区，将 master 与这几个少数节点分在了一起，其他节点可以互相访问。此时其他节点构成多数派，会选举出新的 master，由于这部分节点中没有任何节点 commit 了新的 ClusterState，所以新的 master 仍会使用更新前的 ClusterState，造成 Meta 不一致。

### 1.3.8. 节点数据的恢复

ES 中每个写操作都会分配两个值，`Term` 和 `SequenceNumber`，该值由 Primary 分配，每次变更操作递增。在向 Replica 发送同步请求时，会带上这两个值。

`LocalCheckpoint` 代表本 Shard 中所有小于该值的请求都已经处理完毕。`GlobalCheckpoint` 代表所有小于该值的请求在所有的 Replica 上都处理完毕。`GlobalCheckpoint` 会由 Primary 进行维护，每个 Replica 会向 Primary 汇报自己的 `LocalCheckpoint`，Primary 根据这些信息来提升 `GlobalCheckpoint`。

`GlobalCheckpoint` 是一个全局的安全位置，代表其前面的请求都被所有 Replica 正确处理了，可以应用在节点故障恢复后的数据回补。另一方面，`GlobalCheckpoint` 也可以用于 `Translog` 的清理，标记更旧的日志都无需保留了。

当一个 Replica 故障时，ES 会将其移除，对于故障节点的处理，分为两种情况：
1. 故障超过一定时间
    ES 会分配一个新的 Replica 到新的 Node 上，此时需要全量同步数据。
2. 故障节点自动恢复
    节点恢复后回补故障之后的数据，追平后加回来即可。
    
    实现快速故障恢复的条件有两个：
    1. 能够保存故障期间所有的操作以及其顺序
    2. 能够知道从哪个点开始同步数据
    
    第一个条件可以通过保存一定时间的 Translog 实现，第二个条件可以通过 Checkpoint 实现。

## 1.4. 分片

当索引上的数据量太大时通过水平拆分的方式将一个索引上的数据拆分出来分配到不同的数据块上，拆分出来的数据库块称之为一个分片。

在一个多分片的索引中写入数据时，通过路由来确定具体写入哪一个分片中，所以在创建索引的时候需要指定分片的数量，并且分片的数量一旦确定就不能修改。

## 1.5. 副本

副本是对分片的 Copy，每个主分片都有一个或多个副本分片，其各自位于不同的节点。当主分片异常时，副本可以提供数据的查询等操作。对文档的新建、索引和删除请求都是写操作，必须在主分片上面完成之后才能被复制到相关的副本分片。

副本分片的写入过程是并发写的，为了解决并发写的过程中数据冲突的问题通过乐观锁的方式控制。每个文档都有一个 `_version`（版本）号，当文档被修改时版本号递增。一旦所有的副本分片都报告写成功才会向协调节点报告成功，协调节点向客户端报告成功。

![image.png](https://r2.129870.xyz/img/202309101908800.png)

# 2. 数据存储原理

## 2.1. Es 的存储模型

![image.png](https://r2.129870.xyz/img/202311030114347.png)

## 2.2. 索引写入原理

![image.png](https://r2.129870.xyz/img/202309101911565.png)

索引数据只能写在主分片上，然后同步到副本分片，写入分片的规则确定如下：

$$
shard=hash(routing) \% number\_of\_primary\_shards
$$

Routing 是一个可变值，默认是文档的 `_id`，也可以设置成一个自定义的值。**在创建索引的时候就需要确定主分片的数量并且永远无法改变这个数量**，因为如果数量变化了，那么所有之前路由的值都会无效。

ES 集群中每个节点都有处理读写请求的能力。在一个写请求被发送到某个节点后，协调节点会根据路由公式计算出需要写到哪个分片上，再将请求转发到该分片的主分片节点上。

![image.png](https://r2.129870.xyz/img/202309101915606.png)

每次写入的时候，写入请求会先通过 routing 的 Hash 值选择出 Shard，最后从集群的 Meta 中找出该 Shard 的 Primary 节点。之后请求会发送给 Primary Shard，在 Primary Shard 上执行成功后，再从 Primary Shard 上将请求同时发送给多个 Replica Shard，请求在多个 Replica Shard 上执行成功并返回给 Primary Shard 后，写入请求执行成功，返回结果给客户端。

这种模式下，写入操作的延时：$latency = Latency (Primary Write) + Max (Replicas Write)$。只要有副本在，写入延时最小也是两次单 Shard 的写入时延总和，写入效率会较低，但是这样的好处也很明显，避免写入后，单机或磁盘故障导致数据丢失，在数据重要性和性能方面，一般都是优先选择数据，除非一些允许丢数据的特殊场景。

## 2.3. 数据存储原理

### 2.3.1. 分段写

索引文档以[[数据密集型系统设计1：引入#1.2.2. SSTables (SortStringTables)|段的形式]]存储在磁盘上，何为段？索引文件被拆分为多个子文件，则每个子文件叫作段，每一个段本身都是一个倒排索引，并且段具有不变性，一旦索引的数据被写入硬盘，就不可再修改。在底层采用了分段的存储模式，使它在读写时几乎完全避免了锁的出现，大大提升了读写性能 （在并发更新文档的场景下，ES 是采用乐观锁版本号的方式来实现并发控制）。

段被写入到磁盘后会生成一个提交点，提交点是一个用来记录所有提交后段信息的文件。一个段一旦拥有了提交点，就说明这个段只有读的权限，失去了写的权限。相反，当段在内存中时，就只有写的权限，而不具备读数据的权限，意味着不能被检索。

索引文件分段存储并且不可修改，那么新增、更新和删除如何处理呢？

- 新增
    对当前文档新增一个段即可。
- 删除
    由于不可修改，所以对于删除操作是通过新增一个 `.del` 文件，文件中会列出这些被删除文档的段信息。这个被标记删除的文档仍然可以被查询匹配到，但它会在最终结果被返回前从结果集中移除。
- 更新
    更新相当于是删除和新增这两个动作组成。

段被设定为不可修改具有一定的优势也有一定的缺点，优势主要表现在：

- 不需要锁：如果你从来不更新索引，你就不需要担心多进程同时修改数据的问题。
- 一旦索引被读入内核的文件系统缓存，便会留在哪里，由于其不变性。只要文件系统缓存中还有足够的空间，那么大部分读请求会直接请求内存，而不会命中磁盘。这提供了很大的性能提升。
- 其它缓存 (例如 Filter 缓存)，在索引的生命周期内始终有效。它们不需要在每次数据改变时被重建，因为数据不会变化。
- 写入单个大的倒排索引允许数据被压缩，减少磁盘 I/O 和需要被缓存到内存的索引的使用量。

段的不变性的缺点如下：

- 当对旧数据进行删除时，旧数据不会马上被删除，而是在 `.del` 文件中被标记为删除。而旧数据只能等到段更新时才能被移除，这样会造成大量的空间浪费。
- 若有一条数据频繁的更新，每次更新都是新增新的标记旧的，则会有大量的空间浪费。
- 每次新增数据时都需要新增一个段来存储数据。当段的数量太多时，对服务器的资源例如文件句柄的消耗会非常大。
- 在查询的结果中包含所有的结果集，需要排除被标记删除的旧数据，这增加了查询的负担。

#### 2.3.1.1. Create 流程

![image.png](https://r2.129870.xyz/img/202311112029405.png)

每当有新 Document 要写入时，会进行以下操作：

1. 同时写入 Transaction Log 和 Index Buffer
    新文档首先会被**同时追加到 Transaction Log（WAL）和 Index Buffer**中。Transaction Log 用于崩溃恢复，确保数据持久性；Index Buffer 作为内存缓冲区，临时存储待处理的文档。
2. Refresh 操作生成内存中的 Segment
    每隔一定时间（默认 1 秒），ES 执行 `refresh` 操作：将 Index Buffer 中的文档转换为内存中的**新 Segment**。此时文档**变为可搜索**（近实时搜索，NRT）。Index Buffer 随后被清空。此外 Index Buffer 被写满时也会触发，默认大小是 JVM 内存的 10%
3. Flush 操作将 Segment 落盘并清理 Transaction Log
    周期性 `flush` 操作（默认 30 分钟或 Transaction Log 大小触发）将内存中的 Segment **写入磁盘**（持久化）。同时，**已落盘的日志条目会被删除**（Transaction Log 是追加的，仅清理已提交部分，而非完全清空）。

这里有几个关键点：

1. 写 Lucene 内存后，并不是可被搜索的，内存中的对象转成完整的 Segment 后方可被搜索到
2. 当通过 GetById 时可以直接从 TransLog 中查询，这时候就成了 RT（Real Time）实时系统
3. 定时刷新 segment 到磁盘，此后会清空掉旧的 TransLog

#### 2.3.1.2. Update 流程

![image.png](https://r2.129870.xyz/img/202311112036650.png)

Lucene 中不支持部分字段的 Update，所以需要在 Elasticsearch 中实现该功能，具体流程如下：

1. 读取历史数据
    收到 Update 请求后，读取同 id 的完整 Doc，记录版本号为 V1。
2. 合并新旧数据
    将版本 V1 的全量 Doc 和请求中的部分字段 Doc 合并为一个完整的 Doc。
3. 加锁
    锁是 Refresh Lock，禁止 refresh 该条数据。
4. 再次检查版本
    再次从 versionMap 中读取该 id 的最大版本号 V2，如果 versionMap 中没有则从 Segment 或者 TransLog 中读取，这里基本都会从 versionMap 中获取到。
    
    检查版本是否冲突： `V1 == V2`，如果冲突，则回退到开始的“Update doc” 阶段，重新执行。如果不冲突，则执行最新的 Add 请求。
5. 更新数据
    在 Index Doc 阶段，首先将 Version + 1 得到 V3，再将 Doc 加入到 Lucene 中去，Lucene 中会先删同 id 下的已存在 doc id，然后再增加新 Doc。写入 Lucene 成功后，将当前 V3 更新到 versionMap 中。
6. 释放锁
    此时部分更新的流程就结束了。

#### 2.3.1.3. 整体写入流程

![image.png](https://r2.129870.xyz/img/202311112041329.png)

- 红色：Client Node。
- 绿色：Primary Node。
- 蓝色：Replica Node。

以下解释上图中写入的核心过程：
1. Client Node 执行过程
    1. Auto Create Index
        是否需要手动创建不存在的索引。
    2. Set Routing
        根据路由条件计算所属分片
    3. Construct BulkShardRequest
        由于 Bulk Request 中会包括多个请求，这一步将位于同一个分片的请求划分到同一组。
    4. Send Request To Primary
        将上一步每一个 BulkShardRequest 请求发送给相应 Shard 的 Primary Node。
2. Primary Node 执行过程
    以 Update 为例：
    
    1. Translate Update To Index or Delete
        将 Update 请求转换为 Index 或者 Delete 请求。首先，会通过 GetRequest 查询到已经存在的同 id Doc（如果有）的完整字段和值（依赖 source 字段），然后和请求中的 Doc 合并。同时，这里会获取到读到的 Doc 版本号，记做 V1。
    2. Parse Doc
        解析 Doc 中各个字段。
    3. Update Mapping
        找出 Mapping 中未包含的新 Field，然后判断是否运行自动更新 Mapping，如果允许，则更新 Mapping。
    4. Get Sequence Id and Version
        SequenceID 在 Shard 级别每次递增 1，用于维护 LocalCheckpoint 信息。Version 根据当前 Doc 的最大 Version 递增 1。
    5. Add Doc To Lucene
        这一步开始的时候对数据进行加锁，并判断当前数据的版本和之前获取到的是否一致。若不一致则退回起点重新操作，如果版本一致则进行数据更新操作。
        
        为了保证 Delete-Then-Add 的原子性，在开始 Delete 之前会加一个 Refresh Lock，禁止被 Refresh，只有等 Add 完后释放了 Refresh Lock 后才能被 Refresh，这样就保证了 Delete-Then-Add 的原子性。
        
        > [!info] Lucene 字段写入过程
        > </br> Lucene 的 UpdateDocument 接口中就只是处理多个 Field，会遍历每个 Field 逐个处理，处理顺序是 invert index，store field，doc values，point dimension。
        
    6. Write Translog
        写完 Lucene 的 Segment 后，会以 key-value 的形式写 TransLog，Key 是_id，Value 是 Doc 内容。当查询的时候，如果请求是 GetDocByID，则可以直接根据_id 从 TransLog 中读取到，满足 NoSQL 场景下的实时性要去。
        
        > [!attention] 注意
        > </br>这里**只是写入到内存的 TransLog**，是否 Sync 到磁盘的逻辑还在后面。这一步的最后，会标记当前 SequenceID 已经成功执行，接着会更新当前 Shard 的 LocalCheckPoint。
    7. Renew Bulk Request
        前面已经将 UpdateRequest 翻译成了 Index 或 Delete 请求，则后续所有 Replica 中只需要执行 Index 或 Delete 请求即可，不需要再执行 Update 逻辑。
    8. Flush Translog
        根据 TransLog 的策略，选择不同的执行方式，要么是立即 Flush 到磁盘，要么是等到以后再 Flush。Flush 的频率越高，可靠性越高，对写入性能影响越大。
    9. Send Requests To Replicas
        将更新操作发送给所有 Replica 并等待结果，如果某个 Replica 失败了，则 Primary 会给 Master 发送一个 Remove Shard 请求，要求 Master 将该 Replica Shard 从可用节点中移除。
        
        > [!info] 副本写入的限制
        > </br> ES 中有一个参数，叫做 wait_for_active_shards，这个参数是 Index 的一个 setting，也可以在请求中带上这个参数。这个参数的含义是，<b>在每次写入前，该 shard 至少具有的 active 副本数</b>。假设我们有一个 Index，其每个 Shard 有 3 个 Replica，加上 Primary 则总共有 4 个副本。如果配置 wait_for_active_shards 为 3，那么允许最多有一个 Replica 挂掉，如果有两个 Replica 挂掉，则 Active 的副本数不足 3，此时不允许写入。
        > 
        > 这个参数默认是 1，即只要 Primary 在就可以写入，起不到什么作用。如果配置大于 1，可以起到一种保护的作用，保证写入的数据具有更高的可靠性。但是这个参数只在写入前检查，并不保证数据一定在至少这些个副本上写入成功，所以并不是严格保证了最少写入了多少个副本。
        
        > [!attention]  副本写入失败的后果
        > </br> 如果一个 Replica 写失败了，Primary 会将这个信息报告给 Master，然后 Master 会在 Meta 中更新这个 Index 的 InSyncAllocations 配置，将这个 Replica 从中移除，移除后它就不再承担读请求。在 Meta 更新到各个 Node 之前，用户可能还会读到这个 Replica 的数据，但是更新了 Meta 之后就不会了。所以这个方案并不是非常的严格，考虑到 ES 本身就是一个近实时系统，数据写入后需要 refresh 才可见，所以一般情况下，在短期内读到旧数据应该也是可接受的。
        
    10. Receive Response From Replicas
        Replica 中请求都处理完后，会更新 Primary Node 的 LocalCheckPoint。
3. Replica Node 执行过程
    Replica Node 执行过程和 Primary Node 执行过程基本一致，只不过少了同步 Replica Node 与转化 Update 请求的过程。

#### 2.3.1.4. 写入机制分析

对于 Elasticsearch 的写入流程及其各个流程的工作机制，我们在此分析它是否符合分布式系统中的一些特性：

1. 可靠性
    Elasticsearch 中通过 Replica 和 TransLog 两套机制保证数据的可靠性。
2. 一致性
    Lucene 中的 Refresh 锁只保证 Update 接口里面 Delete 和 Add 中间不会 Flush，但是 **Add 完成后仍然有可能立即发生 Flush，导致 Segment 可读，这样就没法保证 Primary 和所有其他 Replica 可以同一时间 Flush，就会出现查询不一致的情况**，这里只能实现最终一致性。
3. 原子性
    Add 和 Delete 都是直接调用 Lucene 的接口，是原子的。当部分更新时，使用 Version 和锁保证更新是原子的。
4. 隔离性
    仍然采用 Version 和局部锁来保证更新的是特定版本的数据。
5. 实时性
    使用定期 Refresh Segment 到内存，并且 Reopen Segment 方式保证搜索可以在较短时间（比如 1 秒）内被搜索到。通过将未刷新到磁盘数据记入 TransLog，保证对未提交数据可以通过 ID 实时访问到。
6. 性能
    性能是一个系统性工程，所有环节都要考虑对性能的影响，在 Elasticsearch 中，在很多地方的设计都考虑到了性能：
    
    1. 不需要所有 Replica 都返回后才能返回给用户，只需要返回特定数目的即可；
    2. 生成的 Segment 先在内存中提供服务，等一段时间后才刷新到磁盘，Segment 在内存这段时间的可靠性由 TransLog 保证；
    3. TransLog 可以配置为周期性的 Flush，但这个会给可靠性带来伤害；
    4. 每个线程持有一个 Segment，多线程时相互不影响，相互独立，性能更好；
    5. 系统的写入流程对版本依赖较重，读取频率较高，因此采用了 versionMap，减少热点数据的多次磁盘 IO 开销。

#### 2.3.1.5. 查询流程

##### 2.3.1.5.1. 节点与分区的选择

查询时只需要查询 Primary 和 Replica 中的任何一个节点即可。

![image.png](https://r2.129870.xyz/img/202311130052812.png)

当查询的时候，从三个节点中根据 Request 中的 preference 参数选择一个节点查询。preference 可以设置 `_local`，`_primary`，`_replica` 以及其他选项。

由于数据分布在多个分片上，所以查询请求会分发给所有 Shard，每个 Shard 中都是一个独立的查询引擎，之后会基于所有分片的结果聚合返回最终数据。

![image.png](https://r2.129870.xyz/img/202311130053222.png)

> [!warning]  请求膨胀问题
>
>这里有一个问题就是请求膨胀，用户的一个搜索请求在 Elasticsearch 内部会变成 Shard 个请求。一种优化方式是虽然会产生 Shard 个数量的请求，但是这个 Shard 个数不一定要是当前 Index 中的 Shard 个数，只要是当前查询相关的 Shard 即可，这个需要基于业务和请求内容优化，通过这种方式可以优化请求膨胀数。

##### 2.3.1.5.2. 节点内数据的获取

Elasticsearch 中的查询主要分为两类：Get 请求：通过 ID 查询特定 Doc；Search 请求：通过 Query 查询匹配 Doc。

![image.png](https://r2.129870.xyz/img/202311130057857.png)

对于 Search 类请求，查询的时候是一起查询内存（指刚 Refresh Segment，但是还没持久化到磁盘的新 Segment）和磁盘上的 Segment，最后将结果合并后返回。这种查询是近实时（Near Real Time）的，主要是由于内存中的 Index 数据需要一段时间后才会刷新为 Segment。

对于 Get 类请求，查询的时候是先查询内存中的 TransLog，如果找到就立即返回，如果没找到再查询磁盘上的 TransLog，如果还没有则再去查询磁盘上的 Segment。这种查询是实时（Real Time）的。这种查询顺序可以保证查询到的 Doc 是最新版本的 Doc，这个功能也是为了保证 NoSQL 场景下的实时性要求。

##### 2.3.1.5.3. 数据查询阶段

![image.png](https://r2.129870.xyz/img/202311130059187.png)

所有的搜索系统一般都是两阶段查询，第一阶段查询到匹配的 DocID，第二阶段再查询 DocID 对应的完整文档，这种在 Elasticsearch 中称为 `query_then_fetch`，还有一种是一阶段查询的时候就返回完整 Doc，在 Elasticsearch 中称作 `query_and_fetch`，一般第二种适用于只需要查询一个 Shard 的请求。

> [!info] query_then_fetch 与 query_and_fetch
> 
> 在 query_then_fetch 模式下，每个分片返回的是部分结果，这些部分结果包含了匹配查询条件的文档的相关信息（例如文档的 ID、评分等），但不包含完整的文档数据。然后，协调节点会收集所有分片返回的部分结果，并进行合并、排序、去重等操作，以生成最终的搜索结果，最终的搜索结果会包含完整的文档数据。
> 
> 对于大型索引或查询结果集较大的情况，query_then_fetch 模式可以减少每个分片返回的数据量，减轻网络传输和内存消耗。而 query_and_fetch 模式则立即从每个分片获取完整的文档数据，可能导致网络开销和内存消耗较高。

除了一阶段，两阶段外，还有一种三阶段查询的情况。搜索里面有一种算分逻辑是根据 TF（Term Frequency）和 DF（Document Frequency）计算基础分，但是 Elasticsearch 中查询的时候，是在每个 Shard 中独立查询的，每个 Shard 中的 TF 和 DF 也是独立的，虽然在写入的时候通过 `_routing` 保证 Doc 分布均匀，但是没法保证 TF 和 DF 均匀，那么就有会导致局部的 TF 和 DF 不准的情况出现，这个时候基于 TF、DF 的算分就不准。

为了解决这个问题，Elasticsearch 中引入了 DFS 查询，比如 `DFS_query_then_fetch`，会先收集所有 Shard 中的 TF 和 DF 值，然后将这些值带入请求中，再次执行 `query_then_fetch`，这样算分的时候 TF 和 DF 就是准确的，类似的有 `DFS_query_and_fetch`。这种查询的优势是算分更加精准，但是效率会变差。另一种选择是用 BM25 代替 TF/DF 模型。

### 2.3.2. 延迟写策略

为了提升写的性能，ES 并没有每新增一条数据就增加一个段到磁盘上，而是采用延迟写的策略。每当有新增的数据时，就将其先写入到内存中，在内存和磁盘之间是文件系统缓存。

在 Elasticsearch 中，写入和打开一个新段的轻量的过程叫做 Refresh （即内存刷新到文件缓存系统）。当达到默认的时间（1 秒钟）或者内存的数据达到一定量时，会触发一次刷新（Refresh），将内存中的数据生成到一个新的段上并缓存到文件缓存系统上，稍后再被刷新到磁盘中并生成提交点。这里的内存使用的是 ES 的 JVM 内存，而文件缓存系统使用的是操作系统的内存。

新的数据会继续的被写入内存，但内存中的数据并不是以段的形式存储的，因此不能提供检索功能。这就是为什么我们说 Elasticsearch 是近实时搜索，因为文档的变化并不是立即对搜索可见，但会在一秒之内变为可见。由内存刷新到文件缓存系统的时候会生成新的段，并将段打开以供搜索使用，而不需要等到被刷新到磁盘。

我们也可以手动触发 Refresh，`POST /_refresh` 刷新所有索引，`POST /nba/_refresh` 刷新指定的索引。

> [!tip] 刷新时机
> 
> 尽管刷新是比提交轻量很多的操作，它还是会有性能开销。当写测试的时候，手动刷新很有用，但是不要在生产环境下每次索引一个文档都去手动刷新。而且并不是所有的情况都需要每秒刷新。
> 
> 可能你正在使用 Elasticsearch 索引大量的日志文件，你可能想优化索引速度而不是近实时搜索。这时可以在创建索引时在 Settings 中通过调大 `refresh_interval = "30s"` 的值，降低每个索引的刷新频率，设值时需要注意后面带上时间单位，否则默认是毫秒。当 `refresh_interval=-1` 时表示关闭索引的自动刷新。

### 2.3.3. 段合并

由于自动刷新流程每秒会创建一个新的段，这样会导致短时间内的段数量暴增。而段数目太多会带来较大的麻烦。每一个段都会消耗文件句柄、内存和 CPU 运行周期。更重要的是，每个搜索请求都必须轮流检查每个段然后合并查询结果，所以段越多，搜索也就越慢。

Elasticsearch 通过在后台定期进行[[数据密集型系统设计1：引入#1.2.2.2. SSTables 写入与维护|段合并]]来解决这个问题。小的段被合并到大的段，然后这些大的段再被合并到更大的段。

段合并的时候会将那些旧的已删除文档从文件系统中清除。被删除的文档不会被拷贝到新的大段中。合并的过程中不会中断索引和搜索。由于自动刷新流程每秒会创建一个新的段，这样会导致短时间内的段数量暴增。而段数目太多会带来较大的麻烦。

段合并在进行索引和搜索时会自动进行，合并进程选择一小部分大小相似的段，并且在后台将它们合并到更大的段中，这些段既可以是未提交的也可以是已提交的。合并结束后老的段会被删除，新的段被 Flush 到磁盘，同时写入一个包含新段且排除旧的和较小的段的新提交点，新的段被打开可以用来搜索。

段合并的计算量庞大，而且还要吃掉大量磁盘 I/O，段合并会拖累写入速率，如果任其发展会影响搜索性能。Elasticsearch 在默认情况下会对合并流程进行资源限制，所以搜索仍然有足够的资源很好地执行。

## 2.4. 主要查询类型

### 2.4.1. 单词级别查询

- **Term Query (精确查询)**：将输入字符串视为一个完整的单词进行精确查找。
- **Fuzzy Query (模糊查询)**：带编辑距离的 `term` 查询。ES 会根据编辑距离生成候选词，再进行 `term` 查询。

### 2.4.2. 全文级别查询

- **match 查询**：一种自适应查询。
    - 会根据字段是否被 `analyzed` 来决定是分词匹配还是完全匹配。
    - 可以通过 `fuzziness` 参数结合模糊查询。
- **match_phrase 查询**：短语查询。
    在 `match` 的基础上，要求分词后的词项必须顺序一致且紧邻（可以通过 `slop` 参数放宽邻近度要求）。

### 2.4.3. Bool 组合查询

用于实现复杂的组合查询逻辑，包含四种从句：

- **must**：与 (AND)，必须匹配，且计算相关度分数。
- **should**：或 (OR)，满足其中一个或多个条件。
- **must_not**：非 (NOT)，必须不匹配，不计算分数。
- **filter**：过滤 (AND)，必须匹配，但不计算相关度分数。性能比 `must` 好，适合用于精确值的过滤场景（如状态、标签、年份等）。

# 3. ES 的性能优化

## 3.1. 集群索引评估准则

索引配置的评估可根据下面几点准则进行评估：

- 单个分片大小控制在 30-50GB。
- 集群总分片数量控制在 3w 以内。
- 1GB 的内存空间支持 20-30 个分片为佳。
- 一个节点建议不超过 1000 个分片。
- 索引分片数量建议和节点数量保持一致。
- 集群规模较大时建议设置专用主节点。
- 专用主节点配置建议在 8C16G 以上。
- 如果是时序数据，建议结合冷热分离 + ILM 索引生命周期管理。

## 3.2. 写入性能优化

按不同阶段进行考虑：
1. 索引定位阶段
    1. 提前创建索引，使用固定的索引 mapping
        创建索引及新加字段都是更新元数据操作，需要 master 节点将新版本的元数据同步到所有节点。
    
        因此在集群规模比较大，写入 qps 较高的场景下，特别容易出现 master 更新元数据超时的问题，这可导致 master 节点中有大量的 pending_tasks 任务堆积，从而造成集群不可用，甚至出现集群无主的情况。
    1. 写入数据不指定 doc_id，让 ES 自动生成
        如果自定义 doc_id 的话，则 ES 在写入过程中会多一步判断的过程，即先 Get 下该 doc_id 是否已经存在。如果存在的话则执行 Update 操作，不存在则创建新的 doc。
    
        因此如果我们对索引 doc_id 没有特别要求，则建议让 ES 自动生成 doc_id，这样可提升一定的写入性能。
2. 数据发送阶段
    1. 使用 Bulk 批量插入数据时，控制单词 bulk 数量在 10M 左右
        通常我们建议一次 Bulk 的数据量控制在 10M 以下，一次 Bulk 的 doc 数在 10000 上下浮动。
    2. 使用自定义 routing 功能，尽量将请求转发到较少的分片
        协调节点是异步将数据发送给所有的分片，但是却需要等待所有的分片响应后才能返回给客户端，因此一次 Bulk 的延迟则取决于响应最慢的那个分片所在的节点。这就是分布式系统的长尾效应。
    
        因此，我们可以自定义 routing 值，这样写入、查询均带有路由字段信息。请求只会发送给部分分片，避免全量分片扫描。这些节点完成查询后将结果返回给请求节点，由请求节点汇聚各个节点的结果返回给客户端。
    
        ![image.png](https://r2.129870.xyz/img/202309110033326.png)
3. 主分片保存阶段
    1. 实时性要求不高的索引增大 refresh_interval 的时间
        ES 默认的 refresh_interval 是 1s，即 doc 写入 1s 后即可被搜索到。如果业务对数据实时性要求不高的话，如日志场景，可将索引模版的 refresh_interval 设置成 30s，这能够避免过多的小 segment 文件的生成及段合并的操作。
    2. 尽量选择 SSD 磁盘类型
    3. 追求写入效率的场景，可先将索引副本数设置为单副本，写入完成后再打开副本
        例如迁移数据时可先将副本数设置为 0，迁移完毕后再设置回来。
4. 数据使用阶段
    冻结历史索引，释放内存空间。
    
    ![image.png](https://r2.129870.xyz/img/202309110034464.png)
    
    Open 状态的索引由于是通过将倒排索引以 FST 数据结构的方式加载进内存中，因此索引是能够被快速搜索的，且搜索速度也是最快的。但是需要消耗大量的内存空间，且这部分内存为常驻内存，不会被 GC 的。1T 的索引预计需要消耗 2-4 GB 的 JVM 堆内存空间。
    
    > [!info]- FST
    > </br>
    > 
    > `fst`（有限状态转换机）是一种数据结构，用于高效地存储和检索有限状态自动机（finite-state automaton）的形式。
    > 
    > 有限状态转换机是一种表示字符串集合的形式，它可以支持高效的前缀搜索、模式匹配和词典查询等操作。它在自然语言处理、信息检索和编译等领域有广泛的应用，特别是在需要高效处理大量字符串数据的场景下。
    > 
    > `fst` 数据结构的特点包括：
    > 1. 紧凑性：`fst` 以紧凑的形式存储字符串集合，通过有效地压缩状态转换和共享相同的前缀来减小存储空间的需求。
    > 2. 高效性：`fst` 支持高效的字符串操作，如前缀搜索、最长前缀匹配和模式匹配。它可以在常量时间内确定输入字符串是否存在于集合中，并快速地找到匹配的前缀或最佳匹配。
    > 3. 可变性：`fst` 可以进行动态更新和修改，允许在运行时插入、删除和修改字符串集合中的元素。
    > 4. 可序列化：`fst` 可以被序列化为二进制格式，以便在不同的系统之间进行存储和传输。
    
    Frozen 状态的索引特点是可被搜索，但是由于它不占用内存，只是存储在磁盘上，因此冻结索引的搜索速度是相对比较慢的。如果我们集群中的数据量比较大，历史数据也不能被删除，则可以考虑将历史索引冻结起来，这样便可释放出较多的内存空间。

## 3.3. 集群性能优化

以下性能优化点参考[腾讯的 ES 性能优化](https://mp.weixin.qq.com/s/CNf75yT0A0QPki-Qhw3_8w)一节。

![image.png](https://r2.129870.xyz/img/202309110212386.png)

### 3.3.1. 高可用性能优化

高可用可划分为三个维度：

- 系统健壮性
    指 ES 内核自身的健壮性，也是分布式系统面临的共性难题。例如，在异常查询、压力过载下集群的容错能力；在高压力场景下，集群的可扩展性；在集群扩容、节点异常场景下，节点、多硬盘之间的数据均衡能力。
- 容灾方案
    如果通过管控系统建设，保障机房网络故障时快速恢复服务，自然灾害下防止数据丢失，误操作后快速恢复等。
- 系统缺陷
    这在任何系统发展过程中都会持续产生，比如说 Master 节点堵塞、分布式死锁、滚动重启缓慢等。

针对上述问题，下面来介绍我们在高可用方面的解决方案：

- 系统健壮性方面
    通过服务限流，容忍机器网络故障、异常查询等导致的服务不稳定。通过优化集群元数据管控逻辑，提升集群扩展能力一个数量级，支持千级节点集群、百万分片，解决集群可扩展性问题；集群均衡方面，通过优化节点、多硬盘间的分片均衡，保证大规模集群的压力均衡。
- 容灾方案方面
    通过扩展 ES 的插件机制支持备份回档，把 ES 的数据备份回档到廉价存储，保证数据的可恢复；支持跨可用区容灾，用户可以按需部署多个可用区，以容忍单机房故障。垃圾桶机制，保证用户在欠费、误操作等场景下，集群可快速恢复。
- 系统缺陷方面
    修复了滚动重启、Master 阻塞、分布式死锁等一系列 Bug。其中滚动重启优化，可加速节点重启速度 5+倍，具体可参考 PR [ES-46520](https://github.com/elastic/elasticsearch/pull/46520)；

![image.png](https://r2.129870.xyz/img/202309110214557.png)

### 3.3.2. 成本优化

硬盘成本方面，由于数据具有明显的冷热特性，首先我们采用冷热分离架构，使用混合存储的方案来平衡成本、性能；其次，既然对历史数据通常都是访问统计信息，那么以通过预计算来换取存储和性能；如果历史数据完全不使用，也可以备份到更廉价的存储系统；其他一些优化方式包含存储裁剪、生命周期管理等。

内存成本方面，很多用户在使用大存储机型时会发现，存储资源才用了百分之二十，内存已经不足。其实基于时序数据的访问特性，我们可以利用 Cache 进行优化。

![image.png](https://r2.129870.xyz/img/202309110216043.png)

### 3.3.3. 性能优化

以日志、监控为代表的时序场景，对写入性能要求非常高，写入并发可达 1000w/s。然而我们发现在带主键写入时，ES 性能衰减 1+倍，部分压测场景下，CPU 无法充分利用。以搜索服务为代表的场景，对查询性的要求非常高，要求 20w QPS, 平响 20ms，而且尽量避免 GC、执行计划不优等造成的查询毛刺。

![image.png](https://r2.129870.xyz/img/202309110217952.png)

# 4. 集群运维经验

## 4.1. 集群问题排查
ES 集群的健康状态分为三种，分别是 Green、Yellow 和 Red。

- Green (绿色)：全部主&副本分片分配成功；
- Yellow (黄色)：至少有一个副本分片未分配成功；
- Red (红色)：至少有一个主分片未分配成功。

### 4.1.1. 查看集群健康状态

可以通过下面的 API 来查询集群的健康状态及未分配的分片个数：

```bash
GET _cluster/health
{
  "cluster_name": "es-xxxxxxx",
  "status": "yellow",
  "timed_out": false,
  "number_of_nodes": 103,
  "number_of_data_nodes": 100,
  "active_primary_shards": 4610,
  "active_shards": 9212,
  "relocating_shards": 0,
  "initializing_shards": 0,
  "unassigned_shards": 8,
  "delayed_unassigned_shards": 0,
  "number_of_pending_tasks": 0,
  "number_of_in_flight_fetch": 0,
  "task_max_waiting_in_queue_millis": 0,
  "active_shards_percent_as_number": 99.91323210412148
}
```

其中需要重点关注的几个字段有 status、number_of_nodes、unassigned_shards 和 number_of_pending_tasks。number_of_pending_tasks 这个字段如果很高的话，通常是由于 master 节点触发的元数据更新操作，部分节点响应超时导致的大量的任务堆积。我们可以通过下面的 API 来查看具体有那些 task 需要执行：

```bash
GET /_cat/pending_tasks
```

### 4.1.2. 查看分片未分配原因

当集群 Red 时候，我们可以通过下面的 API 来查看分片未分配的原因：

```bash
GET _cluster/allocation/explain
```

其中 index 和 shard 列出了具体哪个索引的哪个分片未分配成功。reason 字段则列出了哪种原因导致的分片未分配。这里也将所有可能的原因列出来：

```text
INDEX_CREATED：由于创建索引的 API 导致未分配。
CLUSTER_RECOVERED ：由于完全集群恢复导致未分配。
INDEX_REOPENED ：由于打开 open 或关闭 close 一个索引导致未分配。
DANGLING_INDEX_IMPORTED ：由于导入 dangling 索引的结果导致未分配。
NEW_INDEX_RESTORED ：由于恢复到新索引导致未分配。
EXISTING_INDEX_RESTORED ：由于恢复到已关闭的索引导致未分配。
REPLICA_ADDED：由于显式添加副本分片导致未分配。
ALLOCATION_FAILED ：由于分片分配失败导致未分配。
NODE_LEFT ：由于承载该分片的节点离开集群导致未分配。
REINITIALIZED ：由于当分片从开始移动到初始化时导致未分配（例如，使用影子 shadow 副本分片）。REROUTE_CANCELLED ：作为显式取消重新路由命令的结果取消分配。
REALLOCATED_REPLICA ：确定更好的副本位置被标定使用，导致现有的副本分配被取消，出现未分配。
```

常见分片未分配的原因如下：

1. 磁盘满了
    通常如果磁盘满了，ES 为了保证集群的稳定性，会将该节点上所有的索引设置为只读。ES `7.x` 版本之后当磁盘空间提升后可自动解除，但是 `7.x` 版本之前则需要手动执行下面的 API 来解除只读模式：
    
    ```bash
    PUT index_name/_settings
    {
     "index": {
       "blocks": {
         "read_only_allow_delete": null
        }
      }
    }
    ```
16. 分片数量超过 21 亿条限制
    该限制是分片维度而不是索引维度的。因此出现这种异常，通常是由于我们的索引分片设置的不是很合理。
    
    解决方法：切换写入到新索引，并修改索引模版，合理设置主分片数。
17. 主分片所在节点掉线
    这种情况通常是由于某个节点故障或者由于负载较高导致的掉线。
    
    解决方法：找到节点掉线原因并重新启动节点加入集群，等待分片恢复。
18. 索引所需属性与节点属性不匹配
    解决方法：重新设置索引所需的属性，和节点保持一致。因为如果重新设置节点属性，则需要重启节点，代价较高。
19. 节点长时间掉线后重新加入集群，引入了脏数据
    解决方法：通过 reroute API 来重新分配一个主分片。
20. 未分配分片太多，达到了分片恢复的阈值，其他分片排队等待
    这种情况通常出现在集群重启，或者某一个节点重启后。且由于设置的分片并发恢复的值较低导致。为了尽快恢复集群健康状态，可以通过调用下面的 API 来提升分片恢复的速度和并发度：
    
    ```bash
    PUT /_cluster/settings
    {
        "transient" : {
            "cluster.routing.allocation.node_concurrent_recoveries": "20",
            "indices.recovery.max_bytes_per_sec": "100mb"
        }
    }
    ```

## 4.2. 索引声明周期管理

在生产环境使用 ES 要面对的第一个问题通常是索引容量的规划，不合理的分片数，副本数和分片大小会对索引的性能产生直接的影响。查询和写入的性能与索引的大小是正相关的，所以要保证高性能，一定要限制索引的大小，具体来说是就限制分片数量和单个分片的大小。

### 4.2.1. 使用 Rollover 管理索引

Rollover 的原理是使用一个别名指向真正的索引，当指向的索引满足一定条件（文档数或时间或索引大小）更新实际指向的索引。

1. 创建索引并使用别名
     索引名称的格式为 {.\*}-d 这种格式的，数字默认是 6 位：
    ```bash
    PUT myro-000001
    {
      "aliases": {
        "myro_write_alias":{}
      }
    }
    ```
22. 通过别名写入数据
    例如 `POST /myro_write_alias/_bulk`
23. 执行 rollover 操作
    
    rollover 的 3 个条件是或关系，任意一个条件满足就会发生 rollover：
    
    ```bash
    POST /myro_write_alias/_rollover
    {
      "conditions": {
        "max_age":   "7d",
        "max_docs":  3,
        "max_size": "5gb"
      }
    }
    ```

使用 Rollover 的缺点：

- 必须明确执行了 rollover 指令才会更新 rollover 的别名对应的索引。
- 通常可以在写入数据之后再执行一下 rollover 命令，或者采用配置系统 cron 脚本的方式。
- 增加了使用的 rollover 的成本，对于开发者来说不够自动化。

### 4.2.2. 使用 ILM（Index Lifecycle Management ） 管理索引

ES 一直在索引管理这块进行优化迭代，从 6.7 版本推出了索引生命周期管理（Index Lifecycle Management ，简称 ILM)机制，是目前官方提供的比较完善的索引管理方法。所谓 Lifecycle (生命周期)是把索引定义了四个阶段：

![image.png](https://r2.129870.xyz/img/202309110159297.png)

- Hot：索引可写入，也可查询，也就是我们通常说的热数据，为保证性能数据通常都是在内存中的。
- Warm：索引不可写入，但可查询，介于热和冷之间，数据可以是全内存的，也可以是在 SSD 的硬盘上的。
- Cold：索引不可写入，但很少被查询，查询的慢点也可接受，基本不再使用的数据，数据通常在大容量的磁盘上。
- Delete：索引可被安全的删除。

这 4 个阶段是 ES 定义的一个索引从生到死的过程, Hot -> Warm -> Cold -> Delete 4 个阶段只有 Hot 阶段是必须的，其他 3 个阶段根据业务的需求可选。

通过 ILM 按以下方式管理索引：

1. 建立 Lifecycle 策略
    
    ![image.png](https://r2.129870.xyz/img/202309110202931.png)
    
25. 建立索引模版
    
    ```bash
    PUT /_template/myes_template
    {
      "index_patterns": [
        "myes-*"
      ],
      "aliases": {
        "myes_reade_alias": {}
      },
      "settings": {
        "index": {
          "lifecycle": {
            "name": "myes-lifecycle",
            "rollover_alias": "myes_write_alias"
          },
          "refresh_interval": "30s",
          "number_of_shards": "12",
          "number_of_replicas": "1"
        }
      },
      "mappings": {
        "properties": {
          "name": {
            "type": "keyword"
          }
        }
      }
    }
    ```
    
    注意：
    - 模版匹配以索引名称 myes- 开头的索引。
    - 所有使用此模版创建的索引都有一个别名 myes_reade_alias 用于方便查询数据；
    - 模版绑定了上面创建的 Lifecycle 策略，并且用于 rollover 的别名是 myes_write_alias。
26. 创建索引
    
    ```bash
    PUT /myes-testindex-000001
    {
      "aliases": {
        "myes_write_alias":{}
      }
    }
    ```
    
    注意:
    - 索引的名称是 .\*-d 的形式；
    - 索引的别名用于 lifecycle 做 rollover。
27. 查看索引配置
    `GET /myes-testindex-000001`
28. 写入数据
    `POST /myes_write_alias/_bulk`
29. 配置 Lifecycle 自动 Rollover 的时间间隔
    由于 ES 是一个准实时系统，很多操作都不能实时生效。Lifecycle 的 rollover 之所以不用每次手动执行 rollover 操作是因为 ES 会隔一段时间判断一次索引是否满足 rollover 的条件，ES 检测 ILM 策略的时间默认为 10min。如果需要手动修改这个时间间隔的话可以修改 Lifecycle 配置：
    
    ```bash
    PUT _cluster/settings
    {
      "transient": {
        "indices.lifecycle.poll_interval": "3s"
      }
    }
    ```

