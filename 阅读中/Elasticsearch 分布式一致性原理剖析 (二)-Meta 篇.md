_本文首发于云栖社区 ([Elasticsearch 分布式一致性原理剖析 (二)-Meta 篇 - 博客 - 云栖社区 - 阿里云](https://link.zhihu.com/?target=https%3A//yq.aliyun.com/articles/576043))，由原作者转载。_

## 前言

“Elasticsearch 分布式一致性原理剖析” 系列将会对 Elasticsearch 的分布式一致性原理进行详细的剖析，介绍其实现方式、原理以及其存在的问题等 (基于 6.2 版本)。前一篇的内容包括了 ES 的集群组成、节点发现与 Master 选举、错误检测与集群扩缩容等。本篇将在前一篇的基础上，重点分析 ES 中 meta 更新的一致性问题，为了便于读者理解 ，本文还介绍了 Master 管理集群的方式、meta 组成、存储方式等等。目录如下：

1.  Master 如何管理集群
2.  Meta 组成、存储和恢复
3.  ClusterState 的更新流程
4.  如何解决当前的一致性问题
5.  小结

## Master 如何管理集群

在上一篇文章中，我们介绍了 ES 集群的组成，如何发现节点，选举 Master 等。那么在选举出 Master 之后，Master 如何管理集群呢？比如以下问题：

1.  Master 如何处理新建或删除 Index？
2.  Master 如何对 Shard 进行重新调度，实现负载均衡？

既然要管理集群，那么 Master 节点必然需要以某种方式通知其他节点，从而让其他节点执行相应的动作，来完成某些事情。比如建立一个新的 Index 就需要将其 Shard 分配在某些节点上，在这些节点上需要创建出对应 Shard 的目录，并在内存中创建对应 Shard 的一些结构等。

在 ES 中，Master 节点是通过发布 ClusterState 来通知其他节点的。Master 会将新的 ClusterState 发布给其他的所有节点，当节点收到新的 ClusterState 后，会把新的 ClusterState 发给相关的各个模块，各个模块根据新的 ClusterState 判断是否要做什么事情，比如创建 Shard 等。即这是一种通过 Meta 数据来驱动各个模块工作的方式。

在 Master 进行 Meta 变更并通知所有节点的过程中，需要考虑 Meta 变更的一致性问题，假如这个过程中 Master 挂掉了，那么可能只有部分节点按照新的 Meta 执行了操作。当选举出新的 Master 后，需要保证所有节点都要按照最新的 Meta 执行操作，不能回退，因为已经有节点按照新的 Meta 执行操作了，再回退就会导致不一致。

ES 中只要新 Meta 在一个节点上被 commit，那么就会开始执行相应的操作。因此我们要保证一旦新 Meta 在某个节点上被 commit，此后无论谁是 master，都要基于这个 commit 来产生更新的 meta，否则就可能产生不一致。本文会分析 ES 处理这一问题的策略和存在的问题。

## Meta 的组成、存储和恢复

在介绍 Meta 更新流程前，我们先介绍一下 ES 中 Meta 的组成、存储方式和恢复方式，不关心这部分内容的读者可以略过本节。

## 1. Meta：ClusterState、MetaData、IndexMetaData

Meta 是用来描述数据的数据。在 ES 中，Index 的 mapping 结构、配置、持久化状态等就属于 meta 数据，集群的一些配置信息也属于 meta。这类 meta 数据非常重要，假如记录某个 index 的 meta 数据丢失了，那么集群就认为这个 index 不再存在了。ES 中的 meta 数据只能由 master 进行更新，master 相当于是集群的大脑。

## ClusterState

集群中的每个节点都会在内存中维护一个当前的 ClusterState，表示当前集群的各种状态。ClusterState 中包含一个 MetaData 的结构，MetaData 中存储的内容更符合 meta 的特征，而且需要持久化的信息都在 MetaData 中，此外的一些变量可以认为是一些临时状态，是集群运行中动态构建出来的。

```
ClusterState内容包括：
    long version: 当前版本号，每次更新加1
    String stateUUID：该state对应的唯一id
    RoutingTable routingTable：所有index的路由表
    DiscoveryNodes nodes：当前集群节点
    MetaData metaData：集群的meta数据
    ClusterBlocks blocks：用于屏蔽某些操作
    ImmutableOpenMap<String, Custom> customs: 自定义配置
    ClusterName clusterName：集群名
```

## MetaData

上面提到，MetaData 更符合 meta 的特征，而且需要持久化，那么我们看下这个 MetaData 中主要包含哪些东西：

```
MetaData中需要持久化的包括：
    String clusterUUID：集群的唯一id。
    long version：当前版本号，每次更新加1
    Settings persistentSettings：持久化的集群设置
    ImmutableOpenMap<String, IndexMetaData> indices: 所有Index的Meta
    ImmutableOpenMap<String, IndexTemplateMetaData> templates：所有模版的Meta
    ImmutableOpenMap<String, Custom> customs: 自定义配置
```

我们看到，MetaData 主要是集群的一些配置，集群所有 Index 的 Meta，所有 Template 的 Meta。下面我们再分析一下 IndexMetaData，后面还会讲到，虽然 IndexMetaData 也是 MetaData 的一部分，但是存储上却是分开存储的。

## IndexMetaData

IndexMetaData 指具体某个 Index 的 Meta，比如这个 Index 的 shard 数，replica 数，mappings 等。

```
IndexMetaData中需要持久化的包括：
    long version：当前版本号，每次更新加1。
    int routingNumShards: 用于routing的shard数, 只能是该Index的numberOfShards的倍数，用于split。
    State state: Index的状态, 是个enum，值是OPEN或CLOSE。
    Settings settings：numbersOfShards，numbersOfRepilicas等配置。
    ImmutableOpenMap<String, MappingMetaData> mappings：Index的mapping
    ImmutableOpenMap<String, Custom> customs：自定义配置。
    ImmutableOpenMap<String, AliasMetaData> aliases： 别名
    long[] primaryTerms：primaryTerm在每次Shard切换Primary时加1，用于保序。
    ImmutableOpenIntMap<Set<String>> inSyncAllocationIds：处于InSync状态的AllocationId，用于保证数据一致性，下一篇文章会介绍。
```

## 2. Meta 的存储

首先，在启动 ES 的一个节点时，会配置一个 data 目录，例如下面这个目录。该节点只有一个单 shard 的 Index。

```
$tree
.
`-- nodes
    `-- 0
        |-- _state
        |   |-- global-1.st
        |   `-- node-0.st
        |-- indices
        |   `-- 2Scrm6nuQOOxUN2ewtrNJw
        |       |-- 0
        |       |   |-- _state
        |       |   |   `-- state-0.st
        |       |   |-- index
        |       |   |   |-- segments_1
        |       |   |   `-- write.lock
        |       |   `-- translog
        |       |       |-- translog-1.tlog
        |       |       `-- translog.ckp
        |       `-- _state
        |           `-- state-2.st
        `-- node.lock
```

我们看到，ES 进程会把 Meta 和 Data 都写入这个目录中，其中目录名为_state 的代表该目录存储的是 meta 文件，根据文件层级的不同，共有 3 种 meta 的存储：

*   nodes/0/_state/:

这层目录在节点级别，该目录下的 global-1.st 文件存储的是上文介绍的 MetaData 中除去 IndexMetaData 的部分，即一些集群级别的配置和 templates。node-0.st 中存储的是 NodeId。

*   nodes/0/indices/2Scrm6nuQOOxUN2ewtrNJw/_state/:

这层目录在 index 级别，2Scrm6nuQOOxUN2ewtrNJw 是 IndexId，该目录下的 state-2.st 文件存储的是上文介绍的 IndexMetaData。

*   nodes/0/indices/2Scrm6nuQOOxUN2ewtrNJw/0/_state/:

这层目录在 shard 级别，该目录下的 state-0.st 存储的是 ShardStateMetaData，包含是否是 primary 和 allocationId 等信息。ShardStateMetaData 是在 IndexShard 模块中管理，与其他 Meta 关联不大，本文不做过多介绍。

可以看到，集群相关的 MetaData 和 Index 的 MetaData 是在不同的目录中存储的。另外，集群相关的 Meta 会在所有的 MasterNode 和 DataNode 上存储，而 Index 的 Meta 会在所有的 MasterNode 和存储了该 Index 数据的 DataNode 上存储。

这里有个问题是，MetaData 是由 Master 管理的，为什么 DataNode 上也要保存 MetaData 呢？主要原因是考虑到数据的安全性，很多用户没有考虑 Master 节点的高可用和数据高可靠，在部署 ES 集群时只配置了一个 MasterNode，如果这个节点不可用，就会出现 Meta 丢失，后果非常严重。

## 3. Meta 的恢复

假设 ES 集群重启了，那么所有进程都没有了之前的 Meta 信息，需要有一个角色来恢复 Meta，这个角色就是 Master。所以 ES 集群需要先进行 Master 选举，选出 Master 后，才会进行故障恢复。

当 Master 选举出来后，Master 进程还会等待一些条件，比如集群当前的节点数大于某个数目等，这是避免有些 DataNode 还没有连上来，造成不必要的数据恢复等。

当 Master 进程决定进行恢复 Meta 时，它会向集群中的 MasterNode 和 DataNode 请求其机器上的 MetaData。对于集群的 Meta，选择其中 version 最大的版本。对于每个 Index 的 Meta，也选择其中最大的版本。然后将集群的 Meta 和每个 Index 的 Meta 再组合起来，构成当前的最新 Meta。

## ClusterState 的更新流程

现在我们开始分析 ClusterState 的更新流程，并通过这个流程来看 ES 如何保证 Meta 更新的一致性。

## 1. master 进程内不同线程更改 ClusterState 时的原子性保证

首先，master 进程内不同线程更改 ClusterState 时要保证是原子的。试想一下这个场景，有两个线程都在修改 ClusterState，各自更改其中的一部分。假如没有任何并发的保护，那么最后提交的线程可能就会覆盖掉前一个线程的修改，或者产生不符合条件的状态变更的发生。

ES 解决这个问题的方式是，每次需要更新 ClusterState 时提交一个 Task 给 MasterService，MasterService 中只使用一个线程来串行处理这些 Task，每次处理时把当前的 ClusterState 作为 Task 中 execute 函数的参数。即保证了所有的 Task 都是在 currentClusterState 的基础上进行更改，然后不同的 Task 是串行执行的。

## 2. ClusterState 更改如何保证一旦 commit，后续就一定会在此基础上 commit，不会回退

这里是为了解决这样一个问题，我们知道，新的 Meta 一旦在某个节点上 commit，那么这个节点就会执行相应的操作，比如删除某个 Shard 等，这样的操作是不可回退的。而假如此时 Master 节点挂掉了，新产生的 Master 一定要在新的 Meta 上进行更改，不能出现回退，否则就会出现 Meta 回退了但是操作无法回退的情况。本质上就是 Meta 更新没有保证一致性。

早期的 ES 版本没有解决这个问题，后来引入了两阶段提交的方式 ([Add two phased commit to Cluster State publishing](https://link.zhihu.com/?target=https%3A//github.com/elastic/elasticsearch/pull/13062))。所谓的两阶段提交，是把 Master 发布 ClusterState 分成两步，第一步是向所有节点 send 最新的 ClusterState，当有超过半数的 master 节点返回 ack 时，再发送 commit 请求，要求节点 commit 接收到的 ClusterState。如果没有超过半数的节点返回 ack，那么认为本次发布失败，同时退出 master 状态，执行 rejoin 重新加入集群。

![](https://pic3.zhimg.com/v2-d50f59c6b9525cd0b2c7133a2954f7c2_r.jpg)

两阶段提交可以解决部分一致性问题，比如以下这种场景：

1.  NodeA 本来是 Master 节点，但由于某些原因 NodeB 成了新的 Master 节点，而 NodeA 由于探测不及时还未发现。
2.  NodeA 认为自己仍然是 Master，于是照常发布新的 ClusterState。
3.  由于此时 NodeB 是 Master，说明超过半数的 Master 节点认为 NodeB 才是新的 Master，于是超过半数的 Master 节点不会返回 ack 给 NodeA。
4.  NodeA 收集不到足够的 ack，于是本次发布失败，同时退出 master 状态。
5.  新的 ClusterState 不会在任何节点上 commit，于是没有不一致发生。

但是这种方式也存在很多一致性的问题，我们下一节来具体分析。

## 3. 一致性问题分析

ES 中，Master 发送 commit 的原则是只要有超过半数 MasterNode(master-eligible node) 接收了新的 ClusterState 就发送 commit。那么实际上就是认为只要超过半数节点接收了新的 ClusterState，这个 ClusterState 就一定可以被 commit，不会在各种场景下回退。

## 问题 1

第一阶段 master 节点 send 新的 ClusterState，接收到的节点只是把新的 ClusterState 放入内存一个队列中，就会返回 ack。这个过程没有持久化，所以当 master 接收到超过半数的 ack 后，也不能认为这些节点上都有新的 ClusterState 了。

## 问题 2

如果 master 在 commit 阶段，只 commit 了少数几个节点就出现了网络分区，将 master 与这几个少数节点分在了一起，其他节点可以互相访问。此时其他节点构成多数派，会选举出新的 master，由于这部分节点中没有任何节点 commit 了新的 ClusterState，所以新的 master 仍会使用更新前的 ClusterState，造成 Meta 不一致。

ES 官方仍在追踪这个 bug：

```
https://www.elastic.co/guide/en/elasticsearch/resiliency/current/index.html

Repeated network partitions can cause cluster state updates to be lost (STATUS: ONGOING)
...This problem is mostly fixed by #20384 (v5.0.0), which takes committed cluster state updates into account during master election. This considerably reduces the chance of this rare problem occurring but does not fully mitigate it. If the second partition happens concurrently with a cluster state update and blocks the cluster state commit message from reaching a majority of nodes, it may be that the in flight update will be lost. If the now-isolated master can still acknowledge the cluster state update to the client this will amount to the loss of an acknowledged change. Fixing that last scenario needs considerable work. We are currently working on it but have no ETA yet.
```

## 什么情况下会有问题

在两阶段提交中，很重要的一个条件是，“超过半数节点返回 ack，表示接收了这个 ClusterState”，这个条件并不能带来任何保证。一方面接收 ClusterState 并不会持久化，另一方面接收了 ClusterState 也不会对未来选举新 Master 产生任何干扰，因为选举时只考虑已经 commit 的 ClusterState。

在两阶段过程中，只到 Master 在超过半数 MasterNode(master-eligible node) 上 commit 了新的 ClusterState，并且这些节点都完成了新 ClusterState 的持久化时，才到达一个安全的状态。在开始 commit 到达到这个状态之间，如果 Master 发送故障，就可能导致 Meta 发生不一致。

## 如何解决当前的一致性问题

既然 ES 目前 Meta 更新存在一些一致性问题，那么我们来脑洞一下，如何解决这些一致性问题呢？

## 1. 实现一个标准的一致性算法，比如 raft

第一种方式是实现一个标准的一致性算法，比如 raft。在上一篇中，我们也比较了 ES 的选举算法与 raft 算法的异同，下面我们继续比较一下 ES 的 meta 更新流程与 raft 的日志复制流程。

相同点：

1.  都是在得到超过半数节点应答之后，执行 commit。

不同点：

1.  raft 算法中，follower 接收到日志后就会进行持久化，写到磁盘上。ES 中，节点接收到 ClusterState 只是放到内存中的一个队列中即返回，并不持久化。
2.  raft 算法可以保证在超过半数节点应答之后，这条日志一定可以被 commit，而 ES 中没有保证这一点，目前还存在一致性问题。

通过上面的比较，我们再次看到，ES 中 meta 更新的算法与 raft 相比很相似，raft 使用了更多的机制来保证一致性，而 ES 还存在一些问题。用 ES 官方的话来说，fix 这些问题还需要大量的工作 (considerable work)。

## 2. 借助额外的组件保证 meta 一致性

比如使用 Zookeeper 来保存 Meta，用 Zookeeper 来保证 Meta 的一致性。这样可以解决一致性的问题，但是性能的问题还需要再评估一下，比如 Meta 是否会过大而导致不保存在 Zookeeper 中，每次请求全量 Meta 还是 Diff 等。

## 3. 使用共享存储来保存 Meta

首先保证不会出现脑裂，然后可以使用共享存储来保存 Meta，解决 Meta 一致性的问题。这种方式会引入一个共享存储，这个存储系统需要具备高可靠、高可用特征。

## 小结

作为 Elasticsearch 分布式一致性原理系列的第二篇，本文主要介绍了 ES 集群中 Master 节点发布 Meta 更新的流程，分析了其中的一致性问题。此外还介绍了 Meta 数据的组成和存储方式等。下一篇文章会分析 ES 中如何保证数据的一致性，介绍其写入流程和算法模型等。

## Reference

[Elasticsearch Resiliency Status](https://link.zhihu.com/?target=https%3A//www.elastic.co/guide/en/elasticsearch/resiliency/current/index.html)

[Add two phased commit to Cluster State publishing #13062](https://link.zhihu.com/?target=https%3A//github.com/elastic/elasticsearch/pull/13062)

[The Raft Consensus Algorithm](https://link.zhihu.com/?target=https%3A//raft.github.io/)