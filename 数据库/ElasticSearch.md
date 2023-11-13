
# 1. ES 的架构

## 1.1. 数据划分

### 1.1.1. 数据组织形式

结构化数据：也称作行数据，是由二维表结构来逻辑表达和实现的数据，严格地遵循数据格式与长度规范，主要通过关系型数据库进行存储和管理。指具有固定格式或有限长度的数据，如数据库，元数据等。

非结构化数据：又可称为全文数据，不定长或无固定格式，不适于由数据库二维表来表现，包括所有格式的办公文档、XML、HTML、Word 文档，邮件，各类报表、图片和咅频、视频信息等。

对于结构化数据，因为它们具有特定的结构，所以我们一般都是可以通过关系型数据库（MySQL，Oracle 等）的二维表（Table）的方式存储和搜索，也可以建立索引。对于非结构化数据，我们一般使用全文搜索引擎进行存储，而 ES 就是一个全文搜索引擎。

### 1.1.2. 使用场景

ES 在全文检索、日志分析、监控分析等场景具有广泛应用。

#### 1.1.2.1. 日志实时分析

日志是互联网行业基础广泛的数据形式。典型日志有用来定位业务问题的运营日志，如慢日志、异常日志；用来分析用户行为的业务日志，如用户的点击、访问日志；以及安全行为分析的审计日志等。

Elastic 生态提供了完整的日志解决方案。通过简单部署，即可搭建一个完整的日志实时分析服务。ES 生态完美的解决了日志实时分析场景需求，这也是近几年 ES 快速发展的一个重要原因。

日志从产生到可访问一般在 10s 级，相比于传统大数据解决方案的几十分钟、小时级时效性非常高。

ES 底层支持倒排索引、列存储等数据结构，使得在日志场景可以利用 ES 非常灵活的搜索分析能力。通过 ES 交互式分析能力，即使在万亿级日志的情况下，日志搜索响应时间也是秒级。

日志处理的基本流程包含：日志采集 -> 数据清洗 -> 存储 -> 可视化分析。Elastic Stack 通过完整的日志解决方案，帮助用户完成对日志处理全链路管理。

![image.png](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202309110133197.png)

#### 1.1.2.2. 时序分析场景

时序数据是按时间顺序记录设备、系统状态变化的数据。典型的时序数据有传统的服务器监控指标数据、应用系统性能监控数据、智能硬件、工业物联网传感器数据等。

ES 提供灵活、多维度的统计分析能力，实现查看监控按照地域、业务模块等灵活的进行统计分析。另外，ES 支持列存储、高压缩比、副本数按需调整等能力，可实现较低存储成本。最后时序数据也可通过 Kibana 组件轻松实现可视化。

![image.png](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202309110135795.png)

#### 1.1.2.3. 搜索场景

搜索服务典型场景有像京东、拼多多、蘑菇街中的商品搜索；应用商店中的应用 APP 搜索；论坛、在线文档等站内搜索。

这类场景用户关注高性能、低延迟、高可靠、搜索质量等。如单个服务最大需达到 10w+ QPS，请求平均响应时间在 20ms 以内，查询毛刺低于 100ms，高可用如搜索场景通常要求 4 个 9 的可用性，支持单机房故障容灾等。

## 1.2. 倒排索引

非结构化数据的处理需要依赖全文搜索，而目前市场上开放源代码的最好全文检索引擎工具包就属于 Apache 的 Lucene 了。

但是 Lucene 只是一个工具包，它不是一个完整的全文检索引擎。Lucene 的目的是为软件开发人员提供一个简单易用的工具包，以方便的在目标系统中实现全文检索的功能，或者是以此为基础建立起完整的全文检索引擎。

目前以 Lucene 为基础建立的开源可用全文搜索引擎主要是 Solr 和 Elasticsearch。而 Lucene 能实现全文搜索主要是因为它实现了倒排索引的查询结构。

如何理解倒排索引呢？假如现有三份数据文档，文档的内容如下分别是：

```txt
Java is the best programming language.
PHP is the best programming language.
Javascript is the best programming language.
```

为了创建倒排索引，我们通过分词器将每个文档的内容域拆分成单独的词（我们称它为词条或 Term），创建一个包含所有不重复词条的排序列表，然后列出每个词条出现在哪个文档。这种结构由文档中所有不重复词的列表构成，对于其中每个词都有一个文档列表与之关联。这种由属性值来确定记录的位置的结构就是倒排索引。带有倒排索引的文件我们称为倒排文件。

![image.png](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202309101735990.png)

其中主要有如下几个核心术语需要理解：

- 词条（Term）
    索引里面最小的存储和查询单元，对于英文来说是一个单词，对于中文来说一般指分词后的一个词。
- 词典（Term Dictionary）
    字典，是词条 Term 的集合。搜索引擎的通常索引单位是单词，单词词典是由文档集合中出现过的所有单词构成的字符串集合，单词词典内每条索引项记载单词本身的一些信息以及指向“倒排列表”的指针。
- 倒排表（Post list）
    一个文档通常由多个词组成，倒排表记录的是某个词在哪些文档里出现过以及出现的位置。每条记录称为一个倒排项（Posting）。倒排表记录的不单是文档编号，还存储了词频等信息。
- 倒排文件（Inverted File）
    所有单词的倒排列表往往顺序地存储在磁盘的某个文件里，这个文件被称之为倒排文件，倒排文件是存储倒排索引的物理文件。

从上图我们可以了解到倒排索引主要由两个部分组成：

- 词典
- 倒排文件

词典和倒排表是 Lucene 中很重要的两种数据结构，是实现快速检索的重要基石。词典和倒排文件是分两部分存储的，词典在内存中而倒排文件存储在磁盘上。

## 1.3. 集群

ES 集群由一个或多个 Elasticsearch 节点组成，每个节点配置相同的 `cluster.name` 即可加入集群，默认值为 “elasticsearch”。需要确保不同的环境中使用不同的集群名称，否则最终会导致节点加入错误的集群。

一个 Elasticsearch 服务启动实例就是一个节点（Node）。节点通过 `node.name` 来设置节点名称，如果不设置则在启动时给节点分配一个随机通用唯一标识符作为名称。

### 1.3.1. 集群的发现

[Zen Discovery](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-discovery-zen.html) 是 ES 的内置默认发现模块（发现模块的职责是发现集群中的节点以及选举 Master 节点）。它提供单播和基于文件的发现，并且可以扩展为通过插件支持云环境和其他形式的发现。Zen Discovery 可与其他模块集成，例如，节点之间的所有通信都使用 Transport 模块完成。节点使用发现机制通过 Ping 的方式查找其他节点。

![](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202309110126695.png)

ES 默认被配置为使用单播发现，以防止节点无意中加入集群。只有在同一台机器上运行的节点才会自动组成集群。如果集群的节点运行在不同的机器上，可以为 ES 提供一些尝试连接的节点列表。当一个节点联系到单播列表中的成员时，它就会得到整个集群所有节点的状态，然后它会联系 Master 节点，并加入集群。这意味着**单播列表不需要包含集群中的所有节点，它只是需要足够的节点，当一个新节点联系上其中一个并且通信就可以了**。该列表可由 `discovery.zen.ping.unicast.hosts` 指定，官方推荐设置为所有的候选主节点。

节点启动后先 Ping ，如果 `discovery.zen.ping.unicast.hosts` 有设置，则 Ping 设置中的 Host ，否则尝试 ping localhost 的几个端口。ES 支持同一个主机启动多个节点，Ping 的 Response 会包含该节点的基本信息以及该节点认为的 Master 节点。

### 1.3.2. 节点类型

每个节点既可以是候选主节点也可以是数据节点，通过在配置文件 `../config/elasticsearch.yml` 中设置即可，默认都为 true。数据节点负责数据的存储和相关的操作，例如对数据进行增、删、改、查和聚合等操作，所以数据节点（Data 节点）对机器配置要求比较高，对 CPU、内存和 I/O 的消耗很大。

通常随着集群的扩大，需要增加更多的数据节点来提高性能和可用性。候选主节点可以被选举为主节点（Master 节点），**集群中只有候选主节点才有选举权和被选举权**，其他节点不参与选举的工作。

主节点负责创建索引、删除索引、跟踪哪些节点是群集的一部分，并决定哪些分片分配给相关的节点、追踪集群中节点的状态等，稳定的主节点对集群的健康是非常重要的。

![image.png](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202311100029080.png)

每个节点上都保存了集群状态，但只有主节点才能修改集群状态信息（如创建索引、决定分片分布等），集群状态（Cluster State)，维护了一个集群中必要的信息   ：
- 所有的节点信息
- 所有的索引和其相关的 Mapping 与 Setting 信息
- 分片的路由信息

所以如果某个节点既是数据节点又是主节点，那么主节点产生影响时可能会对从而对整个集群的状态产生影响。因此为了提高集群的健康性，我们应该对 Elasticsearch 集群中的节点做好角色上的划分和隔离。可以使用几个配置较低的机器群作为候选主节点群。

主节点和其他节点之间通过 Ping 的方式互检查，主节点负责 Ping 所有其他节点，判断是否有节点已经挂掉。其他节点也通过 Ping 的方式判断主节点是否处于可用状态。当节点故障后，会被主节点移出集群，并自动在其他节点上恢复故障节点上的分片。主分片故障时会提升其中一个副本分片为主分片。其他节点也会探活主节点，当主节点故障后，会触发内置的类 Raft 协议选主，并通过设置最少候选主节点数，避免集群脑裂。

![image.png](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202309110128550.png)

虽然对节点做了角色区分，但是**用户的请求可以发往任何一个节点，并由该节点负责分发请求、收集结果等操作，而不需要主节点转发**。这种节点可称之为协调节点，协调节点是不需要指定和配置的，集群中的任何节点都可以充当协调节点的角色。同时协调节点的存在使得请求可以发往任意节点，这提升了灵活性，但另一方面若该协调节点本身已经处于高负载状态，那么势必会降低转发的性能。

### 1.3.3. 节点选举

master 节点的选举过程如下：
1. 选举的时机
    由 master-eligible 节点发起，当一个 master-eligible 节点发现满足以下条件时发起选举：
    1.  该 master-eligible 节点的当前状态不是 master。
    2.  该 master-eligible 节点通过 ZenDiscovery 模块的 ping 操作询问其已知的集群其他节点，没有任何节点连接到 master。
    3.  包括本节点在内，当前已有超过 minimum_master_nodes 个节点没有连接到 master。
    
    即当一个节点发现包括自己在内的多数派的 master-eligible 节点认为集群没有 master 时，就可以发起 master 选举。
1. 选举的对象
    1. 节点的状态越新，优先级越高
        当 clusterStateVersion 越大，优先级越高。这是为了保证新 Master 拥有最新的 clusterState (即集群的 meta)，避免已经 commit 的 meta 变更丢失。
        
        因为 Master 当选后，就会以这个版本的 clusterState 为基础进行更新。(一个例外是集群全部重启，所有节点都没有 meta，需要先选出一个 master，然后 master 再通过持久化的数据进行 meta 恢复，再进行 meta 同步)。
    2. 当 clusterStateVersion 相同时，节点的 Id 越小，优先级越高。
        即总是倾向于选择 Id 小的 Node，这个 Id 是节点第一次启动时生成的一个随机字符串。之所以这么设计，应该是为了让选举结果尽可能稳定，不要出现都想当 master 而选不出来的情况。
1. 选举的流程
    当一个 master-eligible node (我们假设为 Node_A) 发起一次选举时，它会按照上述排序策略选出一个它认为的 master。
    
    - 假设 Node_A 选 Node_B 当 Master  
        Node_A 会向 Node_B 发送 join 请求，那么此时：
        1. 如果 Node_B 已经成为 Master  
            Node_B 就会把 Node_A 加入到集群中，然后发布最新的 cluster_state，最新的 cluster_state 就会包含 Node_A 的信息。
            
            这就相当于一次正常情况的新节点加入。对于 Node_A，等新的 cluster_state 发布到 Node_A 的时候，Node_A 也就完成 join 了。
        2. 如果 Node_B 在竞选 Master  
            此时 Node_B 会把这次 join 当作一张选票。对于这种情况，Node_A 会等待一段时间，看 Node_B 是否能成为真正的 Master，直到超时或者有别的 Master 选成功。
        3. 如果 Node_B 认为自己不是 Master (现在不是，将来也选不上)，那么 Node_B 会拒绝这次 join。对于这种情况，Node_A 会开启下一轮选举。
    - 假设 Node_A 选自己当 Master  
        此时 NodeA 会等别的 node 来 join，即等待别的 node 的选票，当收集到超过半数的选票时，认为自己成为 master，然后变更 cluster_state 中的 master node 为自己，并向集群发布这一消息。

这里有个限制条件就是 `discovery.zen.minimum_master_nodes`，如果**节点数达不到最小值的限制，则循环上述过程，直到节点数足够可以开始选举**。选举结果是肯定能选举出一个 Master ，如果只有一个 Local 节点那就选出的是自己。如果当前节点是 Master ，则开始等待节点数达到 `discovery.zen.minimum_master_nodes`，然后提供服务。如果当前节点不是 Master ，则尝试加入 Master 。ES 将以上服务发现以及选主的流程叫做 Zen Discovery 。

由于它支持任意数目的集群（ 1 - N ），所以不能像 Zookeeper 那样限制节点必须是奇数，也就无法用投票的机制来选主，而是通过一个规则。只要所有的节点都遵循同样的规则，得到的信息都是对等的，选出来的主节点肯定是一致的。但分布式系统的问题就出在信息不对等的情况，这时候很容易出现脑裂（Split-Brain）的问题。大多数解决方案就是设置一个 Quorum 值，要求**可用节点必须大于 Quorum（一般是超过半数节点），才能对外提供服务**。而 ES 中，这个 Quorum 的配置就是 `discovery.zen.minimum_master_nodes`。

### 1.3.4. 脑裂现象

如果由于网络或其他原因导致集群中选举出多个 Master 节点，使得数据更新时出现不一致，这种现象称之为脑裂，即集群中不同的节点对于 Master 的选择出现了分歧，出现了多个 Master 竞争。

Zen Discovery 流程中，master 候选人需要等待多数派节点进行 join 后才能真正成为 master，就是为了保证这个 master 得到了多数派的认可。上述流程在绝大部份场景下没问题，但是却是有 bug 的：**没有限制在选举过程中，一个 Node 只能投一票**。

那么什么场景下会投两票呢？比如 NodeB 投 NodeA 一票，但是 NodeA 迟迟不成为 Master，NodeB 等不及了发起了下一轮选主，这时候发现集群里多了个 Node0，Node0 优先级比 NodeA 还高，那 NodeB 肯定就改投 Node0 了。假设 Node0 和 NodeA 都处在等选票的环节，那显然这时候 NodeB 其实发挥了两票的作用，而且投给了不同的人。

> [!question] 如何解决多次投票的问题呢？
> 在 raft 算法中引入了选举周期 (term) 的概念，保证了每个选举周期中每个成员只能投一票，如果需要再投就会进入下一个选举周期，term+1。假如最后出现两个节点都认为自己是 master，那么肯定有一个 term 要大于另一个的 term，而且因为两个 term 都收集到了多数派的选票，所以多数节点的 term 是较大的那个，保证了 term 小的 master 不可能 commit 任何状态变更(commit 需要多数派节点先持久化日志成功，由于有 term 检测，不可能达到多数派持久化条件)。这就保证了集群的状态变更总是一致的。

“脑裂”问题可能有以下几个原因造成：

- 网络问题
    集群间的网络延迟导致一些节点访问不到 Master，认为 Master 挂掉了从而选举出新的 Master，并对 Master 上的分片和副本标红，分配新的主分片。
- 节点负载
    主节点的角色既为 Master 又为 Data，访问量较大时可能会导致 ES 停止响应（假死状态）造成大面积延迟，此时其他节点得不到主节点的响应认为主节点挂掉了，会重新选取主节点。
- 内存回收
    主节点的角色既为 Master 又为 Data，当 Data 节点上的 ES 进程占用的内存较大，引发 JVM 的大规模内存回收，造成 ES 进程失去响应。

为了避免脑裂现象的发生，我们可以从原因着手通过以下几个方面来做出优化措施：
- 适当调大响应时间，减少误判
    通过参数 `discovery.zen.ping_timeout` 设置节点状态的响应时间，默认为 3s，可以适当调大。
    
    如果 Master 在该响应时间的范围内没有做出响应应答，判断该节点已经挂掉了。调大参数（如 6s，`discovery.zen.ping_timeout:6`），可适当减少误判。
- 选举触发
    我们需要在候选集群中的节点的配置文件中设置参数 `discovery.zen.munimum_master_nodes` 的值。这个参数表示在选举主节点时需要参与选举的候选主节点的节点数，默认值是 1，官方建议取值 (`master_eligibel_nodes2)+1`，其中 `master_eligibel_nodes` 为候选主节点的个数。
    
    这样做既能防止脑裂现象的发生，也能最大限度地提升集群的高可用性，因为只要不少于 `discovery.zen.munimum_master_nodes` 个候选节点存活，选举工作就能正常进行。当小于这个值的时候，无法触发选举行为，集群无法使用，不会造成分片混乱的情况。
- 角色分离
    即候选主节点和数据节点进行角色分离，这样可以减轻主节点的负担，防止主节点的假死状态发生，减少对主节点“已死”的误判。

### 1.3.5. 节点监控

节点监控主要用于检测 master 节点及其他节点的状态，可以理解为类似心跳的机制。有两类错误检测，一类是 Master 定期检测集群内其他的 Node，另一类是集群内其他的 Node 定期检测当前集群的 Master。检查的方法就是定期执行 ping 请求。

> [!quote] ES 的错误检测
> 
> There are two fault detection processes running. The first is by the master, to ping all the other nodes in the cluster and verify that they are alive. And on the other end, each node pings to master to verify if its still alive or an election process needs to be initiated.

检测的方式如下：
1. Master 节点检测其他节点
    如果 Master 检测到某个 Node 连不上了，会执行 removeNode 的操作，将节点从 cluste_state 中移除，并发布新的 cluster_state。当各个模块 apply 新的 cluster_state 时，就会执行一些恢复操作，比如选择新的 primaryShard 或者 replica，执行数据复制等。
2. 其他节点检测 Master 节点
    如果某个 Node 发现 Master 连不上了，会清空 pending 在内存中还未 commit 的 new cluster_state，然后发起 rejoin，重新加入集群 (如果达到选举条件则触发新 master 选举)。
3. Master 节点检查自身
    若 Master 发现自己已经不满足多数派条件 (>=minimumMasterNodes) 了，需要主动退出 master 状态 (退出 master 状态并执行 rejoin) 以避免脑裂的发生，那么 master 如何发现自己需要 rejoin 呢？
    
    1. 移除其他节点后判断 minimumMasterNodes 条件
        当 Master 节点发现其他节点无法连接时，会执行 removeNode。在执行 removeNode 时判断剩余的 Node 是否满足多数派条件，如果不满足，则执行 rejoin。
    2. 发布集群状态时
        在 publish 新的 cluster_state 时，分为 send 阶段和 commit 阶段，send 阶段要求多数派必须成功，然后再进行 commit。如果在 send 阶段没有实现多数派返回成功，那么可能是有了新的 master 或者是无法连接到多数派个节点等，则 master 需要执行 rejoin。

### 1.3.6. 集群扩缩容

假设一个 ES 集群存储或者计算资源不够了，我们需要进行扩容。

#### 1.3.6.1. 数据节点

对于数据节点的扩容，只要配置数据节点信息将其加入集群即可。启动节点后，节点会自动加入到集群中，集群会自动进行 rebalance，或者通过 [reroute api](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-reroute.html) 进行手动操作。

对于数据节点的缩容，为了保证数据安全，我们需要把这个 Node 上的 Shards 迁移到其他节点上，方法是先设置 allocation [规则](https://www.elastic.co/guide/en/elasticsearch/reference/current/allocation-filtering.html)，禁止分配 Shard 到要缩容的机器上，然后让集群进行 rebalance。

````ad-example
title: 规则示例

```bash
PUT _cluster/settings
{
  "transient" : {
    "cluster.routing.allocation.exclude._ip" : "10.0.0.1"
  }
}
```

````

#### 1.3.6.2. 主节点

假如我们想扩容一个 MasterNode (master-eligible node)，那么有个需要考虑的问题是：为了避免脑裂，ES 是采用多数派的策略，需要配置一个 quorum 数。因此在扩容主节点时，需要先提升 quorum 数量，然后再进行集群的扩容操作。缩容与之相似。

### 1.3.7. 集群元数据管理

#### 1.3.7.1. 元数据管理

要管理集群，那么 Master 节点必然需要以某种方式通知其他节点，从而让其他节点执行相应的动作，来完成某些事情。比如建立一个新的 Index 就需要将其 Shard 分配在某些节点上，在这些节点上需要创建出对应 Shard 的目录，并在内存中创建对应 Shard 的一些结构等。

在 ES 中，Master 节点是通过发布 `ClusterState` 来通知其他节点的。Master 会将新的 ClusterState 发布给其他的所有节点，当节点收到新的 ClusterState 后，会把新的 ClusterState 发给相关的各个模块，各个模块根据新的 ClusterState 判断是否要做什么事情，比如创建 Shard 等。即这是一种通过 Meta 数据来驱动各个模块工作的方式。

在 Master 进行 Meta 变更并通知所有节点的过程中，需要考虑 Meta 变更的一致性问题，假如这个过程中 Master 挂掉了，那么可能只有部分节点按照新的 Meta 执行了操作。当选举出新的 Master 后，**需要保证所有节点都要按照最新的 Meta 执行操作**，不能回退，因为已经有节点按照新的 Meta 执行操作了，再回退就会导致不一致。

ES 中只要新 Meta 在一个节点上被 commit，那么就会开始执行相应的操作。因此我们要保证一旦新 Meta 在某个节点上被 commit，此后无论谁是 master，都要基于这个 commit 来产生更新的 meta，否则就可能产生不一致。

Meta 是用来描述数据的数据。在 ES 中，Index 的 mapping 结构、配置、持久化状态等就属于 meta 数据，集群的一些配置信息也属于 meta。这类 meta 数据非常重要，假如记录某个 index 的 meta 数据丢失了，那么集群就认为这个 index 不再存在了。

ES 中的 meta 数据只能由 master 进行更新，并同步给其他节点。集群中的每个节点都会在内存中维护一个当前的 ClusterState，表示当前集群的各种状态。ClusterState 中包含一个 MetaData 的结构，MetaData 中存储的内容更符合 meta 的特征，而且需要持久化的信息都在 MetaData 中，此外的一些变量可以认为是一些临时状态，是集群运行中动态构建出来的。

#### 1.3.7.2. 元数据的恢复

假设 ES 集群重启了，那么所有进程都没有了之前的 Meta 信息，需要有一个角色来恢复 Meta，这个角色就是 Master。所以 ES 集群需要先进行 Master 选举，选出 Master 后，才会进行故障恢复。

当 Master 选举出来后，Master 进程还会等待一些条件，比如集群当前的节点数大于某个数目等，这是避免有些 DataNode 还没有连上来，造成不必要的数据恢复等。

当 Master 进程决定进行恢复 Meta 时，它会向集群中的 MasterNode 和 DataNode 请求其机器上的 MetaData。对于集群的 Meta，选择其中 version 最大的版本。对于每个 Index 的 Meta，也选择其中最大的版本。然后将集群的 Meta 和每个 Index 的 Meta 再组合起来，构成当前的最新 Meta。

#### 1.3.7.3. 元数据的更新

1. 并发更新的控制
    首先，master 进程内不同线程更改 ClusterState 时要保证是原子的。试想一下这个场景，有两个线程都在修改 ClusterState，各自更改其中的一部分。假如没有任何并发的保护，那么最后提交的线程可能就会覆盖掉前一个线程的修改，或者产生不符合条件的状态变更的发生。
    
    ES 解决这个问题的方式是：**串形化**。每次需要更新 ClusterState 时提交一个 Task 给 MasterService，MasterService 中只使用一个线程来串行处理这些 Task，每次处理时把当前的 ClusterState 作为 Task 中 execute 函数的参数。即保证了所有的 Task 都是在 currentClusterState 的基础上进行更改，然后不同的 Task 是串行执行的。
2. 两阶段提交
    新的 Meta 一旦在某个节点上 commit，那么这个节点就会执行相应的操作，比如删除某个 Shard 等，这样的操作是不可回退的。而假如此时 Master 节点挂掉了，新产生的 Master 一定要在新的 Meta 上进行更改，不能出现回退，否则就会出现 Meta 回退了但是操作无法回退的情况。本质上就是 Meta 更新没有保证一致性。
    
    ES 采取了两阶段提交，把 Master 发布 ClusterState 分成两步，第一步是向所有节点 send 最新的 ClusterState，当有超过半数的 master 节点返回 ack 时，再发送 commit 请求，要求节点 commit 接收到的 ClusterState。如果没有超过半数的节点返回 ack，那么认为本次发布失败，同时退出 master 状态，执行 rejoin 重新加入集群。
    
    ![image.png](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202311112240018.png)
    
    采取两阶段提交可以解决一些问题，但并不是完美的：
    1. 第一阶段数据未持久化
        第一阶段 master 节点 send 新的 ClusterState，接收到的节点只是把新的 ClusterState 放入内存一个队列中，就会返回 ack。这个过程没有持久化，所以当 master 接收到超过半数的 ack 后，也不能认为这些节点上都有新的 ClusterState 了。
    2. 节点失效
        如果 master 在 commit 阶段，只 commit 了少数几个节点就出现了网络分区，将 master 与这几个少数节点分在了一起，其他节点可以互相访问。此时其他节点构成多数派，会选举出新的 master，由于这部分节点中没有任何节点 commit 了新的 ClusterState，所以新的 master 仍会使用更新前的 ClusterState，造成 Meta 不一致。

### 1.3.8. 节点数据的恢复

ES 中每个写操作都会分配两个值，`Term` 和 `SequenceNumber`。Term 在每次 Primary 变更时都会加 1，类似于 PacificA 论文中的 `Configuration Version`。`SequenceNumber` 在每次操作后加 1，类似于 PacificA 论文中的 `Serial Number`。由于写请求总是发给 Primary，所以 `Term` 和 `SequenceNumber` 会由 Primary 分配，在向 Replica 发送同步请求时，会带上这两个值。

`LocalCheckpoint` 代表本 Shard 中所有小于该值的请求都已经处理完毕。`GlobalCheckpoint` 代表所有小于该值的请求在所有的 Replica 上都处理完毕。`GlobalCheckpoint` 会由 Primary 进行维护，每个 Replica 会向 Primary 汇报自己的 `LocalCheckpoint`，Primary 根据这些信息来提升 `GlobalCheckpoint`。

`GlobalCheckpoint` 是一个全局的安全位置，代表其前面的请求都被所有 Replica 正确处理了，可以应用在节点故障恢复后的数据回补。另一方面，`GlobalCheckpoint` 也可以用于 `Translog` 的 GC，因为之前的操作记录可以不保存了。

当一个 Replica 故障时，ES 会将其移除，对于故障节点的处理，分为两种情况：
1. 故障超过一定时间
    ES 会分配一个新的 Replica 到新的 Node 上，此时需要全量同步数据。
2. 故障节点自动恢复
    如果之前故障的 Replica 回来了，就可以只回补故障之后的数据，追平后加回来即可，实现快速故障恢复。
    
    实现快速故障恢复的条件有两个：
    1. 能够保存故障期间所有的操作以及其顺序
    2. 能够知道从哪个点开始同步数据
    
    第一个条件可以通过保存一定时间的 Translog 实现，第二个条件可以通过 Checkpoint 实现，所以就能够实现快速的故障恢复。这是 `SequenceNumber` 和 Checkpoint 的第一个重要应用场景。

## 1.4. 分片

ES 支持 PB 级全文搜索，当索引上的数据量太大的时候，ES 通过水平拆分的方式将一个索引上的数据拆分出来分配到不同的数据块上，拆分出来的数据库块称之为一个分片。

这类似于 MySQL 的分库分表，只不过 MySQL 分库分表需要借助第三方组件而 ES 内部自身实现了此功能。在一个多分片的索引中写入数据时，通过路由来确定具体写入哪一个分片中，所以在创建索引的时候需要指定分片的数量，并且分片的数量一旦确定就不能修改。

分片的数量和下面介绍的副本数量都是可以通过创建索引时的 Settings 来配置，ES 默认为一个索引创建 5 个主分片, 并分别为每个分片创建一个副本。

## 1.5. 副本

副本就是对分片的 Copy，每个主分片都有一个或多个副本分片，当主分片异常时，副本可以提供数据的查询等操作。主分片和对应的副本分片是不会在同一个节点上的，所以副本分片数的最大值是 N-1（其中 N 为节点数）。对文档的新建、索引和删除请求都是写操作，必须在主分片上面完成之后才能被复制到相关的副本分片。

ES 为了提高写入的能力这个过程是并发写的，同时为了解决并发写的过程中数据冲突的问题，ES 通过乐观锁的方式控制，每个文档都有一个 `_version`（版本）号，当文档被修改时版本号递增。一旦所有的副本分片都报告写成功才会向协调节点报告成功，协调节点向客户端报告成功。

![image.png](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202309101908800.png)

当原主分片故障后，副本分片会被提升为主分片，这个提升主分片的过程是瞬间发生的。此时集群的状态将会为 Yellow。当该分片恢复时，如果期间有更改的数据只需要从主分片上复制修改的数据文件即可。

# 2. 数据存储原理

## 2.1. Es 的存储模型

![image.png](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202311030114347.png)

## 2.2. 索引写入原理

下图描述了 3 个节点的集群，共拥有 12 个分片，其中有 4 个主分片（S0、S1、S2、S3）和 8 个副本分片（R0、R1、R2、R3），每个主分片对应两个副本分片，节点 1 是主节点（Master 节点）负责整个集群的状态。

![image.png](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202309101911565.png)

写索引是只能写在主分片上，然后同步到副本分片。这里有四个主分片，一条数据 ES 是根据什么规则写到特定分片上的呢？

$$
shard=hash(routing) \% number\_of\_primary\_shards
$$

Routing 是一个可变值，默认是文档的 `_id`，也可以设置成一个自定义的值。Routing 通过 Hash 函数生成一个数字，然后这个数字再除以 `number_of_primary_shards`（主分片的数量）后得到余数。

因此**在创建索引的时候就需要确定主分片的数量并且永远无法改变这个数量**，因为如果数量变化了，那么所有之前路由的值都会无效，文档也再也找不到了。

由于在 ES 集群中每个节点通过上面的计算公式都知道集群中的文档的存放位置，所以每个节点都有处理读写请求的能力。在一个写请求被发送到某个节点后，协调节点会根据路由公式计算出需要写到哪个分片上，再将请求转发到该分片的主分片节点上。

![image.png](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202309101915606.png)

每次写入的时候，写入请求会先根据_routing 规则选择发给哪个 Shard，Index Request 中可以设置使用哪个 Filed 的值作为路由参数，如果没有设置，则使用 Mapping 中的配置，如果 Mapping 中也没有配置，则使用 id 作为路由参数。然后通过 routing 的 Hash 值选择出 Shard，最后从集群的 Meta 中找出该 Shard 的 Primary 节点。

请求接着会发送给 Primary Shard，在 Primary Shard 上执行成功后，再从 Primary Shard 上将请求同时发送给多个 Replica Shard，请求在多个 Replica Shard 上执行成功并返回给 Primary Shard 后，写入请求执行成功，返回结果给客户端。

这种模式下，写入操作的延时就等于 latency = Latency (Primary Write) + Max (Replicas Write)。只要有副本在，写入延时最小也是两次单 Shard 的写入时延总和，写入效率会较低，但是这样的好处也很明显，避免写入后，单机或磁盘故障导致数据丢失，在数据重要性和性能方面，一般都是优先选择数据，除非一些允许丢数据的特殊场景。

## 2.3. 数据存储原理

### 2.3.1. 分段写

索引文档以[[数据密集型系统设计1：引入#1.2.2. SSTables (SortStringTables)|段的形式]]存储在磁盘上，何为段？索引文件被拆分为多个子文件，则每个子文件叫作段，每一个段本身都是一个倒排索引，并且段具有不变性，一旦索引的数据被写入硬盘，就不可再修改。在底层采用了分段的存储模式，使它在读写时几乎完全避免了锁的出现，大大提升了读写性能 （在并发更新文档的场景下，ES 是采用乐观锁版本号的方式来实现并发控制）。

段被写入到磁盘后会生成一个提交点，提交点是一个用来记录所有提交后段信息的文件。一个段一旦拥有了提交点，就说明这个段只有读的权限，失去了写的权限。相反，当段在内存中时，就只有写的权限，而不具备读数据的权限，意味着不能被检索。

段的概念提出主要是因为：在早期全文检索中为整个文档集合建立了一个很大的倒排索引，并将其写入磁盘中。如果索引有更新，就需要重新全量创建一个索引来替换原来的索引。这种方式在数据量很大时效率很低，并且由于创建一次索引的成本很高，所以对数据的更新不能过于频繁，也就不能保证时效性。

索引文件分段存储并且不可修改，那么新增、更新和删除如何处理呢？

- 新增
    由于数据是新的，所以只需要对当前文档新增一个段就可以了。
- 删除
    由于不可修改，所以对于删除操作，不会把文档从旧的段中移除而是通过新增一个 `.del` 文件，文件中会列出这些被删除文档的段信息。这个被标记删除的文档仍然可以被查询匹配到，但它会在最终结果被返回前从结果集中移除。
- 更新
    不能修改旧的段来进行反映文档的更新，其实更新相当于是删除和新增这两个动作组成。会将旧的文档在 `.del` 文件中标记删除，然后文档的新版本被索引到一个新的段中。可能两个版本的文档都会被一个查询匹配到，但被删除的那个旧版本文档在结果集返回前就会被移除。

段被设定为不可修改具有一定的优势也有一定的缺点，优势主要表现在：

- 不需要锁：如果你从来不更新索引，你就不需要担心多进程同时修改数据的问题。
- 一旦索引被读入内核的文件系统缓存，便会留在哪里，由于其不变性。只要文件系统缓存中还有足够的空间，那么大部分读请求会直接请求内存，而不会命中磁盘。这提供了很大的性能提升。
- 其它缓存 (像 Filter 缓存)，在索引的生命周期内始终有效。它们不需要在每次数据改变时被重建，因为数据不会变化。
- 写入单个大的倒排索引允许数据被压缩，减少磁盘 I/O 和需要被缓存到内存的索引的使用量。

段的不变性的缺点如下：

- 当对旧数据进行删除时，旧数据不会马上被删除，而是在 `.del` 文件中被标记为删除。而旧数据只能等到段更新时才能被移除，这样会造成大量的空间浪费。
- 若有一条数据频繁的更新，每次更新都是新增新的标记旧的，则会有大量的空间浪费。
- 每次新增数据时都需要新增一个段来存储数据。当段的数量太多时，对服务器的资源例如文件句柄的消耗会非常大。
- 在查询的结果中包含所有的结果集，需要排除被标记删除的旧数据，这增加了查询的负担。

#### 2.3.1.1. Create 流程

![image.png](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202311112029405.png)

每当有新 Document 要写入时，会进行以下操作：

1. 文档会先写入 Index Buffer，作为缓冲区
2. 将 Index Buffer 写入 Segment（内存），然后文档就可以被查询到了
3. 将 Index Buffer 同时写入 Transaction Log（WAL 机制），用于断电恢复数据
4. 将 Segments 落盘

其中第 2、3 两步合并成为 Refresh，默认 1 秒（`index.refresh_interval`）执行一次，这也就是为什么 ES 是近实时搜索引擎的原因（高版本的 ES 默认落盘）；另外 Index Buffer 被写满时也会触发，默认大小是 JVM 内存的 10%

其中第 4 步，其实是归属于 Flush 操作的一个步骤。Flush 默认 30 分钟执行一次，或者 Transaction Log 满（默认 512MB）也会触发。Flush 执行包含的流程有

1. 调用 Refresh
2. 调用 fsync，将 Segments 落盘
3. 清空 Transaction Log

这里有几个关键点：

1. 写入时机  
    和数据库不同，数据库是先写 CommitLog，然后再写内存，而 Elasticsearch 是先写内存，最后才写 TransLog。
    
    一种可能的原因是 Lucene 的内存写入会有很复杂的逻辑，很容易失败，比如分词，字段长度超过限制等，比较重，为了避免 TransLog 中有大量无效记录，减少 recover 的复杂度和提高速度，所以就把写 Lucene 放在了最前面。
2. 写 Lucene 内存后，并不是可被搜索的  
    需要通过 Refresh 把内存的对象转成完整的 Segment 后，然后再次 reopen 后才能被搜索，一般这个时间设置为 1 秒钟，导致写入 Elasticsearch 的文档，最快要 1 秒钟才可被从搜索到，所以 Elasticsearch 在搜索方面是 NRT（Near Real Time）近实时的系统。
3. 查询方式是 GetById 时可以直接从 TransLog 中查询，这时候就成了 RT（Real Time）实时系统
4. 定时刷新 segment 到磁盘
    每隔一段比较长的时间，比如 30 分钟后，Lucene 会把内存中生成的新 Segment 刷新到磁盘上，刷新后索引文件已经持久化了，历史的 TransLog 就没用了，会清空掉旧的 TransLog。

#### 2.3.1.2. Update 流程

![image.png](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202311112036650.png)

Lucene 中不支持部分字段的 Update，所以需要在 Elasticsearch 中实现该功能，具体流程如下：

1. 读取历史数据
    收到 Update 请求后，从 Segment 或者 TransLog 中读取同 id 的完整 Doc，记录版本号为 V1。
2. 合并新旧数据
    将版本 V1 的全量 Doc 和请求中的部分字段 Doc 合并为一个完整的 Doc，同时更新内存中的 VersionMap。获取到完整 Doc 后，Update 请求就变成了 Index 请求。
3. 加锁
    锁是 Refresh Lock，禁止 refresh 该条数据。
4. 再次检查版本
    再次从 versionMap 中读取该 id 的最大版本号 V2，如果 versionMap 中没有，则从 Segment 或者 TransLog 中读取，这里基本都会从 versionMap 中获取到。
    
    检查版本是否冲突： `V1 == V2`，如果冲突，则回退到开始的“Update doc” 阶段，重新执行。如果不冲突，则执行最新的 Add 请求。
5. 更新数据
    在 Index Doc 阶段，首先将 Version + 1 得到 V3，再将 Doc 加入到 Lucene 中去，Lucene 中会先删同 id 下的已存在 doc id，然后再增加新 Doc。写入 Lucene 成功后，将当前 V3 更新到 versionMap 中。
6. 释放锁
    此时部分更新的流程就结束了。

#### 2.3.1.3. 整体写入流程

![image.png](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202311112041329.png)

- 红色：Client Node。
- 绿色：Primary Node。
- 蓝色：Replica Node。

以下解释上图中写入的核心过程：
1. Client Node 执行过程
    1. Auto Create Index
        判断当前 Index 是否存在，如果不存在，则需要自动创建 Index，这里需要和 Master 交互。也可以通过配置关闭自动创建 Index 的功能。
    2. Set Routing
        设置路由条件，如果 Request 中指定了路由条件，则直接使用 Request 中的 Routing，否则使用 Mapping 中配置的，如果 Mapping 中无配置，则使用默认的_id 字段值。
        
        在这一步中，如果没有指定_id 字段，则会自动生成一个唯一的_id 字段，目前使用的是 UUID。
    3. Construct BulkShardRequest
        由于 Bulk Request 中会包括多个 (Index/Update/Delete) 请求，这些请求根据 routing 可能会落在多个 Shard 上执行，这一步会按 Shard 挑拣 Single Write Request，同一个 Shard 中的请求聚集在一起，构建 BulkShardRequest，每个 BulkShardRequest 对应一个 Shard。
    4. Send Request To Primary
        将上一步每一个 BulkShardRequest 请求发送给相应 Shard 的 Primary Node。
2. Primary Node 执行过程
    Create/Index 是直接新增 Doc，Delete 是直接根据 id 删除 Doc，Update 会稍微复杂些，我们下面就以 Update 为例来介绍。
    
    1. Translate Update To Index or Delete
        这一步是 Update 操作的特有步骤，在这里，会将 Update 请求转换为 Index 或者 Delete 请求。首先，会通过 GetRequest 查询到已经存在的同 id Doc（如果有）的完整字段和值（依赖 source 字段），然后和请求中的 Doc 合并。同时，这里会获取到读到的 Doc 版本号，记做 V1。
    2. Parse Doc
        解析 Doc 中各个字段。
    3. Update Mapping
        Elasticsearch 中有个自动更新 Mapping 的功能，就在这一步生效。会先挑选出 Mapping 中未包含的新 Field，然后判断是否运行自动更新 Mapping，如果允许，则更新 Mapping。
    4. Get Sequence Id and Version
        由于当前是 Primary Shard，则会从 SequenceNumber Service 获取一个 sequenceID 和 Version。SequenceID 在 Shard 级别每次递增 1，SequenceID 在写入 Doc 成功后，会用来初始化 LocalCheckpoint。Version 则是根据当前 Doc 的最大 Version 递增 1。
    5. Add Doc To Lucene
        这一步开始的时候会给特定 uid 加锁，然后判断该 uid 对应的 Version 是否等于之前 Translate Update To Index 步骤里获取到的 Version，如果不相等，则说明刚才读取 Doc 后，该 Doc 发生了变化，出现了版本冲突，这时候会抛出一个 VersionConflict 的异常，该异常会在 Primary Node 最开始处捕获，重新从 “Translate Update To Index or Delete” 开始执行。
        
        如果 Version 相等，则继续执行，如果已经存在同 id 的 Doc，则会调用 Lucene 的 UpdateDocument (uid, doc) 接口，先根据 uid 删除 Doc，然后再 Index 新 Doc。如果是首次写入，则直接调用 Lucene 的 AddDocument 接口完成 Doc 的 Index，AddDocument 也是通过 UpdateDocument 实现。
        
        这一步中有个问题是，如何保证 Delete-Then-Add 的原子性，怎么避免中间状态时被 Refresh？答案是在开始 Delete 之前，会加一个 Refresh Lock，禁止被 Refresh，只有等 Add 完后释放了 Refresh Lock 后才能被 Refresh，这样就保证了 Delete-Then-Add 的原子性。
        
        > [!info] Lucene字段写入过程
        > </br> Lucene 的 UpdateDocument 接口中就只是处理多个 Field，会遍历每个 Field 逐个处理，处理顺序是 invert index，store field，doc values，point dimension。
        
    6. Write Translog
        写完 Lucene 的 Segment 后，会以 keyvalue 的形式写 TransLog，Key 是_id，Value 是 Doc 内容。当查询的时候，如果请求是 GetDocByID，则可以直接根据_id 从 TransLog 中读取到，满足 NoSQL 场景下的实时性要去。
        
        > [!attention] 注意
        > </br>这里**只是写入到内存的 TransLog**，是否 Sync 到磁盘的逻辑还在后面。这一步的最后，会标记当前 SequenceID 已经成功执行，接着会更新当前 Shard 的 LocalCheckPoint。
    7. Renew Bulk Request
        重新构造 Bulk Request，原因是前面已经将 UpdateRequest 翻译成了 Index 或 Delete 请求，则后续所有 Replica 中只需要执行 Index 或 Delete 请求就可以了，不需要再执行 Update 逻辑，一是保证 Replica 中逻辑更简单，性能更好，二是保证同一个请求在 Primary 和 Replica 中的执行结果一样。
    8. Flush Translog
        根据 TransLog 的策略，选择不同的执行方式，要么是立即 Flush 到磁盘，要么是等到以后再 Flush。Flush 的频率越高，可靠性越高，对写入性能影响越大。
    9. Send Requests To Replicas
        将刚才构造的新的 Bulk Request 并行发送给多个 Replica，然后等待 Replica 的返回，这里需要等待所有 Replica 返回后（可能有成功，也有可能失败），Primary Node 才会返回用户。如果某个 Replica 失败了，则 Primary 会给 Master 发送一个 Remove Shard 请求，要求 Master 将该 Replica Shard 从可用节点中移除。
        
        > [!info] 副本写入的限制
        > </br> ES 中有一个参数，叫做 wait_for_active_shards，这个参数是 Index 的一个 setting，也可以在请求中带上这个参数。这个参数的含义是，<b>在每次写入前，该 shard 至少具有的 active 副本数</b>。假设我们有一个 Index，其每个 Shard 有 3 个 Replica，加上 Primary 则总共有 4 个副本。如果配置 wait_for_active_shards 为 3，那么允许最多有一个 Replica 挂掉，如果有两个 Replica 挂掉，则 Active 的副本数不足 3，此时不允许写入。
        > 
        > 这个参数默认是 1，即只要 Primary 在就可以写入，起不到什么作用。如果配置大于 1，可以起到一种保护的作用，保证写入的数据具有更高的可靠性。但是这个参数只在写入前检查，并不保证数据一定在至少这些个副本上写入成功，所以并不是严格保证了最少写入了多少个副本。
        
        > [!attention]  副本写入失败的后果
        > </br> 如果一个 Replica 写失败了，Primary 会将这个信息报告给 Master，然后 Master 会在 Meta 中更新这个 Index 的 InSyncAllocations 配置，将这个 Replica 从中移除，移除后它就不再承担读请求。在 Meta 更新到各个 Node 之前，用户可能还会读到这个 Replica 的数据，但是更新了 Meta 之后就不会了。所以这个方案并不是非常的严格，考虑到 ES 本身就是一个近实时系统，数据写入后需要 refresh 才可见，所以一般情况下，在短期内读到旧数据应该也是可接受的。
        
    11. Receive Response From Replicas
        Replica 中请求都处理完后，会更新 Primary Node 的 LocalCheckPoint。
3. Replica Node 执行过程
    Replica Node 执行过程和 Primary Node 执行过程基本一致，只不过少了同步 Replica Node 与转化 Update 请求的过程。

#### 2.3.1.4. 写入机制分析

对于 Elasticsearch 的写入流程及其各个流程的工作机制，我们在此分析它是否符合分布式系统中的一些特性：

1. 可靠性
    由于 Lucene 的设计中不考虑可靠性，在 Elasticsearch 中通过 Replica 和 TransLog 两套机制保证数据的可靠性。
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

Elasticsearch 中每个 Shard 都会有多个 Replica，主要是为了保证数据可靠性，除此之外，还可以增加读能力，因为写的时候虽然要写大部分 Replica Shard，但是查询的时候只需要查询 Primary 和 Replica 中的任何一个就可以了。

![image.png](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202311130052812.png)

当查询的时候，从三个节点中根据 Request 中的 preference 参数选择一个节点查询。preference 可以设置 `_local`，`_primary`，`_replica` 以及其他选项。如果选择了 primary，则每次查询都是直接查询 Primary，可以保证每次查询都是最新的。如果设置了其他参数，那么可能会查询到 R1 或者 R2，这时候就有可能查询不到最新的数据。

Elasticsearch 中通过分区实现分布式，数据写入的时候根据 \_routing 规则将数据写入某一个 Shard 中，这样就能将海量数据分布在多个 Shard 以及多台机器上，以达到分布式的目标。但这样就导致了查询的时候，潜在数据会在当前 index 的所有的 Shard 中，所以 Elasticsearch 查询的时候需要查询所有 Shard，同一个 Shard 的 Primary 和 Replica 选择一个即可，查询请求会分发给所有 Shard，每个 Shard 中都是一个独立的查询引擎，比如需要返回 Top 10 的结果，那么每个 Shard 都会查询并且返回 Top 10 的结果，然后在 Client Node 里面会接收所有 Shard 的结果，然后通过优先级队列二次排序，选择出 Top 10 的结果返回给用户。

![image.png](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202311130053222.png)

> [!warning]  请求膨胀问题
>
>这里有一个问题就是请求膨胀，用户的一个搜索请求在 Elasticsearch 内部会变成 Shard 个请求。一种优化方式是虽然会产生 Shard 个数量的请求，但是这个 Shard 个数不一定要是当前 Index 中的 Shard 个数，只要是当前查询相关的 Shard 即可，这个需要基于业务和请求内容优化，通过这种方式可以优化请求膨胀数。

##### 2.3.1.5.2. 节点内数据的获取

Elasticsearch 中的查询主要分为两类：Get 请求：通过 ID 查询特定 Doc；Search 请求：通过 Query 查询匹配 Doc。

![image.png](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202311130057857.png)

对于 Search 类请求，查询的时候是一起查询内存（指刚 Refresh Segment，但是还没持久化到磁盘的新 Segment）和磁盘上的 Segment，最后将结果合并后返回。这种查询是近实时（Near Real Time）的，主要是由于内存中的 Index 数据需要一段时间后才会刷新为 Segment。

对于 Get 类请求，查询的时候是先查询内存中的 TransLog，如果找到就立即返回，如果没找到再查询磁盘上的 TransLog，如果还没有则再去查询磁盘上的 Segment。这种查询是实时（Real Time）的。这种查询顺序可以保证查询到的 Doc 是最新版本的 Doc，这个功能也是为了保证 NoSQL 场景下的实时性要求。

##### 2.3.1.5.3. 数据查询阶段

![image.png](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202311130059187.png)

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

1. 写入数据不指定 doc_id，让 ES 自动生成。
    如果自定义 doc_id 的话，则 ES 在写入过程中会多一步判断的过程，即先 Get 下该 doc_id 是否已经存在。如果存在的话则执行 Update 操作，不存在则创建新的 doc。
    
    因此如果我们对索引 doc_id 没有特别要求，则建议让 ES 自动生成 doc_id，这样可提升一定的写入性能。
2. 提前创建索引，使用固定的索引 mapping。
    创建索引及新加字段都是更新元数据操作，需要 master 节点将新版本的元数据同步到所有节点。
    
    因此在集群规模比较大，写入 qps 较高的场景下，特别容易出现 master 更新元数据超时的问题，这可导致 master 节点中有大量的 pending_tasks 任务堆积，从而造成集群不可用，甚至出现集群无主的情况。
3. 实时性要求不高的索引增大 refresh_interval 的时间。
    ES 默认的 refresh_interval 是 1s，即 doc 写入 1s 后即可被搜索到。如果业务对数据实时性要求不高的话，如日志场景，可将索引模版的 refresh_interval 设置成 30s，这能够避免过多的小 segment 文件的生成及段合并的操作。
4. 追求写入效率的场景，可先将索引副本数设置为单副本，写入完成后再打开副本。
    例如迁移数据时可先将副本数设置为 0，迁移完毕后再设置回来。
5. 使用 Bulk 批量插入数据时，控制单词 bulk 数量在 10M 左右。
    通常我们建议一次 Bulk 的数据量控制在 10M 以下，一次 Bulk 的 doc 数在 10000 上下浮动。
6. 使用自定义 routing 功能，尽量将请求转发到较少的分片。
    协调节点是异步将数据发送给所有的分片，但是却需要等待所有的分片响应后才能返回给客户端，因此一次 Bulk 的延迟则取决于响应最慢的那个分片所在的节点。这就是分布式系统的长尾效应。
    
    因此，我们可以自定义 routing 值，这样写入、查询均带有路由字段信息。请求只会发送给部分分片，避免全量分片扫描。这些节点完成查询后将结果返回给请求节点，由请求节点汇聚各个节点的结果返回给客户端。
    
    ![image.png](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202309110033326.png)
    
7. 尽量选择 SSD 磁盘类型。
8. 冻结历史索引，释放内存空间。
    
    ![image.png](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202309110034464.png)
    
    Open 状态的索引由于是通过将倒排索引以 FST 数据结构的方式加载进内存中，因此索引是能够被快速搜索的，且搜索速度也是最快的。但是需要消耗大量的内存空间，且这部分内存为常驻内存，不会被 GC 的。1T 的索引预计需要消耗 2-4 GB 的 JVM 堆内存空间。
    
    > [!info] FST
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
    
    Frozen 状态的索引特点是可被搜索，但是由于它不占用内存，只是存储在磁盘上，因此冻结索引的搜索速度是相对比较慢的。如果我们集群中的数据量比较大，历史数据也不能被删除，则可以考虑使用下面的 API 将历史索引冻结起来，这样便可释放出较多的内存空间。

## 3.3. 集群性能优化

以下性能优化点参考[腾讯的 ES 性能优化](https://mp.weixin.qq.com/s/CNf75yT0A0QPki-Qhw3_8w)一节。

![image.png](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202309110212386.png)

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

![image.png](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202309110214557.png)

### 3.3.2. 成本优化

硬盘成本方面，由于数据具有明显的冷热特性，首先我们采用冷热分离架构，使用混合存储的方案来平衡成本、性能；其次，既然对历史数据通常都是访问统计信息，那么以通过预计算来换取存储和性能；如果历史数据完全不使用，也可以备份到更廉价的存储系统；其他一些优化方式包含存储裁剪、生命周期管理等。

内存成本方面，很多用户在使用大存储机型时会发现，存储资源才用了百分之二十，内存已经不足。其实基于时序数据的访问特性，我们可以利用 Cache 进行优化。

![image.png](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202309110216043.png)

### 3.3.3. 性能优化

以日志、监控为代表的时序场景，对写入性能要求非常高，写入并发可达 1000w/s。然而我们发现在带主键写入时，ES 性能衰减 1+倍，部分压测场景下，CPU 无法充分利用。以搜索服务为代表的场景，对查询性的要求非常高，要求 20w QPS, 平响 20ms，而且尽量避免 GC、执行计划不优等造成的查询毛刺。

![image.png](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202309110217952.png)

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
2. 分片数量超过 21 亿条限制
    该限制是分片维度而不是索引维度的。因此出现这种异常，通常是由于我们的索引分片设置的不是很合理。
    
    解决方法：切换写入到新索引，并修改索引模版，合理设置主分片数。
3. 主分片所在节点掉线
    这种情况通常是由于某个节点故障或者由于负载较高导致的掉线。
    
    解决方法：找到节点掉线原因并重新启动节点加入集群，等待分片恢复。
4. 索引所需属性与节点属性不匹配
    解决方法：重新设置索引所需的属性，和节点保持一致。因为如果重新设置节点属性，则需要重启节点，代价较高。
5. 节点长时间掉线后重新加入集群，引入了脏数据
    解决方法：通过 reroute API 来重新分配一个主分片。
6. 未分配分片太多，达到了分片恢复的阈值，其他分片排队等待
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

以下介绍几种管理索引容量的方法：

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
2. 通过别名写入数据
    例如 `POST /myro_write_alias/_bulk`
3. 执行 rollover 操作
    
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

![image.png](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202309110159297.png)

- Hot：索引可写入，也可查询，也就是我们通常说的热数据，为保证性能数据通常都是在内存中的。
- Warm：索引不可写入，但可查询，介于热和冷之间，数据可以是全内存的，也可以是在 SSD 的硬盘上的。
- Cold：索引不可写入，但很少被查询，查询的慢点也可接受，基本不再使用的数据，数据通常在大容量的磁盘上。
- Delete：索引可被安全的删除。

这 4 个阶段是 ES 定义的一个索引从生到死的过程, Hot -> Warm -> Cold -> Delete 4 个阶段只有 Hot 阶段是必须的，其他 3 个阶段根据业务的需求可选。

通过 ILM 按以下方式管理索引：

1. 建立 Lifecycle 策略
    
    ![image.png](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/202309110202931.png)
    
2. 建立索引模版
    
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
3. 创建索引
    
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
4. 查看索引配置
    `GET /myes-testindex-000001`
5. 写入数据
    `POST /myes_write_alias/_bulk`
6. 配置 Lifecycle 自动 Rollover 的时间间隔
    由于 ES 是一个准实时系统，很多操作都不能实时生效。Lifecycle 的 rollover 之所以不用每次手动执行 rollover 操作是因为 ES 会隔一段时间判断一次索引是否满足 rollover 的条件，ES 检测 ILM 策略的时间默认为 10min。如果需要手动修改这个时间间隔的话可以修改 Lifecycle 配置：
    
    ```bash
    PUT _cluster/settings
    {
      "transient": {
        "indices.lifecycle.poll_interval": "3s"
      }
    }
    ```

