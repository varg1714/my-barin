## 前言

“Elasticsearch 分布式一致性原理剖析” 系列将会对 Elasticsearch 的分布式一致性原理进行详细的剖析，介绍其实现方式、原理以及其存在的问题等 (基于 6.2 版本)。

ES 目前是最流行的分布式搜索引擎系统，其使用 Lucene 作为单机存储引擎并提供强大的搜索查询能力。学习其搜索原理，则必须了解 Lucene，而学习 ES 的架构，就必须了解其分布式如何实现，而一致性是分布式系统的核心之一。

本篇将介绍 ES 的集群组成、节点发现与 Master 选举，错误检测与扩缩容相关的内容。ES 在处理节点发现与 Master 选举等方面没有选择 Zookeeper 等外部组件，而是自己实现的一套，本文会介绍 ES 的这套机制是如何工作的，存在什么问题。本文的主要内容如下：

1.  ES 集群构成
2.  节点发现
3.  Master 选举
4.  错误检测
5.  集群扩缩容
6.  与 Zookeeper、raft 等实现方式的比较
7.  小结

## ES 集群构成

首先，一个 Elasticsearch 集群 (下面简称 ES 集群) 是由许多节点 (Node) 构成的，Node 可以有不同的类型，通过以下配置，可以产生四种不同类型的 Node：

```
conf/elasticsearch.yml:
    node.master: true/false
    node.data: true/false
```

四种不同类型的 Node 是一个 node.master 和 node.data 的 true/false 的两两组合。当然还有其他类型的 Node，比如 IngestNode(用于数据预处理等)，不在本文讨论范围内。

当 node.master 为 true 时，其表示这个 node 是一个 master 的候选节点，可以参与选举，在 ES 的文档中常被称作 master-eligible node，类似于 MasterCandidate。ES 正常运行时只能有一个 master(即 leader)，多于 1 个时会发生脑裂。

当 node.data 为 true 时，这个节点作为一个数据节点，会存储分配在该 node 上的 shard 的数据并负责这些 shard 的写入、查询等。

此外，任何一个集群内的 node 都可以执行任何请求，其会负责将请求转发给对应的 node 进行处理，所以当 node.master 和 node.data 都为 false 时，这个节点可以作为一个类似 proxy 的节点，接受请求并进行转发、结果聚合等。

![](https://pic3.zhimg.com/v2-a8f245e81fea7c513b3b59f44b940db6_r.jpg)

上图是一个 ES 集群的示意图，其中 NodeA 是当前集群的 Master，NodeB 和 NodeC 是 Master 的候选节点，其中 NodeA 和 NodeB 同时也是数据节点 (DataNode)，此外，NodeD 是一个单纯的数据节点，Node_E 是一个 proxy 节点。每个 Node 会跟其他所有 Node 建立连接。

到这里，我们提一个问题，供读者思考：一个 ES 集群应当配置多少个 master-eligible node，当集群的存储或者计算资源不足，需要扩容时，新扩上去的节点应该设置为何种类型？

## 节点发现

ZenDiscovery 是 ES 自己实现的一套用于节点发现和选主等功能的模块，没有依赖 Zookeeper 等工具，官方文档：

[https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery-zen.html](https://link.zhihu.com/?target=https%3A//www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery-zen.html)

简单来说，节点发现依赖以下配置：

```
conf/elasticsearch.yml:
    discovery.zen.ping.unicast.hosts: [1.1.1.1, 1.1.1.2, 1.1.1.3]
```

这个配置可以看作是，在本节点到每个 hosts 中的节点建立一条边，当整个集群所有的 node 形成一个联通图时，所有节点都可以知道集群中有哪些节点，不会形成孤岛。

官方推荐这里设置为所有的 master-eligible node，读者可以想想这样有何好处：

```
It is recommended that the unicast hosts list be maintained as the list of master-eligible nodes in the cluster.
```

## Master 选举

上面提到，集群中可能会有多个 master-eligible node，此时就要进行 master 选举，保证只有一个当选 master。如果有多个 node 当选为 master，则集群会出现脑裂，脑裂会破坏数据的一致性，导致集群行为不可控，产生各种非预期的影响。

为了避免产生脑裂，ES 采用了常见的分布式系统思路，保证选举出的 master 被多数派 (quorum) 的 master-eligible node 认可，以此来保证只有一个 master。这个 quorum 通过以下配置进行配置：

```
conf/elasticsearch.yml:
    discovery.zen.minimum_master_nodes: 2
```

这个配置对于整个集群非常重要。

## 1 master 选举谁发起，什么时候发起？

master 选举当然是由 master-eligible 节点发起，当一个 master-eligible 节点发现满足以下条件时发起选举：

1.  该 master-eligible 节点的当前状态不是 master。
2.  该 master-eligible 节点通过 ZenDiscovery 模块的 ping 操作询问其已知的集群其他节点，没有任何节点连接到 master。
3.  包括本节点在内，当前已有超过 minimum_master_nodes 个节点没有连接到 master。

总结一句话，即当一个节点发现包括自己在内的多数派的 master-eligible 节点认为集群没有 master 时，就可以发起 master 选举。

## 2 当需要选举 master 时，选举谁？

首先是选举谁的问题，如下面源码所示，选举的是排序后的第一个 MasterCandidate(即 master-eligible node)。

```
public MasterCandidate electMaster(Collection<MasterCandidate> candidates) {
        assert hasEnoughCandidates(candidates);
        List<MasterCandidate> sortedCandidates = new ArrayList<>(candidates);
        sortedCandidates.sort(MasterCandidate::compare);
        return sortedCandidates.get(0);
    }
```

那么是按照什么排序的？

```
public static int compare(MasterCandidate c1, MasterCandidate c2) {
    // we explicitly swap c1 and c2 here. the code expects "better" is lower in a sorted
    // list, so if c2 has a higher cluster state version, it needs to come first.
    int ret = Long.compare(c2.clusterStateVersion, c1.clusterStateVersion);
    if (ret == 0) {
        ret = compareNodes(c1.getNode(), c2.getNode());
    }
    return ret;
}
```

如上面源码所示，先根据节点的 clusterStateVersion 比较，clusterStateVersion 越大，优先级越高。clusterStateVersion 相同时，进入 compareNodes，其内部按照节点的 Id 比较 (Id 为节点第一次启动时随机生成)。

总结一下：

1.  当 clusterStateVersion 越大，优先级越高。这是为了保证新 Master 拥有最新的 clusterState(即集群的 meta)，避免已经 commit 的 meta 变更丢失。因为 Master 当选后，就会以这个版本的 clusterState 为基础进行更新。(一个例外是集群全部重启，所有节点都没有 meta，需要先选出一个 master，然后 master 再通过持久化的数据进行 meta 恢复，再进行 meta 同步)。
2.  当 clusterStateVersion 相同时，节点的 Id 越小，优先级越高。即总是倾向于选择 Id 小的 Node，这个 Id 是节点第一次启动时生成的一个随机字符串。之所以这么设计，应该是为了让选举结果尽可能稳定，不要出现都想当 master 而选不出来的情况。

## 3 什么时候选举成功？

当一个 master-eligible node(我们假设为 Node_A) 发起一次选举时，它会按照上述排序策略选出一个它认为的 master。

*   假设 Node_A 选 Node_B 当 Master：

Node_A 会向 Node_B 发送 join 请求，那么此时：

(1) 如果 Node_B 已经成为 Master，Node_B 就会把 Node_A 加入到集群中，然后发布最新的 cluster_state, 最新的 cluster_state 就会包含 Node_A 的信息。相当于一次正常情况的新节点加入。对于 Node_A，等新的 cluster_state 发布到 Node_A 的时候，Node_A 也就完成 join 了。  
(2) 如果 Node_B 在竞选 Master，那么 Node_B 会把这次 join 当作一张选票。对于这种情况，Node_A 会等待一段时间，看 Node_B 是否能成为真正的 Master，直到超时或者有别的 Master 选成功。  
(3) 如果 Node_B 认为自己不是 Master(现在不是，将来也选不上)，那么 Node_B 会拒绝这次 join。对于这种情况，Node_A 会开启下一轮选举。

*   假设 Node_A 选自己当 Master：

此时 NodeA 会等别的 node 来 join，即等待别的 node 的选票，当收集到超过半数的选票时，认为自己成为 master，然后变更 cluster_state 中的 master node 为自己，并向集群发布这一消息。

有兴趣的同学可以看看下面这段源码：

```
if (transportService.getLocalNode().equals(masterNode)) {
            final int requiredJoins = Math.max(0, electMaster.minimumMasterNodes() - 1); // we count as one
            logger.debug("elected as master, waiting for incoming joins ([{}] needed)", requiredJoins);
            nodeJoinController.waitToBeElectedAsMaster(requiredJoins, masterElectionWaitForJoinsTimeout,
                    new NodeJoinController.ElectionCallback() {
                        @Override
                        public void onElectedAsMaster(ClusterState state) {
                            synchronized (stateMutex) {
                                joinThreadControl.markThreadAsDone(currentThread);
                            }
                        }

                        @Override
                        public void onFailure(Throwable t) {
                            logger.trace("failed while waiting for nodes to join, rejoining", t);
                            synchronized (stateMutex) {
                                joinThreadControl.markThreadAsDoneAndStartNew(currentThread);
                            }
                        }
                    }

            );
        } else {
            // process any incoming joins (they will fail because we are not the master)
            nodeJoinController.stopElectionContext(masterNode + " elected");

            // send join request
            final boolean success = joinElectedMaster(masterNode);

            synchronized (stateMutex) {
                if (success) {
                    DiscoveryNode currentMasterNode = this.clusterState().getNodes().getMasterNode();
                    if (currentMasterNode == null) {
                        // Post 1.3.0, the master should publish a new cluster state before acking our join request. we now should have
                        // a valid master.
                        logger.debug("no master node is set, despite of join request completing. retrying pings.");
                        joinThreadControl.markThreadAsDoneAndStartNew(currentThread);
                    } else if (currentMasterNode.equals(masterNode) == false) {
                        // update cluster state
                        joinThreadControl.stopRunningThreadAndRejoin("master_switched_while_finalizing_join");
                    }

                    joinThreadControl.markThreadAsDone(currentThread);
                } else {
                    // failed to join. Try again...
                    joinThreadControl.markThreadAsDoneAndStartNew(currentThread);
                }
            }
        }
```

按照上述流程，我们描述一个简单的场景来帮助大家理解：

假如集群中有 3 个 master-eligible node，分别为 Node_A、 Node_B、 Node_C, 选举优先级也分别为 Node_A、Node_B、Node_C。三个 node 都认为当前没有 master，于是都各自发起选举，选举结果都为 Node_A(因为选举时按照优先级排序，如上文所述)。于是 Node_A 开始等 join(选票)，Node_B、Node_C 都向 Node_A 发送 join，当 Node_A 接收到一次 join 时，加上它自己的一票，就获得了两票了 (超过半数)，于是 Node_A 成为 Master。此时 cluster_state(集群状态) 中包含两个节点，当 Node_A 再收到另一个节点的 join 时，cluster_state 包含全部三个节点。

## 4 选举怎么保证不脑裂？

基本原则还是多数派的策略，如果必须得到多数派的认可才能成为 Master，那么显然不可能有两个 Master 都得到多数派的认可。

上述流程中，master 候选人需要等待多数派节点进行 join 后才能真正成为 master，就是为了保证这个 master 得到了多数派的认可。但是我这里想说的是，上述流程在绝大部份场景下没问题，听上去也非常合理，但是却是有 bug 的。

因为上述流程并没有限制在选举过程中，一个 Node 只能投一票，那么什么场景下会投两票呢？比如 NodeB 投 NodeA 一票，但是 NodeA 迟迟不成为 Master，NodeB 等不及了发起了下一轮选主，这时候发现集群里多了个 Node0，Node0 优先级比 NodeA 还高，那 NodeB 肯定就改投 Node0 了。假设 Node0 和 NodeA 都处在等选票的环节，那显然这时候 NodeB 其实发挥了两票的作用，而且投给了不同的人。

那么这种问题应该怎么解决呢，比如 raft 算法中就引入了选举周期 (term) 的概念，保证了每个选举周期中每个成员只能投一票，如果需要再投就会进入下一个选举周期，term+1。假如最后出现两个节点都认为自己是 master，那么肯定有一个 term 要大于另一个的 term，而且因为两个 term 都收集到了多数派的选票，所以多数节点的 term 是较大的那个，保证了 term 小的 master 不可能 commit 任何状态变更(commit 需要多数派节点先持久化日志成功，由于有 term 检测，不可能达到多数派持久化条件)。这就保证了集群的状态变更总是一致的。

而 ES 目前 (6.2 版本) 并没有解决这个问题，构造类似场景的测试 case 可以看到会选出两个 master，两个 node 都认为自己是 master，向全集群发布状态变更，这个发布也是两阶段的，先保证多数派节点 “接受” 这次变更，然后再要求全部节点 commit 这次变更。很不幸，目前两个 master 可能都完成第一个阶段，进入 commit 阶段，导致节点间状态出现不一致，而在 raft 中这是不可能的。那么为什么都能完成第一个阶段呢，因为第一个阶段 ES 只是将新的 cluster_state 做简单的检查后放入内存队列，如果当前 cluster_state 的 master 为空，不会对新的 clusterstate 中的 master 做检查，即在接受了 NodeA 成为 master 的 cluster_state 后(还未 commit)，还可以继续接受 NodeB 成为 master 的 cluster_state。这就使 NodeA 和 NodeB 都能达到 commit 条件，发起 commit 命令，从而将集群状态引向不一致。当然，这种脑裂很快会自动恢复，因为不一致发生后某个 master 再次发布 cluster_state 时就会发现无法达到多数派条件，或者是发现它的 follower 并不构成多数派而自动降级为 candidate 等。

这里要表达的是，ES 的 ZenDiscovery 模块与成熟的一致性方案相比，在某些特殊场景下存在缺陷，下一篇文章讲 ES 的 meta 变更流程时也会分析其他的 ES 无法满足一致性的场景。

## 错误检测

## 1. MasterFaultDetection 与 NodesFaultDetection

这里的错误检测可以理解为类似心跳的机制，有两类错误检测，一类是 Master 定期检测集群内其他的 Node，另一类是集群内其他的 Node 定期检测当前集群的 Master。检查的方法就是定期执行 ping 请求。ES 文档：

```
There are two fault detection processes running. The first is by the master, to ping all the other nodes in the cluster and verify that they are alive. And on the other end, each node pings to master to verify if its still alive or an election process needs to be initiated.
```

如果 Master 检测到某个 Node 连不上了，会执行 removeNode 的操作，将节点从 cluste_state 中移除，并发布新的 cluster_state。当各个模块 apply 新的 cluster_state 时，就会执行一些恢复操作，比如选择新的 primaryShard 或者 replica，执行数据复制等。

如果某个 Node 发现 Master 连不上了，会清空 pending 在内存中还未 commit 的 new cluster_state，然后发起 rejoin，重新加入集群 (如果达到选举条件则触发新 master 选举)。

## 2. rejoin

除了上述两种情况，还有一种情况是 Master 发现自己已经不满足多数派条件 (>=minimumMasterNodes) 了，需要主动退出 master 状态 (退出 master 状态并执行 rejoin) 以避免脑裂的发生，那么 master 如何发现自己需要 rejoin 呢？

*   上面提到，当有节点连不上时，会执行 removeNode。在执行 removeNode 时判断剩余的 Node 是否满足多数派条件，如果不满足，则执行 rejoin。

```
if (electMasterService.hasEnoughMasterNodes(remainingNodesClusterState.nodes()) == false) {
                final int masterNodes = electMasterService.countMasterNodes(remainingNodesClusterState.nodes());
                rejoin.accept(LoggerMessageFormat.format("not enough master nodes (has [{}], but needed [{}])",
                                                         masterNodes, electMasterService.minimumMasterNodes()));
                return resultBuilder.build(currentState);
            } else {
                return resultBuilder.build(allocationService.deassociateDeadNodes(remainingNodesClusterState, true, describeTasks(tasks)));
            }
```

*   在 publish 新的 cluster_state 时，分为 send 阶段和 commit 阶段，send 阶段要求多数派必须成功，然后再进行 commit。如果在 send 阶段没有实现多数派返回成功，那么可能是有了新的 master 或者是无法连接到多数派个节点等，则 master 需要执行 rejoin。

```
try {
            publishClusterState.publish(clusterChangedEvent, electMaster.minimumMasterNodes(), ackListener);
        } catch (FailedToCommitClusterStateException t) {
            // cluster service logs a WARN message
            logger.debug("failed to publish cluster state version [{}] (not enough nodes acknowledged, min master nodes [{}])",
                newState.version(), electMaster.minimumMasterNodes());

            synchronized (stateMutex) {
                pendingStatesQueue.failAllStatesAndClear(
                    new ElasticsearchException("failed to publish cluster state"));

                rejoin("zen-disco-failed-to-publish");
            }
            throw t;
        }
```

*   在对其他节点进行定期的 ping 时，发现有其他节点也是 master，此时会比较本节点与另一个 master 节点的 cluster_state 的 version，谁的 version 大谁成为 master，version 小的执行 rejoin。

```
if (otherClusterStateVersion > localClusterState.version()) {
            rejoin("zen-disco-discovered another master with a new cluster_state [" + otherMaster + "][" + reason + "]");
        } else {
            // TODO: do this outside mutex
            logger.warn("discovered [{}] which is also master but with an older cluster_state, telling [{}] to rejoin the cluster ([{}])", otherMaster, otherMaster, reason);
            try {
                // make sure we're connected to this node (connect to node does nothing if we're already connected)
                // since the network connections are asymmetric, it may be that we received a state but have disconnected from the node
                // in the past (after a master failure, for example)
                transportService.connectToNode(otherMaster);
                transportService.sendRequest(otherMaster, DISCOVERY_REJOIN_ACTION_NAME, new RejoinClusterRequest(localClusterState.nodes().getLocalNodeId()), new EmptyTransportResponseHandler(ThreadPool.Names.SAME) {

                    @Override
                    public void handleException(TransportException exp) {
                        logger.warn((Supplier<?>) () -> new ParameterizedMessage("failed to send rejoin request to [{}]", otherMaster), exp);
                    }
                });
            } catch (Exception e) {
                logger.warn((Supplier<?>) () -> new ParameterizedMessage("failed to send rejoin request to [{}]", otherMaster), e);
            }
        }
```

## 集群扩缩容

上面讲了节点发现、Master 选举、错误检测等机制，那么现在我们可以来看一下如何对集群进行扩缩容。

## 1 扩容 DataNode

假设一个 ES 集群存储或者计算资源不够了，我们需要进行扩容，这里我们只针对 DataNode，即配置为：

```
conf/elasticsearch.yml:
    node.master: false
    node.data: true
```

然后需要配置集群名、节点名等其他配置，为了让该节点能够加入集群，我们把 discovery.zen.ping.unicast.hosts 配置为集群中的 master-eligible node。

```
conf/elasticsearch.yml:
    cluster.name: es-cluster
    node.name: node_Z
    discovery.zen.ping.unicast.hosts: ["x.x.x.x", "x.x.x.y", "x.x.x.z"]
```

然后启动节点，节点会自动加入到集群中，集群会自动进行 rebalance，或者通过 reroute api 进行手动操作。

[https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-reroute.html](https://link.zhihu.com/?target=https%3A//www.elastic.co/guide/en/elasticsearch/reference/current/cluster-reroute.html)

[https://www.elastic.co/guide/en/elasticsearch/reference/current/shards-allocation.html](https://link.zhihu.com/?target=https%3A//www.elastic.co/guide/en/elasticsearch/reference/current/shards-allocation.html)

## 2 缩容 DataNode

假设一个 ES 集群使用的机器数太多了，需要缩容，我们怎么安全的操作来保证数据安全，并且不影响可用性呢？

首先，我们选择需要缩容的节点，注意本节只针对 DataNode 的缩容，MasterNode 缩容涉及到更复杂的问题，下面再讲。

然后，我们需要把这个 Node 上的 Shards 迁移到其他节点上，方法是先设置 allocation 规则，禁止分配 Shard 到要缩容的机器上，然后让集群进行 rebalance。

```
PUT _cluster/settings
{
  "transient" : {
    "cluster.routing.allocation.exclude._ip" : "10.0.0.1"
  }
}
```

等这个节点上的数据全部迁移完成后，节点可以安全下线。

更详细的操作方式可以参考官方文档：

[https://www.elastic.co/guide/en/elasticsearch/reference/current/allocation-filtering.html](https://link.zhihu.com/?target=https%3A//www.elastic.co/guide/en/elasticsearch/reference/current/allocation-filtering.html)

## 3 扩容 MasterNode

假如我们想扩容一个 MasterNode(master-eligible node)， 那么有个需要考虑的问题是，上面提到为了避免脑裂，ES 是采用多数派的策略，需要配置一个 quorum 数：

```
conf/elasticsearch.yml:
    discovery.zen.minimum_master_nodes: 2
```

假设之前 3 个 master-eligible node，我们可以配置 quorum 为 2，如果扩容到 4 个 master-eligible node，那么 quorum 就要提高到 3。

所以我们应该先把 discovery.zen.minimum_master_nodes 这个配置改成 3，再扩容 master，更改这个配置可以通过 API 的方式：

```
curl -XPUT localhost:9200/_cluster/settings -d '{
    "persistent" : {
        "discovery.zen.minimum_master_nodes" : 3
    }
}'
```

这个 API 发送给当前集群的 master，然后新的值立即生效，然后 master 会把这个配置持久化到 cluster meta 中，之后所有节点都会以这个配置为准。

但是这种方式有个问题在于，配置文件中配置的值和 cluster meta 中的值很可能出现不一致，不一致很容易导致一些奇怪的问题，比如说集群重启后，在恢复 cluster meta 前就需要进行 master 选举，此时只可能拿配置中的值，拿不到 cluster meta 中的值，但是 cluster meta 恢复后，又需要以 cluster meta 中的值为准，这中间肯定存在一些正确性相关的边界 case。

总之，动 master 节点以及相关的配置一定要谨慎，master 配置错误很有可能导致脑裂甚至数据写坏、数据丢失等场景。

## 4 缩容 MasterNode

缩容 MasterNode 与扩容跟扩容是相反的流程，我们需要先把节点缩下来，再把 quorum 数调下来，不再详细描述。

## 与 Zookeeper、raft 等实现方式的比较

## 1. 与使用 Zookeeper 相比

本篇讲了 ES 集群中节点相关的几大功能的实现方式：

1.  节点发现
2.  Master 选举
3.  错误检测
4.  集群扩缩容

试想下，如果我们使用 Zookeeper 来实现这几个功能，会带来哪些变化？

## Zookeeper 介绍

我们首先介绍一下 Zookeeper，熟悉的同学可以略过。

Zookeeper 分布式服务框架是 Apache Hadoop 的一个子项目，它主要是用来解决分布式应用中经常遇到的一些数据管理问题，如：统一命名服务、状态同步服务、集群管理、分布式应用配置项的管理等。

简单来说，Zookeeper 就是用于管理分布式系统中的节点、配置、状态，并完成各个节点间配置和状态的同步等。大量的分布式系统依赖 Zookeeper 或者是类似的组件。

Zookeeper 通过目录树的形式来管理数据，每个节点称为一个 znode，每个 znode 由 3 部分组成:

*   stat. 此为状态信息, 描述该 znode 的版本, 权限等信息.
*   data. 与该 znode 关联的数据.
*   children. 该 znode 下的子节点.

stat 中有一项是 ephemeralOwner，如果有值，代表是一个临时节点，临时节点会在 session 结束后删除，可以用来辅助应用进行 master 选举和错误检测。

Zookeeper 提供 watch 功能，可以用于监听相应的事件，比如某个 znode 下的子节点的增减，某个 znode 本身的增减，某个 znode 的更新等。

## 怎么使用 Zookeeper 实现 ES 的上述功能

1.  节点发现：每个节点的配置文件中配置一下 Zookeeper 服务器的地址，节点启动后到 Zookeeper 中某个目录中注册一个临时的 znode。当前集群的 master 监听这个目录的子节点增减的事件，当发现有新节点时，将新节点加入集群。
2.  master 选举：当一个 master-eligible node 启动时，都尝试到固定位置注册一个名为 master 的临时 znode，如果注册成功，即成为 master，如果注册失败则监听这个 znode 的变化。当 master 出现故障时，由于是临时 znode，会自动删除，这时集群中其他的 master-eligible node 就会尝试再次注册。使用 Zookeeper 后其实是把选 master 变成了抢 master。
3.  错误检测：由于节点的 znode 和 master 的 znode 都是临时 znode，如果节点故障，会与 Zookeeper 断开 session，znode 自动删除。集群的 master 只需要监听 znode 变更事件即可，如果 master 故障，其他的候选 master 则会监听到 master znode 被删除的事件，尝试成为新的 master。
4.  集群扩缩容：扩缩容将不再需要考虑 minimum_master_nodes 配置的问题，会变得更容易。

## 使用 Zookeeper 的优劣点

使用 Zookeeper 的好处是，把一些复杂的分布式一致性问题交给 Zookeeper 来做，ES 本身的逻辑就可以简化很多，正确性也有保证，这也是大部分分布式系统实践过的路子。而 ES 的这套 ZenDiscovery 机制经历过很多次 bug fix，到目前仍有一些边角的场景存在 bug，而且运维也不简单。

那为什么 ES 不使用 Zookeeper 呢，大概是官方开发觉得增加 Zookeeper 依赖后会多依赖一个组件，使集群部署变得更复杂，用户在运维时需要多运维一个 Zookeeper。

那么在自主实现这条路上，还有什么别的算法选择吗？当然有的，比如 raft。

## 2. 与使用 raft 相比

raft 算法是近几年很火的一个分布式一致性算法，其实现相比 paxos 简单，在各种分布式系统中也得到了应用。这里不再描述其算法的细节，我们单从 master 选举算法角度，比较一下 raft 与 ES 目前选举算法的异同点：

## 相同点

1.  多数派原则：必须得到超过半数的选票才能成为 master。
2.  选出的 leader 一定拥有最新已提交数据：在 raft 中，数据更新的节点不会给数据旧的节点投选票，而当选需要多数派的选票，则当选人一定有最新已提交数据。在 es 中，version 大的节点排序优先级高，同样用于保证这一点。

## 不同点

1.  正确性论证：raft 是一个被论证过正确性的算法，而 ES 的算法是一个没有经过论证的算法，只能在实践中发现问题，做 bug fix，这是我认为最大的不同。
2.  是否有选举周期 term：raft 引入了选举周期的概念，每轮选举 term 加 1，保证了在同一个 term 下每个参与人只能投 1 票。ES 在选举时没有 term 的概念，不能保证每轮每个节点只投一票。
3.  选举的倾向性：raft 中只要一个节点拥有最新的已提交的数据，则有机会选举成为 master。在 ES 中，version 相同时会按照 NodeId 排序，总是 NodeId 小的人优先级高。

## 看法

raft 从正确性上看肯定是更好的选择，而 ES 的选举算法经过几次 bug fix 也越来越像 raft。当然，在 ES 最早开发时还没有 raft，而未来 ES 如果继续沿着这个方向走很可能最终就变成一个 raft 实现。

raft 不仅仅是选举，下一篇介绍 meta 数据一致性时也会继续比较 ES 目前的实现与 raft 的异同。

## 小结

本篇介绍了 Elasticsearch 集群的组成、节点发现、master 选举、故障检测和扩缩容等方面的实现，与一般的文章不同，本文对其原理、存在的问题也进行了一些分析，并与其他实现方式进行了比较。

作为 Elasticsearch 分布式一致性原理剖析系列的第一篇，本文先从节点入手，下一篇会介绍 meta 数据变更的一致性问题，会在本文的基础上对 ES 的分布式原理做进一步分析。

最后，我们在招人，有兴趣的可以私信联系我。