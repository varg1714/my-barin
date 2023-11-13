---
create: 2023-11-13 01:19
---
Elasticsearch 是一款优秀的开源企业级搜索引擎，其查询接口主要为 Search 接口，提供了丰富的各类查询、排序、统计聚合等功能。本文将要介绍的是另一个查询接口 SearchScroll，同时介绍一下我们在这方面做的一些性能和稳定性等方面的优化工作。  
  
Elasticsearch 的 SearchScroll 接口可用于从索引中检索大量数据，或者是所有的数据，值得注意的是 Elasticsearch 的 SearchScroll 请求不是为了用户进行实时请求，而是为了更快导出大量数据。同时该接口提供稳定的查询结果，不会因为用户一直在更新数据导致查询结果集合重复或缺失。典型场景如索引重建、将符合某一个条件的所有的数据全部导出来然后交给计算平台进行分析处理。SearchScroll 支持多 slice 进行请求，在客户端以多并发的方式进行查询，导出速度可以更快。

## 为什么需要 SearchScroll

Search 接口的功能已经足够丰富，那么为什么还需要 SearchScroll？原因就是 Search 接口的速度不够快和结果不够稳定。

### from+size

Search 接口进行翻页的方式主要有两种，一是 size+from 的翻页方式，这种翻页方式存在很大的性能瓶颈，时间复杂度 O(n)，空间复杂度 O(n)。其每次查询都需要从第 1 页翻到第 n 页，但是只有第 n 页的数据需要返回给用户。那么之前 n-1 页都是做的无用功。如果翻的更深，那么消耗的系统资源更是翻倍增长，很容易出现 OOM，系统各项指标出现异常。举个例子，假设每个文档在协调节点进行 merge 的 ScoreDoc 需要 16 字节，那么翻到一亿条时候，需要 1.6G 的内存，如果多来几个并发，普通用户的计算机根本扛不住这么大的内存开销。因此，很多产品在功能上直接禁止用户深度翻页来避免这种技术难题。

### SearchAfter

Search 接口另一种翻页方式是 SearchAfter，时间复杂度 O(n)，空间复杂度 O(1)。SearchAfter 是一种动态指针的技术，每次查询都会携带上一次的排序值，这样下次取结果只需要从上次的位点继续扫数据，前提条件也是该字段是数值类型且设置了 docValue。举个例子，假设 "val_1" 是数值类型的字段，然后使用 Search 接口查询时候添加 Sort("val_1")，那么 response 中可以拿到最后一条数据的 "val_1" 的值，，也就是 response 中 sort 字段的值，然后下次查询将该值放在 query 中的 searchAfter 参数中，下次查询就可以在上一次结果之后继续查询，如此反复，最后可以翻页很深，内存消耗相比 size+from 的方式降低了数倍。该方式效果类似于我们直接在 bool 查询中主动加一个 rangeFilter，可以达到类似的效果。表面看这种方案能将查询速度降到 O(1) 的复杂度，实际上其内部还是会扫 sort 字段的 docValue，翻页越深，则扫 docValve 越多，因此复杂度和翻页深度成正比，越往后查询越慢，但是相比 size+from 的方式，至少可以完成深度翻页的任务，不至于 OOM，速度勉强可以接受。SearchAfter 的翻页方式在性能上有了质的提升，但是其限制了用户只能一页一页往后翻，无法跳页，因此很多产品在功能设计时候是不允许跳页的，只能一页一页往后翻，也是有一定的技术原因的。

### SearchScroll

Search 接口在使用 SearchAfter 后，相比 size+from 的翻页方式，翻页性能有质的提升，但是和 SearchScroll 相比，性能逊色很多，用户需要获取的数据越多，翻的越深，则差别越大。

在查询性能上，SearchScroll 的翻页方式，时间复杂度 O(1)，空间复杂度 O(1)。SearchScroll 能够以恒定的速度翻页获取完所有数据，而采用 SearchAfter 的方式获取数据会随翻页深度增大而吞吐能力大幅下降。在我们的单机单 shard2 亿数据测试中，采用 SearchScroll 方式能够以每次 50ms 延时稳定获取完 2 亿数据，而 SearchAfter 深度翻页到千万级条数据后查询延时就到了秒级别，查询速度线性下降。

在吞吐能力上，SearchScroll 请求天然支持多并发方式查询，因此 SearchScroll 特别适合批量快速拉取大量数据，然后交给 spark 等计算平台进行后续数据分析处理。在 Elasticsearch 中把每个并发称之为一个 Slice(分片)，Elasticsearch 内部对用户的请求进行分片，分片越多则速度越快，拉取数据的速度翻倍提高。当然之前的普通的 Search 查询方式也可以并发访问，但是需要用户将 Search 请求的 query 进行拆分，比如原来是获取 1 年的数据，那么可以将 query 拆分为 12 个，一个月一个请求，体现在查询语句里就是将月份条件添加到 query 语句中的 filter 中来保证仅返回某一个月的数据。Search 查询通过拆分 query 有时候可以达到类似的并发效果，来加速 Search 查询，但是有些 query 语句是难以拆分的，使用成本较高，因此直接利用 SearchScroll 让 Elasticsearch 帮助我们进行并发拆分是一个不错的选择。

在结果稳定性上，SearchScroll 由于会 “打 snapshot”，context 会保留目前的 segments，后续写入的数据都是感知不到的，因此不会造成查到的结果中存在重复数据或者缺失数据。在批量导数据等要求结果稳定的场景下，SearchScroll 特别适用。从另一个角度讲，对需要稳定结果的用户来说是件好事，但是会导致该部分 segments 暂时无法被 merge，也会占用一些操作系统的文件句柄，因此需要留意系统的这些方面的指标，确保 Elasticsearch 系统稳定运行。  
  
总之，SearchScroll 的查询速度很快，吞吐能力很高，结果很稳定。

## 原理剖析

本节主要简单介绍 SearchScroll 的流程和 SearchScroll 的并发原理。

### 流程剖析

使用 SearchScroll 功能，用户的请求主要分为两个阶段，我们将第一阶段称之为 Search 阶段，第二阶段称之为 Scroll 阶段。如下图所示。

![](https://pic2.zhimg.com/v2-8c4505f97a6121d28fd828b8184371a1_r.jpg)

其中第一阶段和传统的 Search 请求流程几乎一致，在 Search 流程的基础上进行了一些额外的特殊处理，比如 Slice 并发处理、Context 上下文保留、Response 中返回 scroll_id、记录本次的游标地址方便下一次 scroll 请求继续获取数据等等。

第二阶段 Scroll 请求则大大简化，Search 中的许多流程都不要再次进行，仅需要执行 query、fetch、response 三个阶段。而完整的 search 请求包含 rewrite、can_match、dfs、query、fetch、dfs_query、expand、response 等复杂的流程，因此其在 es 的代码实现中也没有严格遵循上述的流程流转的框架，也没有 SearchPhaseContext 等 context 实现。

### Search 阶段

第一个阶段是 Search 的流程，其中在 [Elasticsearch 内核解析 - 查询篇](https://zhuanlan.zhihu.com/p/34674517) 有详细的介绍。这里按照查询流程，仅介绍一些不同的地方。

### CreateContext

创建 SearchContex 后，如果是 scroll 请求，则在 searchContext 中设置 ScrollContext。ScrollContext 中主要包含 context 的有效时间、上一次访问了哪个文档 lastEmittedDoc（即游标位置）等信息。具体如下：

```
private Map<String, Object> context = null;
    public long totalHits = -1;
    public float maxScore;
    public ScoreDoc lastEmittedDoc;
    public Scroll scroll;
```

queryPhase.preProcess 中会处理 sliceFilter，判断该 slice 请求到达哪个 shard。这里是进行 slice 并发请求核心处理逻辑，简单来说根据 slice 的 id 和 shard_id 是否匹配来判断是否在本 shard 上进行请求。然后将 query 进行重写，将用户原有的 query 放入到 boolQuery 的 must 中，slice 构建出的 filter 放入 boolQuery 的 filter 中。  
  
SearchScroll 通过 SearchContext 保留上下文。每个 context 都有一个 id，它是单机原子自增的，后续如果还需要使用则可以根据 id 拿到该 context。context 会自动清理，默认 5 分钟的 keepAlive，新来的请求会刷新 keepAlive，或者通过 clearScroll 来主动清除该 context。

### LoadOrExecuteQueryPhase

SearchScroll 请求结果永远不会被 cache，判断条件很简单，如果请求中携带了 scroll 参数，这一步会直接跳过。

### QueryPhase.execute

该步骤为 search 查询的核心逻辑，search 请求携带 scroll 和不携带 scroll 在这里几乎是一模一样的，具体参考上述链接的文章介绍。

### FetchSearchPhase

fetch 阶段，需要将 query 阶段返回的 doc_id 进行 fetch 其 doc 内容。  
  
如果是 scroll 类型的 search 请求，则需要 buildScrollId，scrollid 中保存了一个数组，每个元素包含 2 个值：

*   nodeid，下次请求知道上一次请求在哪个 shard 上进行的。
*   RequestId(ContextId)，找到上一次请求对应的 searchContext，方便进行下一次请求。

fetch 结束的时候，需要将本次请求发给用户的最后一个元素的排序字段的值的大小保留下来，这个值是哪个字段取决于 search 请求中的 sort 设置了什么值。elasticsearch 推荐使用_doc 进行排序，这样性能最好。当获取到最后一个文档后，需要更新到 searchContext 中的 ScrollContext 的 lastEmittedDoc 值，这样下次请求就知道从哪里开始进行搜索了。

### 小结

总结一下 Search 和 Scroll 的核心区别，主要是在 query 阶段需要处理并发的 scroll 请求 (slice)，fetch 阶段需要得到本次返回给用户的最后一个文档 lastEmittedDoc，然后告知 data 节点的 context，这样下次请求就可以继续从上一个记录点进行搜索。

![](https://pic3.zhimg.com/v2-3e73d11312b5370673288bf1f912bb12_r.jpg)

### Scroll 阶段

该阶段是在 elasticsearch 中是通过调用 SearchScrollRequest 发起请求，其参数主要有两个：

1.  scroll_id，方便在 data 节点上找到对应的 context，继续上一次的请求。
2.  scroll 失效时间，即刷新 context 的 aliveTime，aliveTime 过后该 context 失效。这个参数一般使用不多，使用默认值即可。

该阶段从 api 层面来看已经区别很大，一个是 SearchRequest，另一个是 SearchScrollRequest。search 的流程上面主要是分析了一些不同的地方，接下来讲一下 scroll 的流程，只有 query、fetch、response 三个 phase，其中 response 仅仅是拼装和返回数据，这里略过。

### query

1.  在协调节点上，将 scroll_id 进行 parse，得到本次请求的目标 shard 和对应 shard 上的 searchContext 的 id，将这两个参数通过 InternalScrollSearchRequest 请求转发到 data 节点上。
2.  在 data 节点上，从内存中获取到对应的 searchContext，即获取到了用户原来的 query 和上次游标信息 lastEmittedDoc。然后再执行 QueryPhase.execute 时，会将 query 进行改写，如下代码所示。改写后将 lastEmittedDoc 放入 boolQuery 的 filter 中，这就是为什么 scroll 请求可以知道下次请求的数据应该从哪里开始。并且这个 MinDocQuery 的性能是比传统的 rangeQuery 要快很多的，它仅仅匹配 >=after.doc + 1 的文档，可以直接跳过很多无效的扫描。

```
final ScoreDoc after = scrollContext.lastEmittedDoc;
if (after != null) {
    BooleanQuery bq = new BooleanQuery.Builder()
        .add(query, BooleanClause.Occur.MUST)
        .add(new MinDocQuery(after.doc + 1), BooleanClause.Occur.FILTER)
        .build();
    query = bq;
}
// ... and stop collecting after ${size} matches
searchContext.terminateAfter(searchContext.size());
searchContext.trackTotalHits(false);
```

### fetch

1.  在协调节点上，将各个 shard 返回的数据进行排序，然后将用户想要的 size 个数据进行 fetch，这些数据同样需要得到 lastEmittedDoc， 与 Search 阶段一致，都是通过 ShardFetchRequest 告知 data 节点上 searchContext 本次的 lastEmittedDoc，并更新在 context 中供下次查询使用。
2.  在 data 节点上，如果传入的 request.lastEmittedDoc 不为空，则更新 searchContext 中的 lastEmittedDoc。

### SearchScroll 的并发原理介绍

SearchScroll 天然支持基于 shard 的并发查询，而 Search 接口想要支持并发查询，需要将 query 进行拆分，虽然也能进行并发查询，但是其背后浪费的集群资源相对较多。  
  
首先从 API 使用方式上介绍 SearchScroll 的并发，我们用一个简单的例子做说明。Slice 参数是 SearchScroll 控制并发切分的参数，id、max 是其最主要的两个参数，id 取值为 [0,max)，max 取值没有特别的限制，一般不超过 1024，但是推荐 max 取值为小于等于索引 shard 的个数。id、max 两个参数决定了后续在 data 节点如何检索数据。

```
GET /bar/_search?scroll=1m
{
    "slice": {
        "id": 0, 
        "max": 128 
    },
    "query": {
        "match" : {
            "title" : "foo"
        }
    }
}
```

SearchScroll 并发获取数据只需要我们多个线程调用 Elasticsearch 的接口即可，然后请求到达 data 节点后，开始处理 slice，如果该 slice 不应该查询本 shard，则直接返回一个 MatchNoDocsQuery 这样的 filter，然后本 shard 上的查询会迅速得到执行。如果并发数等于 shard 数，就相当于一个并发真实的查询了一个 shard。而用 Search 接口拆 query 后进行并发查询，每个并发还是会访问所有的 shard 在所有数据上进行查询，浪费集群的资源。  
  
SearchScroll 如何判定一个 slice 是否应该查询一个节点上的 shard，只需要进行简单的 hash 值判断即可。有 4 个参数 id、max、shardID、numShards(索引 shard 个数) 决定了是否会进行 MatchNoDocsQuery，具体规则如下：

*   当 max>=numShards，如果 id%numShards!=shardID，则返回 MatchNoDocsQuery
*   当 max<numShards，如果 id!=shardId%max，则返回 MatchNoDocsQuery

为什么推荐 SearchScroll 的 max 取值小于等于索引 shard 个数？简单说明就是并发数大于索引 shard 数后，需要将一个 shard 切分为多份来给多个 slice 使用，而切分单个 shard 是需要消耗一些资源的，会造成首次查询较慢，且有内存溢出风险。

首先看一下 slice 是如何切分 shard 的，规则如下：

*   numShards=1

*   直接 TermsSliceQuery 切分，单个 shard 的 slice_id 就是 TermsSliceQuery 请求的 slice_id，单个 shard 内如何切分见下方介绍。

*   max<=numShards

*   一个 slice 对应 numShards/max 个完整 shard

*   max>numShards

*   靠前的单个 shard 被分为 (max/numShards + 1) 份，后面的被分为 (max/numShards) 份
*   例如：

*   2shard 5 个 slice

*   shard0->slice0、2、4
*   shard1->slice1、3

*   5shard 8 个 slice，则

*   shard0->slice0、5
*   shard1->slice1、6
*   shard2->slice2、7
*   shard3->slice3
*   shard4->slice4

单 shard 内 slice 是根据 slice.field 参数来切分的，推荐使用_id 或者_uid 来进行切分，_uid 也是该参数的默认值。其它支持 DocValue 的 number 类型的 field 都可以进行切分。

*   根据_uid 字段进行切分，则使用 TermsSliceQuery 进行切分

*   这个 filter 是 O（N*M），其中 N 是 term 的枚举数量，M 是每个 term 出现的平均次数。
*   每个 segment 会生成一个 DocIdSet
*   首轮 Search 请求由于 score 没有 cache，需要真正的去遍历拿 docid，因此执行较慢。
*   针对每个 segment，遍历 term dictionary，计算每个 term 的 hashCode， Math._floorMod_(hashCode, slice_max) == slice_id 来决定是否放入到 DocIdSet。
*   计算 hash 值的函数：StringHelper.murmurhash3_x86_32

*   其它 DocValue 数值类型字段进行切分，则使用 DocValuesSliceQuery 进行切分

*   DocValuesSliceQuery 和 TermsSliceQuery 类似，只是没有使用_uid 作为切分，它使用了指定 field 的排序好的 SortedNumericDocValues
*   它构造出的 DocIdSet 是一个全量的 DocIdSet(DocIdSetIterator._all_)，但是在 scorer 时候有一个两阶段的过程，TwoPhaseIterator 中如果 match 才会取出，不然就指向下一个。match 中定义的逻辑和上面_uid 切分是一致的，都是根据 hash 值是否和 slice_id 对应。如果 Math._floorMod_(hashCode, slice_max) == slice_id 就拿出来，不然就跳过。
*   计算 hash 值：BitMixer.mix。该计算 hash 值的速度估计会比 string 的要快，因为实现要比 murmurhash3_x86_32 简单很多。
*   基于 DocValue 数值的注意点：

*   该字段不能更新，只能设置一次
*   该字段的分布要均匀，不然每个 slice 获取到的 docId 不均匀。

单 shard 内切分 slice 的两种方式总结：

1.  TermsSliceQuery 耗内存，可能会造成 jvm 内存紧张；DocValuesSliceQuery 不占用内存，但是依赖读 DocValue，因此速度没有 TermsSliceQuery 快。
2.  TermsSliceQuery 真实的遍历了_uid 的值，而 DocValuesSliceQuery 遍历了 doc_id 序号，根据这个 doc_id 去取 DocValue。

## 性能和稳定性优化改进

当前 Elasticsearch 在 SearchScroll 接口上有很多地方存在性能问题或者稳定性问题，我们对他们进行了一些优化和改进，让该接口性能更好和使用更佳。本节主要介绍的是我们在 SearchScroll 接口上做的一些优化的工作。

### queryAndFetch

这个优化是 Elasticsearch 目前就有的，但是还有改进的空间。  
  
当索引只有一个 shard 的时候，Elasticsearch 能够启用该优化，这时候 SearchScroll 查询能够启用 queryAndFetch 查询策略，这样在协调节点上只需要一步 queryAndFetch 操作就可以从 data 节点上拿到数据，而默认的查询策略 queryThenFetch 需要经历一个两阶段操作。如图所示，queryAndFetch 这种查询方式可以节省一次网络开销，查询时间缩短。  

![](https://pic4.zhimg.com/v2-d8a4cfc80eeef5f7cdc92ce1bdecc543_r.jpg)

![](https://pic1.zhimg.com/v2-4b5fb9ea0597d528394bf312474af7f0_r.jpg)

  
当用户的 shard 数不等于 1 时候，Elasticsearch 没有任何优化。但是，当用户的 SearchScroll 的 max 和 shard 数一致的时候，也是可以开启 queryAndFetch 优化的，因为一个并发仅仅在一个 shard 上真正的执行。我们将这些 case 也进行了优化，在多并发时候也能进行 queryAndFetch 优化，节省 CPU、网络、内存等资源消耗，提高整体吞吐率。

### 查询剪枝

SearchScroll 多并发场景下，请求刚到协调节点上，会查询出每个 shard 在哪些节点上，然后将请求转发到这些节点上。当查询请求到达 data 节点上，根据 slice 参数重写 query 时候，会判断该 shard 应不应该被当前 slice 进行查询。主要判断逻辑本文上述章节已经介绍。如果该 slice 不应该查询本 shard，则直接返回一个 MatchNoDocsQuery 这样的 filter，相当于该请求在 data 节点上浪费了一次查询。虽然加了 MatchNoDocsQuery 的原请求执行速度很快，但是会占用线程池浪费一些 cpu 时间，而且会浪费线程池的队列空间。

假如用户有 512 个 shard，且用户用 512 个并发进行访问。需要注意的是，每个并发请求都会转发到所有的 shard 上，因此在集群的 data 节点上瞬间会有 512*512=26 万个任务需要执行，其中仅有 512 个任务是真正需要执行的，其它的请求都是在浪费集群资源。默认情况下单个节点查询线程池队列是 1000，一般集群也没有那么多 data 节点，难支撑 26 万个请求。  
  
针对该问题，我们将 slice 的 MatchNoDocsQuery 的 filter 过滤提前到协调节点，不需要再转发这些无用的请求。在协调节点上会计算哪些 shard 需要真正执行查询任务，因此我们将 MatchNoDocsQuery 的 filter 逻辑前置，达到查询剪枝的目的。

除此之外，在并发数和 shard 数不相等时候，一个并发请求可能会发送到 n 个 shard 上。假如用户需要返回 m 条数据，会向 n 个 shard 各请求 m 条数据，然后在协调节点需要将 n_m 条数据进行排序，选出前 m 条进行 fetch 然后再返回给用户，这样相当于浪费了 (n-1)_m 条数据的计算和 io 资源。因此可以仅从一个 shard 上获取数据，按顺序将所有 shard 上的数据拉取结束，在挨个拉取的过程中，还要保持之前在各个 shard 创建的 searchContext，避免 SearchContext 失效。

查询剪枝后，并发访问方式下，scroll_id 也将变得特别短。之前用户拿到的 scroll_id 特别长，跟用户的 shard 数成正比，当 shard 数较多时候，scroll_id 也特别长，在传输过程和 scroll_id 编码解析过程中都会浪费一些系统资源。

### shard 选择策略

一个索引通常会有很多副本，当请求到达协调节点后，请求应该转发到哪个副本呢？  
  
默认情况下，采用的是随机策略，将所有副本打乱随机拿出一个副本即可。默认的随机策略能够将请求均匀地打散在每一个 shard 上。假如我们的 data 节点处理能力不一致，或者由于一些原因造成某些机器负载较高，那么采用随机策略可能不太适用。Elasticsearch 提供了一个自适应的选择策略，其能够根据当前的每个节点的状态来选择最佳的副本。参考因素有如下源码列出的，包括节点的 client 数、队列长度、响应时间、服务时间等。因此，通过 "cluster.routing.use_adaptive_replica_selection" 参数将副本自适应选择策略打开，能够发挥每一台机器的能力，请求延时能够有效降低，每台机器的负载能够更加均匀。

```
ComputedNodeStats(int clientNum, NodeStatistics nodeStats) {
    this(nodeStats.nodeId, clientNum,(int) nodeStats.queueSize.getAverage(), nodeStats.responseTime.getAverage(), nodeStats.serviceTime);
}
```

  
针对 SearchScroll 请求，如果是频率较高的拉取不同索引的少量数据，那么副本自适应选择策略可以满足需求。但是针对一些大索引拉取数据的 case 则不再适用。假如某一个索引有 512 个 shard，且需要拉取的数据较多，那么集群资源可能仅够该索引大量拉取，不会再有其他请求过来。当 512 个并发请求一下子进来协调节点，这时候协调节点会拉取每个 data 节点的状态来决定把请求发往哪个副本。但是 512 个并发是一起过来的，因此拿到的 nodeStats 可能是一致的，会造请求发往相同的 data 节点，造成一些 data 节点负载较高，而其他 data 节点负载较低。SearchScroll 的首轮请求会决定了后续请求在哪个 data 节点执行，因此后续所有请求和首轮一样，造成各个 data 节点负载不一致。

针对这种情况，如果索引 shard 较多，且用户是 SearchScroll 请求，则需要不再使用副本自适应选择策略。

### 请求支持重试

自 Elasticsearch 支持 SearchScroll 以来，scroll_id 都是不变的，所有的游标位点信息都是维护在 data 节点的 searchContext 中。scroll_id 仅仅编码了 node_id 和 context_id。协调节点根据 node_id 将请求转发到对应的 data 节点，在 data 节点上根据 context_id 拿到 searchContext，最后拿到所有相关的具体信息。

当前 scroll_id 是不支持重试的，强行进行重试可能会造成数据丢失，因此遇到失败则需要全部重新拉取。比如用户有 100 条数据需要拉取，每次拉 10 条。当拉取 20~30 条时候，Elasticsearch 已经拿到数据，代表着 data 节点的游标位点信息已经更新，但是用户网络发生问题，没有取到这 10 条数据。这时候用户忽略网络异常而继续请求的话，会拿到 30~40 的 10 条数据，而 20~30 的 10 条数据再也拿不到，造成读取数据丢失。针对这一问题，我们将 searchContext 中维护的 last_emitted_doc 编码到 scroll_id 中，这样在部分场景失败下就可以进行重试。

之前 scroll_id 的编码是为 query_type + array_size + array[context_id + node_id]，我们优化后的 scroll_id 为增加了 version、index_name、last_emitted_doc 等信息：

*   version 字段是为了以后做版本兼容使用，当前的 scroll_id 并没有版本的概念，因此版本兼容难做。
*   index_name 是索引的名字。虽然该字段对查询没有任何用处，但是在 stats 监控中需要用到。之前我们仅能统计 SearchScroll 的整个集群或者 Node 级别的监控，现在拿到 index_name 后，可以做到索引级别更细粒度的监控，比如拿到某一个索引 Scroll 阶段的 query、merge、sort、fetch 等各项监控信息。
*   last_emitted_doc 是新增的字段，在 Elasticsearch 中是 ScoreDoc.java，主要编码的是 doc 和 score 两个字段。如果 ScoreDoc 是 FieldDoc 子类型，则还会编码 fields。

scroll_id 中编码 last_emitted_doc 后，用户的每次请求我们都能拿到当前的游标位点信息。在协调节点中，通过 InternalScrollSearchRequest 将该 Request 从协调节点发送到 data 节点，最终 data 节点不再从 searchContext 中拿 last_emitted_doc，而是从 InternalScrollSearchRequest 拿到 last_emitted_doc 游标位点。

除此之外，当前 Elasticsearch 的 SearchContext 是不支持并发访问的，且没有给出任何提示，如果并发访问会造成拿到的数据错乱。因此，我们将 SearchContext 加了状态，如果访问一个正在被访问的 SearchContext，则抛出冲突异常。

## 最后

本文介绍了 SearchScroll 的基本概念和一些内部原理，最后介绍了我们在 SearchScroll 方面做的一些性能优化工作，希望大家对 SearchScroll 有更深的理解。  
  
如果您是 Java 老司机，或者对 Lucene、Elasticsearch、Solr 等相关引擎运用熟练、理解到位，或者想从事搜索引擎相关的一些工作，可以联系寻剑 [xunjian.sl@alibaba-inc.com](mailto:xunjian.sl@alibaba-inc.com)，团队技术氛围浓厚、简单淳朴，欢迎大家私聊了解情况。