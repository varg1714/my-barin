目前的 Elasticsearch 有两个明显的身份，一个是分布式搜索系统，另一个是分布式 NoSQL 数据库，对于这两种不同的身份，读写语义基本类似，但也有一点差异。

![](https://pic4.zhimg.com/v2-aced78c779161b2a5baa366f63d86883_r.jpg)

## **写操作**

*   实时性：

*   搜索系统的 Index 一般都是 NRT（Near Real Time），近实时的，比如 Elasticsearch 中，Index 的实时性是由 refresh 控制的，默认是 1s，最快可到 100ms，那么也就意味着 Index doc 成功后，需要等待一秒钟后才可以被搜索到。
*   NoSQL 数据库的 Write 基本都是 RT（Real Time），实时的，写入成功后，立即是可见的。Elasticsearch 中的 Index 请求也能保证是实时的，因为 Get 请求会直接读内存中尚未 Flush 到存储介质的 TransLog。

*   可靠性：

*   搜索系统对可靠性要求都不高，一般数据的可靠性通过将原始数据存储在另一个存储系统来保证，当搜索系统的数据发生丢失时，再从其他存储系统导一份数据过来重新 rebuild 就可以了。在 Elasticsearch 中，通过设置 TransLog 的 Flush 频率可以控制可靠性，要么是按请求，每次请求都 Flush；要么是按时间，每隔一段时间 Flush 一次。一般为了性能考虑，会设置为每隔 5 秒或者 1 分钟 Flush 一次，Flush 间隔时间越长，可靠性就会越低。
*   NoSQL 数据库作为一款数据库，必须要有很高的可靠性，数据可靠性是生命底线，决不能有闪失。如果把 Elasticsearch 当做 NoSQL 数据库，此时需要设置 TransLog 的 Flush 策略为每个请求都要 Flush，这样才能保证当前 Shard 写入成功后，数据能尽量持久化下来。

上面简单介绍了下 NoSQL 数据库和搜索系统的一些异同，我们会在后面有一篇文章，专门用来介绍 Elasticsearch 作为 NoSQL 数据库时的一些局限和特点。

## 读操作

下一篇《Elasticsearch 内核解析 - 查询篇》中再详细介绍。

上面大概对比了下搜索和 NoSQL 在写方面的特点，接下来，我们看一下 Elasticsearch 6.0.0 版本中写入流程都做了哪些事情，希望能对大家有用。

## **写操作的关键点**

在考虑或分析一个分布式系统的写操作时，一般需要从下面几个方面考虑：

*   可靠性：或者是持久性，数据写入系统成功后，数据不会被回滚或丢失。
*   一致性：数据写入成功后，再次查询时必须能保证读取到最新版本的数据，不能读取到旧数据。
*   原子性：一个写入或者更新操作，要么完全成功，要么完全失败，不允许出现中间状态。
*   隔离性：多个写入操作相互不影响。
*   实时性：写入后是否可以立即被查询到。
*   性能：写入性能，吞吐量到底怎么样。

Elasticsearch 作为分布式系统，也需要在写入的时候满足上述的四个特点，我们在后面的写流程介绍中会涉及到上述四个方面。

接下来, 我们一层一层剖析 Elasticsearch 内部的写机制。

## **Lucene 的写**

众所周知，Elasticsearch 内部使用了 Lucene 完成索引创建和搜索功能，Lucene 中写操作主要是通过 IndexWriter 类实现，IndexWriter 提供三个接口：

```
public long addDocument();
 public long updateDocuments();
 public long deleteDocuments();
```

通过这三个接口可以完成单个文档的写入，更新和删除功能，包括了分词，倒排创建，正排创建等等所有搜索相关的流程。只要 Doc 通过 IndesWriter 写入后，后面就可以通过 IndexSearcher 搜索了，看起来功能已经完善了，但是仍然有一些问题没有解：

1.  上述操作是单机的，而不是我们需要的分布式。
2.  文档写入 Lucene 后并不是立即可查询的，需要生成完整的 Segment 后才可被搜索，如何保证实时性？
3.  Lucene 生成的 Segment 是在内存中，如果机器宕机或掉电后，内存中的 Segment 会丢失，如何保证数据可靠性 ？
4.  Lucene 不支持部分文档更新，但是这又是一个强需求，如何支持部分更新？

上述问题，在 Lucene 中是没有解决的，那么就需要 Elasticsearch 中解决上述问题。

Elasticsearch 在解决上述问题时，除了我们在上一篇《Elasticsearch 数据模型简介》中介绍的几种系统字段外，在引擎架构上也引入了多重机制来解决问题。我们再来看 Elasticsearch 中的写机制。

## **Elasticsearch 的写**

Elasticsearch 采用多 Shard 方式，通过配置 routing 规则将数据分成多个数据子集，每个数据子集提供独立的索引和搜索功能。当写入文档的时候，根据 routing 规则，将文档发送给特定 Shard 中建立索引。这样就能实现分布式了。

此外，Elasticsearch 整体架构上采用了一主多副的方式：

![](https://pic2.zhimg.com/v2-8203d235d8cfc14849012e6ea229fa89_r.jpg)

每个 Index 由多个 Shard 组成，每个 Shard 有一个主节点和多个副本节点，副本个数可配。但每次写入的时候，写入请求会先根据_routing 规则选择发给哪个 Shard，Index Request 中可以设置使用哪个 Filed 的值作为路由参数，如果没有设置，则使用 Mapping 中的配置，如果 mapping 中也没有配置，则使用_id 作为路由参数，然后通过_routing 的 Hash 值选择出 Shard（在 OperationRouting 类中），最后从集群的 Meta 中找出出该 Shard 的 Primary 节点。

请求接着会发送给 Primary Shard，在 Primary Shard 上执行成功后，再从 Primary Shard 上将请求同时发送给多个 Replica Shard，请求在多个 Replica Shard 上执行成功并返回给 Primary Shard 后，写入请求执行成功，返回结果给客户端。

这种模式下，写入操作的延时就等于 latency = Latency(Primary Write) + Max(Replicas Write)。只要有副本在，写入延时最小也是两次单 Shard 的写入时延总和，写入效率会较低，但是这样的好处也很明显，避免写入后，单机或磁盘故障导致数据丢失，在数据重要性和性能方面，一般都是优先选择数据，除非一些允许丢数据的特殊场景。

采用多个副本后，避免了单机或磁盘故障发生时，对已经持久化后的数据造成损害，但是 Elasticsearch 里为了减少磁盘 IO 保证读写性能，一般是每隔一段时间（比如 5 分钟）才会把 Lucene 的 Segment 写入磁盘持久化，对于写入内存，但还未 Flush 到磁盘的 Lucene 数据，如果发生机器宕机或者掉电，那么内存中的数据也会丢失，这时候如何保证？

对于这种问题，Elasticsearch 学习了数据库中的处理方式：增加 CommitLog 模块，Elasticsearch 中叫 TransLog。

![](https://pic4.zhimg.com/v2-20a780ddd33a74b37a81e18d3baf8983_r.jpg)

在每一个 Shard 中，写入流程分为两部分，先写入 Lucene，再写入 TransLog。

写入请求到达 Shard 后，先写 Lucene 文件，创建好索引，此时索引还在内存里面，接着去写 TransLog，写完 TransLog 后，刷新 TransLog 数据到磁盘上，写磁盘成功后，请求返回给用户。这里有几个关键点，一是和数据库不同，数据库是先写 CommitLog，然后再写内存，而 Elasticsearch 是先写内存，最后才写 TransLog，一种可能的原因是 Lucene 的内存写入会有很复杂的逻辑，很容易失败，比如分词，字段长度超过限制等，比较重，为了避免 TransLog 中有大量无效记录，减少 recover 的复杂度和提高速度，所以就把写 Lucene 放在了最前面。二是写 Lucene 内存后，并不是可被搜索的，需要通过 Refresh 把内存的对象转成完整的 Segment 后，然后再次 reopen 后才能被搜索，一般这个时间设置为 1 秒钟，导致写入 Elasticsearch 的文档，最快要 1 秒钟才可被从搜索到，所以 Elasticsearch 在搜索方面是 NRT（Near Real Time）近实时的系统。三是当 Elasticsearch 作为 NoSQL 数据库时，查询方式是 GetById，这种查询可以直接从 TransLog 中查询，这时候就成了 RT（Real Time）实时系统。四是每隔一段比较长的时间，比如 30 分钟后，Lucene 会把内存中生成的新 Segment 刷新到磁盘上，刷新后索引文件已经持久化了，历史的 TransLog 就没用了，会清空掉旧的 TransLog。

上面介绍了 Elasticsearch 在写入时的两个关键模块，Replica 和 TransLog，接下来，我们看一下 Update 流程：

![](https://pic4.zhimg.com/v2-e728bc042a75f8798925d708dc61b1ef_r.jpg)

Lucene 中不支持部分字段的 Update，所以需要在 Elasticsearch 中实现该功能，具体流程如下：

1.  收到 Update 请求后，从 Segment 或者 TransLog 中读取同 id 的完整 Doc，记录版本号为 V1。
2.  将版本 V1 的全量 Doc 和请求中的部分字段 Doc 合并为一个完整的 Doc，同时更新内存中的 VersionMap。获取到完整 Doc 后，Update 请求就变成了 Index 请求。
3.  加锁。
4.  再次从 versionMap 中读取该 id 的最大版本号 V2，如果 versionMap 中没有，则从 Segment 或者 TransLog 中读取，这里基本都会从 versionMap 中获取到。
5.  检查版本是否冲突 (V1==V2)，如果冲突，则回退到开始的“Update doc” 阶段，重新执行。如果不冲突，则执行最新的 Add 请求。
6.  在 Index Doc 阶段，首先将 Version + 1 得到 V3，再将 Doc 加入到 Lucene 中去，Lucene 中会先删同 id 下的已存在 doc id，然后再增加新 Doc。写入 Lucene 成功后，将当前 V3 更新到 versionMap 中。
7.  释放锁，部分更新的流程就结束了。

介绍完部分更新的流程后，大家应该从整体架构上对 Elasticsearch 的写入有了一个初步的映象，接下来我们详细剖析下写入的详细步骤。

## **Elasticsearch 写入请求类型**

Elasticsearch 中的写入请求类型，主要包括下列几个：Index(Create)，Update，Delete 和 Bulk，其中前 3 个是单文档操作，后一个 Bulk 是多文档操作，其中 Bulk 中可以包括 Index(Create)，Update 和 Delete。

在 6.0.0 及其之后的版本中，前 3 个单文档操作的实现基本都和 Bulk 操作一致，甚至有些就是通过调用 Bulk 的接口实现的。估计接下来几个版本后，Index(Create)，Update，Delete 都会被当做 Bulk 的一种特例化操作被处理。这样，代码和逻辑都会更清晰一些。

下面，我们就以 Bulk 请求为例来介绍写入流程。

## **Elasticsearch 写入流程图**

![](https://pic2.zhimg.com/v2-4e32cd77e69ae4932665d110d6bf13a1_r.jpg)

*   红色：Client Node。
*   绿色：Primary Node。
*   蓝色：Replica Node。

## **注册 Action**

在 Elasticsearch 中，所有 action 的入口处理方法都是注册在 ActionModule.java 中，比如 Bulk Request 有两个注册入口，分别是 Rest 和 Transport 入口：

![](https://pic1.zhimg.com/v2-7506f5d87f80ec63cbd2030b785441ec_r.jpg)

![](https://pic4.zhimg.com/v2-ac3d0e6ee82d507d38110565d121febf_r.jpg)

如果请求是 Rest 请求，则会在 RestBulkAction 中 Parse Request，构造出 BulkRequest，然后发给后面的 TransportAction 处理。

TransportShardBulkAction 的基类 TransportReplicationAction 中注册了对 Primary，Replica 等的不同处理入口:

![](https://pic4.zhimg.com/v2-4ac69cd72079de47adfb2d81906579db_r.jpg)

这里对原始请求，Primary Node 请求和 Replica Node 请求各自注册了一个 handler 处理入口。

## **Client Node**

Client Node 也包括了前面说过的 Parse Request，这里就不再赘述了，接下来看一下其他的部分。

**1. Ingest Pipeline**

在这一步可以对原始文档做一些处理，比如 HTML 解析，自定义的处理，具体处理逻辑可以通过插件来实现。在 Elasticsearch 中，由于 Ingest Pipeline 会比较耗费 CPU 等资源，可以设置专门的 Ingest Node，专门用来处理 Ingest Pipeline 逻辑。

如果当前 Node 不能执行 Ingest Pipeline，则会将请求发给另一台可以执行 Ingest Pipeline 的 Node。

**2. Auto Create Index**

判断当前 Index 是否存在，如果不存在，则需要自动创建 Index，这里需要和 Master 交互。也可以通过配置关闭自动创建 Index 的功能。

**3. Set Routing**

设置路由条件，如果 Request 中指定了路由条件，则直接使用 Request 中的 Routing，否则使用 Mapping 中配置的，如果 Mapping 中无配置，则使用默认的_id 字段值。

在这一步中，如果没有指定_id 字段，则会自动生成一个唯一的_id 字段，目前使用的是 UUID。_

_**4. Construct BulkShardRequest**_

_由于 Bulk Request 中会包括多个 (Index/Update/Delete) 请求，这些请求根据 routing 可能会落在多个 Shard 上执行，这一步会按 Shard 挑拣 Single Write Request，同一个 Shard 中的请求聚集在一起，构建 BulkShardRequest，每个 BulkShardRequest 对应一个 Shard。_

_**5. Send Request To Primary**_

_这一步会将每一个 BulkShardRequest 请求发送给相应 Shard 的 Primary Node。_

## **Primary Node**

Primary 请求的入口是在 PrimaryOperationTransportHandler 的 messageReceived，我们来看一下相关的逻辑流程。

**1. Index or Update or Delete**

循环执行每个 Single Write Request，对于每个 Request，根据操作类型 (_CREATE/INDEX/UPDATE/DELETE_) 选择不同的处理逻辑。

其中，Create/Index 是直接新增 Doc，Delete 是直接根据_id 删除 Doc，Update 会稍微复杂些，我们下面就以 Update 为例来介绍。

**2. Translate Update To Index or Delete**

这一步是 Update 操作的特有步骤，在这里，会将 Update 请求转换为 Index 或者 Delete 请求。首先，会通过 GetRequest 查询到已经存在的同_id Doc（如果有）的完整字段和值（依赖_source 字段），然后和请求中的 Doc 合并。同时，这里会获取到读到的 Doc 版本号，记做 V1。

**3. Parse Doc**

这里会解析 Doc 中各个字段。生成 ParsedDocument 对象，同时会生成 uid Term。在 Elasticsearch 中，_uid = type # _id，对用户，_Id 可见，而 Elasticsearch 中存储的是_uid。这一部分生成的 ParsedDocument 中也有 Elasticsearch 的系统字段，大部分会根据当前内容填充，部分未知的会在后面继续填充 ParsedDocument。

**4. Update Mapping**

Elasticsearch 中有个自动更新 Mapping 的功能，就在这一步生效。会先挑选出 Mapping 中未包含的新 Field，然后判断是否运行自动更新 Mapping，如果允许，则更新 Mapping。

**5. Get Sequence Id and Version**

由于当前是 Primary Shard，则会从 SequenceNumber Service 获取一个 sequenceID 和 Version。SequenceID 在 Shard 级别每次递增 1，SequenceID 在写入 Doc 成功后，会用来初始化 LocalCheckpoint。Version 则是根据当前 Doc 的最大 Version 递增 1。

**6. Add Doc To Lucene**

这一步开始的时候会给特定_uid 加锁，然后判断该_uid 对应的 Version 是否等于之前 Translate Update To Index 步骤里获取到的 Version，如果不相等，则说明刚才读取 Doc 后，该 Doc 发生了变化，出现了版本冲突，这时候会抛出一个 VersionConflict 的异常，该异常会在 Primary Node 最开始处捕获，重新从 “Translate Update To Index or Delete” 开始执行。

如果 Version 相等，则继续执行，如果已经存在同 id 的 Doc，则会调用 Lucene 的 UpdateDocument(uid, doc) 接口，先根据 uid 删除 Doc，然后再 Index 新 Doc。如果是首次写入，则直接调用 Lucene 的 AddDocument 接口完成 Doc 的 Index，AddDocument 也是通过 UpdateDocument 实现。

这一步中有个问题是，如何保证 Delete-Then-Add 的原子性，怎么避免中间状态时被 Refresh？答案是在开始 Delete 之前，会加一个 Refresh Lock，禁止被 Refresh，只有等 Add 完后释放了 Refresh Lock 后才能被 Refresh，这样就保证了 Delete-Then-Add 的原子性。

Lucene 的 UpdateDocument 接口中就只是处理多个 Field，会遍历每个 Field 逐个处理，处理顺序是 invert index，store field，doc values，point dimension，后续会有文章专门介绍 Lucene 中的写入。

**7. Write Translog**

写完 Lucene 的 Segment 后，会以 keyvalue 的形式写 TransLog，Key 是_id，Value 是 Doc 内容。当查询的时候，如果请求是 GetDocByID，则可以直接根据_id 从 TransLog 中读取到，满足 NoSQL 场景下的实时性要去。

需要注意的是，这里只是写入到内存的 TransLog，是否 Sync 到磁盘的逻辑还在后面。

这一步的最后，会标记当前 SequenceID 已经成功执行，接着会更新当前 Shard 的 LocalCheckPoint。

**8. Renew Bulk Request**

这里会重新构造 Bulk Request，原因是前面已经将 UpdateRequest 翻译成了 Index 或 Delete 请求，则后续所有 Replica 中只需要执行 Index 或 Delete 请求就可以了，不需要再执行 Update 逻辑，一是保证 Replica 中逻辑更简单，性能更好，二是保证同一个请求在 Primary 和 Replica 中的执行结果一样。

**9. Flush Translog**

这里会根据 TransLog 的策略，选择不同的执行方式，要么是立即 Flush 到磁盘，要么是等到以后再 Flush。Flush 的频率越高，可靠性越高，对写入性能影响越大。

**10. Send Requests To Replicas**

这里会将刚才构造的新的 Bulk Request 并行发送给多个 Replica，然后等待 Replica 的返回，这里需要等待所有 Replica 返回后（可能有成功，也有可能失败），Primary Node 才会返回用户。如果某个 Replica 失败了，则 Primary 会给 Master 发送一个 Remove Shard 请求，要求 Master 将该 Replica Shard 从可用节点中移除。

这里，同时会将 SequenceID，PrimaryTerm，GlobalCheckPoint 等传递给 Replica。

发送给 Replica 的请求中，Action Name 等于原始 ActionName + [R]，这里的 R 表示 Replica。通过这个 [R] 的不同，可以找到处理 Replica 请求的 Handler。

**11. Receive Response From Replicas**

Replica 中请求都处理完后，会更新 Primary Node 的 LocalCheckPoint。

## **Replica Node**

Replica 请求的入口是在 ReplicaOperationTransportHandler 的 messageReceived，我们来看一下相关的逻辑流程。

**1. Index or Delete**

根据请求类型是 Index 还是 Delete，选择不同的执行逻辑。这里没有 Update，是因为在 Primary Node 中已经将 Update 转换成了 Index 或 Delete 请求了。

**2. Parse Doc**

**3. Update Mapping**

以上都和 Primary Node 中逻辑一致。

**4. Get Sequence Id and Version**

Primary Node 中会生成 Sequence ID 和 Version，然后放入 ReplicaRequest 中，这里只需要从 Request 中获取到就行。

**5. Add Doc To Lucene**

由于已经在 Primary Node 中将部分 Update 请求转换成了 Index 或 Delete 请求，这里只需要处理 Index 和 Delete 两种请求，不再需要处理 Update 请求了。比 Primary Node 会更简单一些。

**6. Write Translog**

**7. Flush Translog**

以上都和 Primary Node 中逻辑一致。

## **最后**

上面详细介绍了 Elasticsearch 的写入流程及其各个流程的工作机制，我们在这里再次总结下之前提出的分布式系统中的六大特性：

1.  可靠性：由于 Lucene 的设计中不考虑可靠性，在 Elasticsearch 中通过 Replica 和 TransLog 两套机制保证数据的可靠性。
2.  一致性：Lucene 中的 Flush 锁只保证 Update 接口里面 Delete 和 Add 中间不会 Flush，但是 Add 完成后仍然有可能立即发生 Flush，导致 Segment 可读。这样就没法保证 Primary 和所有其他 Replica 可以同一时间 Flush，就会出现查询不稳定的情况，这里只能实现最终一致性。
3.  原子性：Add 和 Delete 都是直接调用 Lucene 的接口，是原子的。当部分更新时，使用 Version 和锁保证更新是原子的。
4.  隔离性：仍然采用 Version 和局部锁来保证更新的是特定版本的数据。
5.  实时性：使用定期 Refresh Segment 到内存，并且 Reopen Segment 方式保证搜索可以在较短时间（比如 1 秒）内被搜索到。通过将未刷新到磁盘数据记入 TransLog，保证对未提交数据可以通过 ID 实时访问到。
6.  性能：性能是一个系统性工程，所有环节都要考虑对性能的影响，在 Elasticsearch 中，在很多地方的设计都考虑到了性能，一是不需要所有 Replica 都返回后才能返回给用户，只需要返回特定数目的就行；二是生成的 Segment 现在内存中提供服务，等一段时间后才刷新到磁盘，Segment 在内存这段时间的可靠性由 TransLog 保证；三是 TransLog 可以配置为周期性的 Flush，但这个会给可靠性带来伤害；四是每个线程持有一个 Segment，多线程时相互不影响，相互独立，性能更好；五是系统的写入流程对版本依赖较重，读取频率较高，因此采用了 versionMap，减少热点数据的多次磁盘 IO 开销。Lucene 中针对性能做了大量的优化。后面我们也会有文章专门介绍 Lucene 中的优化思路。

到此，Elasticsearch 的写入流程介绍完了，下一篇《Elasticsearch 读取流程》中再见。

最后，我们在招人，JAVA / Elasticsearch / Lucene 研发，有兴趣的可以私信联系我。