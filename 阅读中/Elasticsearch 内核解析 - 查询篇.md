在上一篇《[Elasticsearch 内核解析 - 写入篇](https://zhuanlan.zhihu.com/p/34669354)》中，我们介绍了 Elasticsearch 的写入流程。这一篇，我们会介绍 Elasticsearch 查询流程。

我们仍然先从 Elasticsearch 的两个身份：NoSQL 和 Search 领域的查询区别说起。

## 读操作

实时性和《[Elasticsearch 内核解析 - 写入篇](https://zhuanlan.zhihu.com/p/34669354)》中的 “写操作” 一样，对于搜索而言是近实时的，延迟在 100ms 以上，对于 NoSQL 则需要是实时的。

一致性指的是写入成功后，下次读操作一定要能读取到最新的数据。对于搜索，这个要求会低一些，可以有一些延迟。但是对于 NoSQL 数据库，则一般要求最好是强一致性的。

结果匹配上，NoSQL 作为数据库，查询过程中只有符合不符合两种情况，而搜索里面还有是否相关，类似于 NoSQL 的结果只能是 0 或 1，而搜索里面可能会有 0.1，0.5，0.9 等部分匹配或者更相关的情况。

结果召回上，搜索一般只需要召回最满足条件的 Top N 结果即可，而 NoSQL 一般都需要返回满足条件的所有结果。

搜索系统一般都是两阶段查询，第一个阶段查询到对应的 Doc ID，也就是 PK；第二阶段再通过 Doc ID 去查询完整文档，而 NoSQL 数据库一般是一阶段就返回结果。在 Elasticsearch 中两种都支持。

目前 NoSQL 的查询，聚合、分析和统计等功能上都是要比搜索弱的。

## Lucene 的读

Elasticsearch 使用了 Lucene 作为搜索引擎库，通过 Lucene 完成特定字段的搜索等功能，在 Lucene 中这个功能是通过 IndexSearcher 的下列接口实现的：

```
public TopDocs search(Query query, int n);
public Document doc(int docID);
public int count(Query query);
......(其他)
```

第一个 search 接口实现搜索功能，返回最满足 Query 的 N 个结果；第二个 doc 接口通过 doc id 查询 Doc 内容；第三个 count 接口通过 Query 获取到命中数。

这三个功能是搜索中的最基本的三个功能点，对于大部分 Elasticsearch 中的查询都是比较复杂的，直接用这个接口是无法满足需求的，比如分布式问题。这些问题都留给了 Elasticsearch 解决，我们接下来看 Elasticsearch 中相关读功能的剖析。

## Elasticsearch 的读

Elasticsearch 中每个 Shard 都会有多个 Replica，主要是为了保证数据可靠性，除此之外，还可以增加读能力，因为写的时候虽然要写大部分 Replica Shard，但是查询的时候只需要查询 Primary 和 Replica 中的任何一个就可以了。

![](https://pic1.zhimg.com/v2-1ad1351408bdf0ce7f76f251d6ef8bc4_r.jpg)

在上图中，该 Shard 有 1 个 Primary 和 2 个 Replica Node，当查询的时候，从三个节点中根据 Request 中的 preference 参数选择一个节点查询。preference 可以设置_local，_primary，_replica 以及其他选项。如果选择了 primary，则每次查询都是直接查询 Primary，可以保证每次查询都是最新的。如果设置了其他参数，那么可能会查询到 R1 或者 R2，这时候就有可能查询不到最新的数据。

上述代码逻辑在 OperationRouting.Java 的 searchShards 方法中。

接下来看一下，Elasticsearch 中的查询是如何支持分布式的。

![](https://pic1.zhimg.com/v2-737f6cb48ccf22c50c2e630433c6ad48_r.jpg)

Elasticsearch 中通过分区实现分布式，数据写入的时候根据_routing 规则将数据写入某一个 Shard 中，这样就能将海量数据分布在多个 Shard 以及多台机器上，已达到分布式的目标。这样就导致了查询的时候，潜在数据会在当前 index 的所有的 Shard 中，所以 Elasticsearch 查询的时候需要查询所有 Shard，同一个 Shard 的 Primary 和 Replica 选择一个即可，查询请求会分发给所有 Shard，每个 Shard 中都是一个独立的查询引擎，比如需要返回 Top 10 的结果，那么每个 Shard 都会查询并且返回 Top 10 的结果，然后在 Client Node 里面会接收所有 Shard 的结果，然后通过优先级队列二次排序，选择出 Top 10 的结果返回给用户。

这里有一个问题就是请求膨胀，用户的一个搜索请求在 Elasticsearch 内部会变成 Shard 个请求，这里有个优化点，虽然是 Shard 个请求，但是这个 Shard 个数不一定要是当前 Index 中的 Shard 个数，只要是当前查询相关的 Shard 即可，这个需要基于业务和请求内容优化，通过这种方式可以优化请求膨胀数。

Elasticsearch 中的查询主要分为两类，Get 请求：通过 ID 查询特定 Doc；Search 请求：通过 Query 查询匹配 Doc。

![](https://pic2.zhimg.com/v2-1f4c1cf921049b841ae2612b4734cdb1_r.jpg)

上图中内存中的 Segment 是指刚 Refresh Segment，但是还没持久化到磁盘的新 Segment，而非从磁盘加载到内存中的 Segment。

对于 Search 类请求，查询的时候是一起查询内存和磁盘上的 Segment，最后将结果合并后返回。这种查询是近实时（Near Real Time）的，主要是由于内存中的 Index 数据需要一段时间后才会刷新为 Segment。

对于 Get 类请求，查询的时候是先查询内存中的 TransLog，如果找到就立即返回，如果没找到再查询磁盘上的 TransLog，如果还没有则再去查询磁盘上的 Segment。这种查询是实时（Real Time）的。这种查询顺序可以保证查询到的 Doc 是最新版本的 Doc，这个功能也是为了保证 NoSQL 场景下的实时性要求。

![](https://pic1.zhimg.com/v2-c5455432442548d1f12c975684fc4a00_r.jpg)

所有的搜索系统一般都是两阶段查询，第一阶段查询到匹配的 DocID，第二阶段再查询 DocID 对应的完整文档，这种在 Elasticsearch 中称为 query_then_fetch，还有一种是一阶段查询的时候就返回完整 Doc，在 Elasticsearch 中称作 query_and_fetch，一般第二种适用于只需要查询一个 Shard 的请求。

除了一阶段，两阶段外，还有一种三阶段查询的情况。搜索里面有一种算分逻辑是根据 TF（Term Frequency）和 DF（Document Frequency）计算基础分，但是 Elasticsearch 中查询的时候，是在每个 Shard 中独立查询的，每个 Shard 中的 TF 和 DF 也是独立的，虽然在写入的时候通过_routing 保证 Doc 分布均匀，但是没法保证 TF 和 DF 均匀，那么就有会导致局部的 TF 和 DF 不准的情况出现，这个时候基于 TF、DF 的算分就不准。为了解决这个问题，Elasticsearch 中引入了 DFS 查询，比如 DFS_query_then_fetch，会先收集所有 Shard 中的 TF 和 DF 值，然后将这些值带入请求中，再次执行 query_then_fetch，这样算分的时候 TF 和 DF 就是准确的，类似的有 DFS_query_and_fetch。这种查询的优势是算分更加精准，但是效率会变差。另一种选择是用 BM25 代替 TF/DF 模型。

在新版本 Elasticsearch 中，用户没法指定 DFS_query_and_fetch 和 query_and_fetch，这两种只能被 Elasticsearch 系统改写。

## Elasticsearch 查询流程

Elasticsearch 中的大部分查询，以及核心功能都是 Search 类型查询，上面我们了解到查询分为一阶段，二阶段和三阶段，这里我们就以最常见的的二阶段查询为例来介绍查询流程。

![](https://pic3.zhimg.com/v2-10acab5576a2359ca279331e81adc1e2_r.jpg)

**注册 Action**

Elasticsearch 中，查询和写操作一样都是在 ActionModule.java 中注册入口处理函数的。

```
registerHandler.accept(new RestSearchAction(settings, restController));
......
actions.register(SearchAction.INSTANCE, TransportSearchAction.class);
......
```

如果请求是 Rest 请求，则会在 RestSearchAction 中解析请求，检查查询类型，不能设置为 dfs_query_and_fetch 或者 query_and_fetch，这两个目前只能用于 Elasticsearch 中的优化场景，然后将请求发给后面的 TransportSearchAction 处理。然后构造 SearchRequest，将请求发送给 TransportSearchAction 处理。

![](https://pic1.zhimg.com/v2-02fdc9375a9d6af62ac6c2035b5a9730_r.jpg)

如果是第一阶段的 Query Phase 请求，则会调用 SearchService 的 executeQueryPhase 方法。

![](https://pic3.zhimg.com/v2-be3a37c7fe08d4f1559b9eb0aaa8d37a_r.jpg)

如果是第二阶段的 Fetch Phase 请求，则会调用 SearchService 的 executeFetchPhase 方法。

## **Client Node**

Client Node 也包括了前面说过的 Parse Request，这里就不再赘述了，接下来看一下其他的部分。

**1. Get Remove Cluster Shard**

判断是否需要跨集群访问，如果需要，则获取到要访问的 Shard 列表。

**2. Get Search Shard Iterator**

获取当前 Cluster 中要访问的 Shard，和上一步中的 Remove Cluster Shard 合并，构建出最终要访问的完整 Shard 列表。

这一步中，会根据 Request 请求中的参数从 Primary Node 和多个 Replica Node 中选择出一个要访问的 Shard。

**3. For Every Shard:Perform**

遍历每个 Shard，对每个 Shard 执行后面逻辑。

**4. Send Request To Query Shard**

将查询阶段请求发送给相应的 Shard。

**5. Merge Docs**

上一步将请求发送给多个 Shard 后，这一步就是异步等待返回结果，然后对结果合并。这里的合并策略是维护一个 Top N 大小的优先级队列，每当收到一个 shard 的返回，就把结果放入优先级队列做一次排序，直到所有的 Shard 都返回。

翻页逻辑也是在这里，如果需要取 Top 30~ Top 40 的结果，这个的意思是所有 Shard 查询结果中的第 30 到 40 的结果，那么在每个 Shard 中无法确定最终的结果，每个 Shard 需要返回 Top 40 的结果给 Client Node，然后 Client Node 中在 merge docs 的时候，计算出 Top 40 的结果，最后再去除掉 Top 30，剩余的 10 个结果就是需要的 Top 30~ Top 40 的结果。

上述翻页逻辑有一个明显的缺点就是每次 Shard 返回的数据中包括了已经翻过的历史结果，如果翻页很深，则在这里需要排序的 Docs 会很多，比如 Shard 有 1000，取第 9990 到 10000 的结果，那么这次查询，Shard 总共需要返回 1000 * 10000，也就是一千万 Doc，这种情况很容易导致 OOM。

另一种翻页方式是使用 search_after，这种方式会更轻量级，如果每次只需要返回 10 条结构，则每个 Shard 只需要返回 search_after 之后的 10 个结果即可，返回的总数据量只是和 Shard 个数以及本次需要的个数有关，和历史已读取的个数无关。这种方式更安全一些，推荐使用这种。

如果有 aggregate，也会在这里做聚合，但是不同的 aggregate 类型的 merge 策略不一样，具体的可以在后面的 aggregate 文章中再介绍。

**6. Send Request To Fetch Shard**

选出 Top N 个 Doc ID 后发送给这些 Doc ID 所在的 Shard 执行 Fetch Phase，最后会返回 Top N 的 Doc 的内容。

## Query Phase

接下来我们看第一阶段查询的步骤：

**1. Create Search Context**

创建 Search Context，之后 Search 过程中的所有中间状态都会存在 Context 中，这些状态总共有 50 多个，具体可以查看 DefaultSearchContext 或者其他 SearchContext 的子类。

**2. Parse Query**

解析 Query 的 Source，将结果存入 Search Context。这里会根据请求中 Query 类型的不同创建不同的 Query 对象，比如 TermQuery、FuzzyQuery 等，最终真正执行 TermQuery、FuzzyQuery 等语义的地方是在 Lucene 中。

这里包括了 dfsPhase、queryPhase 和 fetchPhase 三个阶段的 preProcess 部分，只有 queryPhase 的 preProcess 中有执行逻辑，其他两个都是空逻辑，执行完 preProcess 后，所有需要的参数都会设置完成。

由于 Elasticsearch 中有些请求之间是相互关联的，并非独立的，比如 scroll 请求，所以这里同时会设置 Context 的生命周期。

同时会设置 lowLevelCancellation 是否打开，这个参数是集群级别配置，同时也能动态开关，打开后会在后面执行时做更多的检测，检测是否需要停止后续逻辑直接返回。

**3. Get From Cache**

判断请求是否允许被 Cache，如果允许，则检查 Cache 中是否已经有结果，如果有则直接读取 Cache，如果没有则继续执行后续步骤，执行完后，再将结果加入 Cache。

**4. Add Collectors**

Collector 主要目标是收集查询结果，实现排序，对自定义结果集过滤和收集等。这一步会增加多个 Collectors，多个 Collector 组成一个 List。

1.  FilteredCollector_：_先判断请求中是否有 Post Filter，Post Filter 用于 Search，Agg 等结束后再次对结果做 Filter，希望 Filter 不影响 Agg 结果。如果有 Post Filter 则创建一个 FilteredCollector，加入 Collector List 中。
2.  PluginInMultiCollector：判断请求中是否制定了自定义的一些 Collector，如果有，则创建后加入 Collector List。
3.  MinimumScoreCollector：判断请求中是否制定了最小分数阈值，如果指定了，则创建 MinimumScoreCollector 加入 Collector List 中，在后续收集结果时，会过滤掉得分小于最小分数的 Doc。
4.  EarlyTerminatingCollector：判断请求中是否提前结束 Doc 的 Seek，如果是则创建 EarlyTerminatingCollector，加入 Collector List 中。在后续 Seek 和收集 Doc 的过程中，当 Seek 的 Doc 数达到 Early Terminating 后会停止 Seek 后续倒排链。
5.  CancellableCollector：判断当前操作是否可以被中断结束，比如是否已经超时等，如果是会抛出一个 TaskCancelledException 异常。该功能一般用来提前结束较长的查询请求，可以用来保护系统。
6.  EarlyTerminatingSortingCollector：如果 Index 是排序的，那么可以提前结束对倒排链的 Seek，相当于在一个排序递减链表上返回最大的 N 个值，只需要直接返回前 N 个值就可以了。这个 Collector 会加到 Collector List 的头部。EarlyTerminatingSorting 和 EarlyTerminating 的区别是，EarlyTerminatingSorting 是一种对结果无损伤的优化，而 EarlyTerminating 是有损的，人为掐断执行的优化。
7.  TopDocsCollector：这个是最核心的 Top N 结果选择器，会加入到 Collector List 的头部。TopScoreDocCollector 和 TopFieldCollector 都是 TopDocsCollector 的子类，TopScoreDocCollector 会按照固定的方式算分，排序会按照分数 + doc id 的方式排列，如果多个 doc 的分数一样，先选择 doc id 小的文档。而 TopFieldCollector 则是根据用户指定的 Field 的值排序。

**5. lucene::search**

这一步会调用 Lucene 中 IndexSearch 的 search 接口，执行真正的搜索逻辑。每个 Shard 中会有多个 Segment，每个 Segment 对应一个 LeafReaderContext，这里会遍历每个 Segment，到每个 Segment 中去 Search 结果，然后计算分数。

搜索里面一般有两阶段算分，第一阶段是在这里算的，会对每个 Seek 到的 Doc 都计算分数，为了减少 CPU 消耗，一般是算一个基本分数。这一阶段完成后，会有个排序。然后在第二阶段，再对 Top 的结果做一次二阶段算分，在二阶段算分的时候会考虑更多的因子。二阶段算分在后续操作中。

具体请求，比如 TermQuery、WildcardQuery 的查询逻辑都在 Lucene 中，后面会有专门文章介绍。

**6. rescore**

根据 Request 中是否包含 rescore 配置决定是否进行二阶段排序，如果有则执行二阶段算分逻辑，会考虑更多的算分因子。二阶段算分也是一种计算机中常见的多层设计，是一种资源消耗和效率的折中。

Elasticsearch 中支持配置多个 Rescore，这些 rescore 逻辑会顺序遍历执行。每个 rescore 内部会先按照请求参数 window 选择出 Top window 的 doc，然后对这些 doc 排序，排完后再合并回原有的 Top 结果顺序中。

**7. suggest::execute()**

如果有推荐请求，则在这里执行推荐请求。如果请求中只包含了推荐的部分，则很多地方可以优化。推荐不是今天的重点，这里就不介绍了，后面有机会再介绍。

**8. aggregation::execute()**

如果含有聚合统计请求，则在这里执行。Elasticsearch 中的 aggregate 的处理逻辑也类似于 Search，通过多个 Collector 来实现。在 Client Node 中也需要对 aggregation 做合并。aggregate 逻辑更复杂一些，就不在这里赘述了，后面有需要就再单独开文章介绍。

上述逻辑都执行完成后，如果当前查询请求只需要查询一个 Shard，那么会直接在当前 Node 执行 Fetch Phase。

## Fetch Phase

Elasticsearch 作为搜索系统时，或者任何搜索系统中，除了 Query 阶段外，还会有一个 Fetch 阶段，这个 Fetch 阶段在数据库类系统中是没有的，是搜索系统中额外增加的阶段。搜索系统中额外增加 Fetch 阶段的原因是搜索系统中数据分布导致的，在搜索中，数据通过 routing 分 Shard 的时候，只能根据一个主字段值来决定，但是查询的时候可能会根据其他非主字段查询，那么这个时候所有 Shard 中都可能会存在相同非主字段值的 Doc，所以需要查询所有 Shard 才能不会出现结果遗漏。同时如果查询主字段，那么这个时候就能直接定位到 Shard，就只需要查询特定 Shard 即可，这个时候就类似于数据库系统了。另外，数据库中的二级索引又是另外一种情况，但类似于查主字段的情况，这里就不多说了。

基于上述原因，第一阶段查询的时候并不知道最终结果会在哪个 Shard 上，所以每个 Shard 中管都需要查询完整结果，比如需要 Top 10，那么每个 Shard 都需要查询当前 Shard 的所有数据，找出当前 Shard 的 Top 10，然后返回给 Client Node。如果有 100 个 Shard，那么就需要返回 100 * 10 = 1000 个结果，而 Fetch Doc 内容的操作比较耗费 IO 和 CPU，如果在第一阶段就 Fetch Doc，那么这个资源开销就会非常大。所以，一般是当 Client Node 选择出最终 Top N 的结果后，再对最终的 Top N 读取 Doc 内容。通过增加一点网络开销而避免大量 IO 和 CPU 操作，这个折中是非常划算的。

Fetch 阶段的目的是通过 DocID 获取到用户需要的完整 Doc 内容。这些内容包括了 DocValues，Store，Source，Script 和 Highlight 等，具体的功能点是在 SearchModule 中注册的，系统默认注册的有：

*   ExplainFetchSubPhase
*   DocValueFieldsFetchSubPhase
*   ScriptFieldsFetchSubPhase
*   FetchSourceSubPhase
*   VersionFetchSubPhase
*   MatchedQueriesFetchSubPhase
*   HighlightPhase
*   ParentFieldSubFetchPhase

除了系统默认的 8 种外，还有通过插件的形式注册自定义的功能，这些 SubPhase 中最重要的是 Source 和 Highlight，Source 是加载原文，Highlight 是计算高亮显示的内容片断。

上述多个 SubPhase 会针对每个 Doc 顺序执行，可能会产生多次的随机 IO，这里会有一些优化方案，但是都是针对特定场景的，不具有通用性。

Fetch Phase 执行完后，整个查询流程就结束了。

## 总结

Elasticsearch 中的查询流程比较简单，更多的查询原理都在 Lucene 中，后续我们会有针对不同请求的 Lucene 原理介绍性文章。

【转载自云栖社区】