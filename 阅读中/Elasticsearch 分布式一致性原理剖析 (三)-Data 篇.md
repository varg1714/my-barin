_本文首发于云栖社区 (_[Elasticsearch 分布式一致性原理剖析 (三)-Data 篇 - 博客 - 云栖社区 - 阿里云](https://link.zhihu.com/?target=https%3A//yq.aliyun.com/articles/576044)_)，由原作者转载。_

## 前言

“Elasticsearch 分布式一致性原理剖析” 系列将会对 Elasticsearch 的分布式一致性原理进行详细的剖析，介绍其实现方式、原理以及其存在的问题等 (基于 6.2 版本)。前两篇文章介绍了 ES 中集群如何组成，master 选举算法，master 更新 meta 的流程等，并分析了选举、Meta 更新中的一致性问题。本文会分析 ES 中的数据流，包括其写入流程、算法模型 PacificA、SequenceNumber 与 Checkpoint 等，并比较 ES 的实现与标准 PacificA 算法的异同。目录如下：

1.  问题背景
2.  数据写入流程
3.  PacificA 算法
4.  SequenceNumber、Checkpoint 与故障恢复
5.  ES 与 PacificA 的比较
6.  小结

## 问题背景

用过 ES 的同学都知道，ES 中每个 Index 会划分为多个 Shard，Shard 分布在不同的 Node 上，以此来实现分布式的存储和查询，支撑大规模的数据集。对于每个 Shard，又会有多个 Shard 的副本，其中一个为 Primary，其余的一个或多个为 Replica。数据在写入时，会先写入 Primary，由 Primary 将数据再同步给 Replica。在读取时，为了提高读取能力，Primary 和 Replica 都会接受读请求。

![](https://pic4.zhimg.com/v2-7dfe9fb9b58745eee5c6b09fa8abca53_r.jpg)

在这种模型下，我们能够感受到 ES 具有这样的一些特性，比如：

1.  数据高可靠：数据具有多个副本。
2.  服务高可用：Primary 挂掉之后，可以从 Replica 中选出新的 Primary 提供服务。
3.  读能力扩展：Primary 和 Replica 都可以承担读请求。
4.  故障恢复能力：Primary 或 Replica 挂掉都会导致副本数不足，此时可以由新的 Primary 通过复制数据产生新的副本。

另外，我们也可以想到一些问题，比如：

1.  数据怎么从 Primary 复制到 Replica？
2.  一次写入要求所有副本都成功吗？
3.  Primary 挂掉会丢数据吗？
4.  数据从 Replica 读，总是能读到最新数据吗？
5.  故障恢复时，需要拷贝 Shard 下的全部数据吗？

可以看到，对于 ES 中的数据一致性，虽然我们可以很容易的了解到其大概原理，但是对其细节我们还有很多的困惑。那么本文就从 ES 的写入流程，采用的一致性算法，SequenceId 和 Checkpoint 的设计等方面来介绍 ES 如何工作，进而回答上述这些问题。需要注意的是，本文基于 ES6.2 版本进行分析，可能很多内容并不适用于 ES 之前的版本，比如 2.X 的版本等。

## 数据写入流程

首先我们来看一下数据的写入流程，读者也可以阅读这篇文章来详细了解：[https://zhuanlan.zhihu.com/p/34669354](https://zhuanlan.zhihu.com/p/34669354)。

## Replication 角度: Primary -> Replica

我们从大的角度来看，ES 写入流程为先写入 Primary，再并发写入 Replica，最后应答客户端，流程如下：

*   检查 Active 的 Shard 数。

```
final String activeShardCountFailure = checkActiveShardCount();
```

*   写入 Primary。

```
primaryResult = primary.perform(request);
```

*   并发的向所有 Replicate 发起写入请求

```
performOnReplicas(replicaRequest, globalCheckpoint, replicationGroup.getRoutingTable());
```

*   等所有 Replicate 返回或者失败后，返回给 Client。

```
private void decPendingAndFinishIfNeeded() {
     assert pendingActions.get() > 0 : "pending action count goes below 0 for request [" + request + "]";
     if (pendingActions.decrementAndGet() == 0) {
         finish();
     }
 }
```

上述过程在 ReplicationOperation 类的 execute 函数中，完整代码如下：

```
public void execute() throws Exception {
        final String activeShardCountFailure = checkActiveShardCount();
        final ShardRouting primaryRouting = primary.routingEntry();
        final ShardId primaryId = primaryRouting.shardId();
        if (activeShardCountFailure != null) {
            finishAsFailed(new UnavailableShardsException(primaryId,
                "{} Timeout: [{}], request: [{}]", activeShardCountFailure, request.timeout(), request));
            return;
        }

        totalShards.incrementAndGet();
        pendingActions.incrementAndGet(); // increase by 1 until we finish all primary coordination
        primaryResult = primary.perform(request);
        primary.updateLocalCheckpointForShard(primaryRouting.allocationId().getId(), primary.localCheckpoint());
        final ReplicaRequest replicaRequest = primaryResult.replicaRequest();
        if (replicaRequest != null) {
            if (logger.isTraceEnabled()) {
                logger.trace("[{}] op [{}] completed on primary for request [{}]", primaryId, opType, request);
            }

            // we have to get the replication group after successfully indexing into the primary in order to honour recovery semantics.
            // we have to make sure that every operation indexed into the primary after recovery start will also be replicated
            // to the recovery target. If we used an old replication group, we may miss a recovery that has started since then.
            // we also have to make sure to get the global checkpoint before the replication group, to ensure that the global checkpoint
            // is valid for this replication group. If we would sample in the reverse, the global checkpoint might be based on a subset
            // of the sampled replication group, and advanced further than what the given replication group would allow it to.
            // This would entail that some shards could learn about a global checkpoint that would be higher than its local checkpoint.
            final long globalCheckpoint = primary.globalCheckpoint();
            final ReplicationGroup replicationGroup = primary.getReplicationGroup();
            markUnavailableShardsAsStale(replicaRequest, replicationGroup.getInSyncAllocationIds(), replicationGroup.getRoutingTable());
            performOnReplicas(replicaRequest, globalCheckpoint, replicationGroup.getRoutingTable());
        }

        successfulShards.incrementAndGet();  // mark primary as successful
        decPendingAndFinishIfNeeded();
    }
```

下面我们针对这个流程，来分析几个问题：

## 1. 为什么第一步要检查 Active 的 Shard 数？

ES 中有一个参数，叫做 wait_for_active_shards，这个参数是 Index 的一个 setting，也可以在请求中带上这个参数。这个参数的含义是，在每次写入前，该 shard 至少具有的 active 副本数。假设我们有一个 Index，其每个 Shard 有 3 个 Replica，加上 Primary 则总共有 4 个副本。如果配置 wait_for_active_shards 为 3，那么允许最多有一个 Replica 挂掉，如果有两个 Replica 挂掉，则 Active 的副本数不足 3，此时不允许写入。

这个参数默认是 1，即只要 Primary 在就可以写入，起不到什么作用。如果配置大于 1，可以起到一种保护的作用，保证写入的数据具有更高的可靠性。但是这个参数只在写入前检查，并不保证数据一定在至少这些个副本上写入成功，所以并不是严格保证了最少写入了多少个副本。关于这一点，可参考以下官方文档：

```
https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html
...It is important to note that this setting greatly reduces the chances of the write operation not writing to the requisite number of shard copies, but it does not completely eliminate the possibility, because this check occurs before the write operation commences. Once the write operation is underway, it is still possible for replication to fail on any number of shard copies but still succeed on the primary. The _shards section of the write operation’s response reveals the number of shard copies on which replication succeeded/failed.
```

## 2. 写入 Primary 完成后，为何要等待所有 Replica 响应 (或连接失败) 后返回

在更早的 ES 版本，Primary 和 Replica 之间是允许异步复制的，即写入 Primary 成功即可返回。但是这种模式下，如果 Primary 挂掉，就有丢数据的风险，而且从 Replica 读数据也很难保证能读到最新的数据。所以后来 ES 就取消异步模式了，改成 Primary 等 Replica 返回后再返回给客户端。

因为 Primary 要等所有 Replica 返回才能返回给客户端，那么延迟就会受到最慢的 Replica 的影响，这确实是目前 ES 架构的一个弊端。之前曾误认为这里是等 wait_for_active_shards 个副本写入成功即可返回，但是后来读源码发现是等所有 Replica 返回的。

```
https://github.com/elastic/elasticsearch/blob/master/docs/reference/docs/data-replication.asciidoc
... Once all replicas have successfully performed the operation and responded to the primary, the primary acknowledges the successful completion of the request to the client.
```

如果 Replica 写入失败，ES 会执行一些重试逻辑等，但最终并不强求一定要在多少个节点写入成功。在返回的结果中，会包含数据在多少个 shard 中写入成功了，多少个失败了：

```
{
    "_shards" : {
        "total" : 2,
        "failed" : 0,
        "successful" : 2
    }
}
```

## 3. 如果某个 Replica 持续写失败，用户是否会经常查到旧数据？

这个问题是说，假如一个 Replica 持续写入失败，那么这个 Replica 上的数据可能落后 Primary 很多。我们知道 ES 中 Replica 也是可以承担读请求的，那么用户是否会读到这个 Replica 上的旧数据呢？

答案是如果一个 Replica 写失败了，Primary 会将这个信息报告给 Master，然后 Master 会在 Meta 中更新这个 Index 的 InSyncAllocations 配置，将这个 Replica 从中移除，移除后它就不再承担读请求。在 Meta 更新到各个 Node 之前，用户可能还会读到这个 Replica 的数据，但是更新了 Meta 之后就不会了。所以这个方案并不是非常的严格，考虑到 ES 本身就是一个近实时系统，数据写入后需要 refresh 才可见，所以一般情况下，在短期内读到旧数据应该也是可接受的。

```
ReplicationOperation.java，写入Replica失败的OnFailure函数：

            public void onFailure(Exception replicaException) {
                logger.trace(
                    (org.apache.logging.log4j.util.Supplier<?>) () -> new ParameterizedMessage(
                        "[{}] failure while performing [{}] on replica {}, request [{}]",
                        shard.shardId(),
                        opType,
                        shard,
                        replicaRequest),
                    replicaException);
                if (TransportActions.isShardNotAvailableException(replicaException)) {
                    decPendingAndFinishIfNeeded();
                } else {
                    RestStatus restStatus = ExceptionsHelper.status(replicaException);
                    shardReplicaFailures.add(new ReplicationResponse.ShardInfo.Failure(
                        shard.shardId(), shard.currentNodeId(), replicaException, restStatus, false));
                    String message = String.format(Locale.ROOT, "failed to perform %s on replica %s", opType, shard);
                    replicasProxy.failShardIfNeeded(shard, message,
                            replicaException, ReplicationOperation.this::decPendingAndFinishIfNeeded,
                            ReplicationOperation.this::onPrimaryDemoted, throwable -> decPendingAndFinishIfNeeded());
                }
            }

调用failShardIfNeeded：

        public void failShardIfNeeded(ShardRouting replica, String message, Exception exception,
                                      Runnable onSuccess, Consumer<Exception> onPrimaryDemoted, Consumer<Exception> onIgnoredFailure) {

            logger.warn((org.apache.logging.log4j.util.Supplier<?>)
                    () -> new ParameterizedMessage("[{}] {}", replica.shardId(), message), exception);
            shardStateAction.remoteShardFailed(replica.shardId(), replica.allocationId().getId(), primaryTerm, message, exception,
                    createListener(onSuccess, onPrimaryDemoted, onIgnoredFailure));
        }

shardStateAction.remoteShardFailed向Master发送请求，执行该Replica的ShardFailed逻辑，将Shard从InSyncAllocation中移除。

    public void shardFailed(ShardRouting failedShard, UnassignedInfo unassignedInfo) {
        if (failedShard.active() && unassignedInfo.getReason() != UnassignedInfo.Reason.NODE_LEFT) {
            removeAllocationId(failedShard);

            if (failedShard.primary()) {
                Updates updates = changes(failedShard.shardId());
                if (updates.firstFailedPrimary == null) {
                    // more than one primary can be failed (because of batching, primary can be failed, replica promoted and then failed...)
                    updates.firstFailedPrimary = failedShard;
                }
            }
        }

        if (failedShard.active() && failedShard.primary()) {
            increasePrimaryTerm(failedShard.shardId());
        }
    }
```

ES 中维护 InSyncAllocation 的做法，是遵循的 PacificA 算法，下一节会详述。

## Primary 自身角度

从 Primary 自身的角度，一次写入请求会先写入 Lucene，然后写入 translog。具体流程可以看这篇文章：[https://zhuanlan.zhihu.com/p/34669354](https://zhuanlan.zhihu.com/p/34669354) 。

## 1. 为什么要写 translog？

translog 类似于数据库中的 commitlog，或者 binlog。只要 translog 写入成功并 flush，那么这笔数据就落盘了，数据安全性有了保证，Segment 就可以晚一点落盘。因为 translog 是 append 方式写入，写入性能也会比随机写更高。

另一方面是，translog 记录了每一笔数据更改，以及数据更改的顺序，所以 translog 也可以用于数据恢复。数据恢复包含两方面，一方面是节点重启后，从 translog 中恢复重启前还未落盘的 Segment 数据，另一方面是用于 Primary 和新的 Replica 之间的数据同步，即 Replica 逐步追上 Primary 数据的过程。

## 2. 为什么先写 Lucene，再写 translog？

写 Lucene 是写入内存，写入后在内存中 refresh 即可读到，写 translog 是落盘，为了数据持久化以及恢复。正常来讲，分布式系统中是先写 commitLog 进行数据持久化，再在内存中 apply 这次更改，那么 ES 为什么要反其道而行之呢？主要原因大概是写入 Lucene 时，Lucene 会再对数据进行一些检查，有可能出现写入 Lucene 失败的情况。如果先写 translog，那么就要处理写入 translog 成功但是写入 Lucene 一直失败的问题，所以 ES 采用了先写 Lucene 的方式。

## PacificA 算法

PacificA 是微软亚洲研究院提出的一种用于日志复制系统的分布式一致性算法，论文发表于 2008 年 ([PacificA paper](https://link.zhihu.com/?target=https%3A//www.microsoft.com/en-us/research/publication/pacifica-replication-in-log-based-distributed-storage-systems/))。ES 官方明确提出了其 Replication 模型基于该算法：

```
https://github.com/elastic/elasticsearch/blob/master/docs/reference/docs/data-replication.asciidoc
Elasticsearch’s data replication model is based on the primary-backup model and is described very well in the PacificA paper of Microsoft Research. That model is based on having a single copy from the replication group that acts as the primary shard. The other copies are called replica shards. The primary serves as the main entry point for all indexing operations. It is in charge of validating them and making sure they are correct. Once an index operation has been accepted by the primary, the primary is also responsible for replicating the operation to the other copies.
```

网上讲解这个算法的文章较少，因此本文根据 PacificA 的论文，简单介绍一下这个算法。该算法具有以下几个特点：

1.  强一致性。
2.  单 Primary 向多 Secondary 的数据同步模式。
3.  使用额外的一致性组件维护 Configuration。
4.  少数派 Replica 可用时仍可写入。

## 一些名词

首先我们介绍一下算法中的一些名词：

1.  Replica Group：一个互为副本的数据集合叫做 Replica Group，每个副本是一个 Replica。一个 Replica Group 中只有一个副本是 Primary，其余为 Secondary。
2.  Configuration：一个 Replica Group 的 Configuration 描述了这个 Replica Group 包含哪些副本，其中 Primary 是谁等。
3.  Configuration Version：Configuration 的版本号，每次 Configuration 发生变更时加 1。
4.  Configuration Manager: 管理 Configuration 的全局组件，其保证 Configuration 数据的一致性。Configuration 变更会由某个 Replica 发起，带着 Version 发送给 Configuration Manager，Configuration Manager 会检查 Version 是否正确，如果不正确则拒绝更改。
5.  Query & Update：对一个 Replica Group 的操作分为两种，Query 和 Update，Query 不会改变数据，Update 会更改数据。
6.  Serial Number(sn)：代表每个 Update 操作执行的顺序，每次 Update 操作加 1，为连续的数字。
7.  Prepared List：Update 操作的准备序列。
8.  Committed List：Update 操作的提交序列，提交序列中的操作一定不会丢失 (除非全部副本挂掉)。在同一个 Replica 上，Committed List 一定是 Prepared List 的前缀。

## Primary Invariant

在 PacificA 算法中，要求采用某种错误检测机制来满足以下不变式：

**Primary Invariant:** 任何时候，当一个 Replica 认为自己是 Primary 时，Configuration Manager 中维护的 Configuration 也认为其是当前的 Primary。任何时候，最多只有一个 Replica 认为自己是这个 Replica Group 的 Primary。

Primary Invariant 保证了当一个节点认为自己是 Primary 时，其肯定是当前的 Primary。如果不能满足 Primary Invariant，那么 Query 请求就可能发送给 Old Primary，读到旧的数据。

怎么保证满足 Primary Invariant 呢？论文给出的一种方法是通过 Lease 机制，这也是分布式系统中常用的一种方式。具体来说，Primary 会定期获取一个 Lease，获取之后认为某段时间内自己肯定是 Primary，一旦超过这个时间还未获取到新的 Lease 就退出 Primary 状态。只要各个机器的 CPU 不出现较大的时钟漂移，那么就能够保证 Lease 机制的有效性。

论文中实现 Lease 机制的方式是，Primary 定期向所有 Secondary 发送心跳来获取 Lease，而不是所有节点都向某个中心化组件获取 Lease。这样的好处是分散了压力，不会出现中心化组件故障而导致所有节点失去 Lease 的情况。

## Query

Query 流程比较简单，Query 只能发送给 Primary，Primary 根据最新 commit 的数据，返回对应的值。由于算法要求满足 Primary Invariant，所以 Query 总是能读到最新 commit 的数据。

## Update

Update 流程如下：

1.  Primary 分配一个 Serial Number(简称 sn) 给一个 UpdateRequest。
2.  Primary 将这个 UpdateRequest 加入自己的 Prepared List，同时向所有 Secondary 发送 Prepare 请求，要求将这个 UpdateRequest 加入 Prepared List。
3.  当所有 Replica 都完成了 Prepare，即所有 Replica 的 Prepared List 中都包含了该 Update 请求时，Primary 开始 Commit 这个请求，即将这个 UpdateRequest 放入 Committed List 中，同时 Apply 这个 Update。需要注意的是，同一个 Replica 上，Committed List 永远是 Prepared List 的前缀，所以 Primary 实际上是提高 Committed Point，把这个 Update Request 包含进来。
4.  返回客户端，Update 操作成功。

当下一次 Primary 向 Secondary 发送请求时，会带上 Primary 当前的 Committed Point，此时 Secondary 才会提高自己的 Committed Point。

从 Update 流程我们可以得出以下不变式：

## Commited Invariant

我们把某一个 Secondary 的 Committed List 记为 SecondaryCommittedList，其 Prepared List 记为 SecondaryPreparedList，把 Primary 的 Committed List 记为 PrimaryCommittedList。

**Commited Invariant**：SecondaryCommittedList 一定是 PrimaryCommittedList 的前缀，PrimaryCommittedList 一定是 SecondaryPreparedList 的前缀。

## Reconfiguration：Secondary 故障，Primary 故障，新加节点

## 1. Secondary 故障

当一个 Secondary 故障时，Primary 向 Configuration Manager 发起 Reconfiguration，将故障节点从 Replica Group 中删除。一旦移除这个 Replica，它就不属于这个 Replica Group 了，所有请求都不会再发给它。

假设某个 Primary 和 Secondary 发生了网络分区，但是都可以连接 Configuration Manager。这时候 Primary 会检测到 Secondary 没有响应了，Secondary 也会检测到 Primary 没有响应。此时两者都会试图发起 Reconfiguration，将对方从 Replica Group 中移除，这里的策略是 First Win 的原则，谁先到 Configuration Manager 中更改成功，谁就留在 Replica Group 里，而另外一个已经不属于 Replica Group 了，也就无法再更新 Configuration 了。由于 Primary 会向 Secondary 请求一个 Lease，在 Lease 有效期内 Secondary 不会执行 Reconfiguration，而 Primary 的探测间隔必然是小于 Lease 时间的，所以我认为这种情况下总是倾向于 Primary 先进行 Reconfiguration，将 Secondary 剔除。

## 2. Primary 故障

当一个 Primary 故障时，Secondary 会收不到 Primary 的心跳，如果超过 Lease 的时间，那么 Secondary 就会发起 Reconfiguration，将 Primary 剔除，这里也是 First Win 的原则，哪个 Secondary 先成功，就会变成 Primary。

当一个 Secondary 变成 Primary 后，需要先经过一个叫做 Reconciliation 的阶段才能提供服务。由于上述的 Commited Invariant，所以原先的 Primary 的 Committed List 一定是新的 Primary 的 Prepared List 的前缀，那么我们将新的 Primary 的 Prepared List 中的内容与当前 Replica Group 中的其他节点对齐，相当于把该节点上未 Commit 的记录在所有节点上再 Commit 一次，那么就一定包含之前所有的 Commit 记录。即以下不变式：

**Reconfiguration Invariant**：当一个新的 Primary 在 T 时刻完成 Reconciliation 时，那么 T 时刻之前任何节点 (包括原 Primary) 的 Commited List 都是新 Primary 当前 Commited List 的前缀。

Reconfiguration Invariant 表明了已经 Commit 的数据在 Reconfiguration 过程中不会丢。

## 3. 新加节点

新加的节点需要先成为 Secondary Candidate，这时候 Primary 就开始向其发送 Prepare 请求，此时这个节点还会追之前未同步过来的记录，一旦追平，就申请成为一个 Secondary，然后 Primary 向 Configuration Manager 发起配置变更，将这个节点加入 Replica Group。

还有一种情况时，如果一个节点曾经在 Replica Group 中，由于临时发生故障被移除，现在需要重新加回来。此时这个节点上的 Commited List 中的数据肯定是已经被 Commit 的了，但是 Prepared List 中的数据未必被 Commit，所以应该将未 Commit 的数据移除，从 Committed Point 开始向 Primary 请求数据。

## 算法总结

PacificA 是一个读写都满足强一致性的算法，它把数据的一致性与配置 (Configuration) 的一致性分开，使用额外的一致性组件 (Configuration Manager) 维护配置的一致性，在数据的可用副本数少于半数时，仍可以写入新数据并保证强一致性。

ES 在设计上参考了 PacificA 算法，其通过 Master 维护 Index 的 Meta，类似于论文中的 Configuration Manager 维护 Configuration。其 IndexMeta 中的 InSyncAllocationIds 代表了当前可用的 Shard，类似于论文中维护 Replica Group。下一节我们会介绍 ES 中的 SequenceNumber 和 Checkpoint，这两个类似于 PacificA 算法中的 Serial Number 和 Committed Point，在这一节之后，会再有一节来比较 ES 的实现与 PacificA 的异同。

## SequenceNumber、Checkpoint 与故障恢复

上面介绍了 ES 的一致性算法模型 PacificA，该算法很重要的一点是每个 Update 操作都会有一个对应的 Serial Number，表示执行的顺序。在之前的 ES 版本中，每个写入操作并没有类似 Serial Number 的东西，所以很多事情做不了。在 15 年的时候，ES 官方开始规划给每个写操作加入 SequenceNumber，并设想了很多应用场景。具体信息可以参考以下两个链接：

[Add Sequence Numbers to write operations #10708](https://link.zhihu.com/?target=https%3A//github.com/elastic/elasticsearch/issues/10708)

[Sequence IDs: Coming Soon to an Elasticsearch Cluster Near You](https://link.zhihu.com/?target=https%3A//www.elastic.co/blog/elasticsearch-sequence-ids-6-0)

下面我们简单介绍一下 Sequence、Checkpoint 是什么，以及其应用场景。

## Term 和 SequenceNumber

每个写操作都会分配两个值，Term 和 SequenceNumber。Term 在每次 Primary 变更时都会加 1，类似于 PacificA 论文中的 Configuration Version。SequenceNumber 在每次操作后加 1，类似于 PacificA 论文中的 Serial Number。

由于写请求总是发给 Primary，所以 Term 和 SequenceNumber 会由 Primary 分配，在向 Replica 发送同步请求时，会带上这两个值。

## LocalCheckpoint 和 GlobalCheckpoint

LocalCheckpoint 代表本 Shard 中所有小于该值的请求都已经处理完毕。

GlobalCheckpoint 代表所有小于该值的请求在所有的 Replica 上都处理完毕。GlobalCheckpoint 会由 Primary 进行维护，每个 Replica 会向 Primary 汇报自己的 LocalCheckpoint，Primary 根据这些信息来提升 GlobalCheckpoint。

GlobalCheckpoint 是一个全局的安全位置，代表其前面的请求都被所有 Replica 正确处理了，可以应用在节点故障恢复后的数据回补。另一方面，GlobalCheckpoint 也可以用于 Translog 的 GC，因为之前的操作记录可以不保存了。不过 ES 中 Translog 的 GC 策略是按照大小或者时间，好像并没有使用 GlobalCheckpoint。

## 快速故障恢复

当一个 Replica 故障时，ES 会将其移除，当故障超过一定时间，ES 会分配一个新的 Replica 到新的 Node 上，此时需要全量同步数据。但是如果之前故障的 Replica 回来了，就可以只回补故障之后的数据，追平后加回来即可，实现快速故障恢复。实现快速故障恢复的条件有两个，一个是能够保存故障期间所有的操作以及其顺序，另一个是能够知道从哪个点开始同步数据。第一个条件可以通过保存一定时间的 Translog 实现，第二个条件可以通过 Checkpoint 实现，所以就能够实现快速的故障恢复。这是 SequenceNumber 和 Checkpoint 的第一个重要应用场景。

## ES 与 PacificA 的比较

## 相同点

1.  Meta 一致性和 Data 一致性分开处理：PacificA 中通过 Configuration Manager 维护 Configuration 的一致性，ES 中通过 Master 维护 Meta 的一致性。
2.  维护同步中的副本集合：PacificA 中维护 Replica Group，ES 中维护 InSyncAllocationIds。
3.  SequenceNumber：在 PacificA 和 ES 中，写操作都具有 SequenceNumber，记录操作顺序。

## 不同点

不同点主要体现在 ES 虽然遵循 PacificA，但是目前其实现还有很多地方不满足算法要求，所以不能保证严格的强一致性。主要有以下几点：

1.  Meta 一致性：上一篇中分析了 ES 中 Meta 一致性的问题，可以看到 ES 并不能完全保证 Meta 一致性，因此也必然无法严格保证 Data 的一致性。
2.  Prepare 阶段：PacificA 中有 Prepare 阶段，保证数据在所有节点 Prepare 成功后才能 Commit，保证 Commit 的数据不丢，ES 中没有这个阶段，数据会直接写入。
3.  读一致性：ES 中所有 InSync 的 Replica 都可读，提高了读能力，但是可能读到旧数据。另一方面是即使只能读 Primary，ES 也需要 Lease 机制等避免读到 Old Primary。因为 ES 本身是近实时系统，所以读一致性要求可能并不严格。

## 小结

本文分析了 ES 中数据流的一致性问题，可以看到 ES 最近几年在这一块有很多进展，但也存在许多问题。本文是 Elasticsearch 分布式一致性原理剖析的最后一篇文章，该系列文章是对 ES 的一个调研分析总结，逐步分析了 ES 中的节点发现、Master 选举、Meta 一致性、Data 一致性等，对能够读完该系列文章的同学说一声感谢，期待与大家的交流。

## Reference

[Index API | Elasticsearch Reference [6.2]](https://link.zhihu.com/?target=https%3A//www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html)

[Reading and Writing documents | Elasticsearch Reference [6.2]](https://link.zhihu.com/?target=https%3A//www.elastic.co/guide/en/elasticsearch/reference/current/docs-replication.html)

[PacificA: Replication in Log-Based Distributed Storage Systems](https://link.zhihu.com/?target=https%3A//www.microsoft.com/en-us/research/publication/pacifica-replication-in-log-based-distributed-storage-systems/)

[Add Sequence Numbers to write operations #10708](https://link.zhihu.com/?target=https%3A//github.com/elastic/elasticsearch/issues/10708)

[Sequence IDs: Coming Soon to an Elasticsearch Cluster Near You](https://link.zhihu.com/?target=https%3A//www.elastic.co/blog/elasticsearch-sequence-ids-6-0)