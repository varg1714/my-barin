---
create: 2023-11-13 01:20
---
## 前言

在上一篇文章我们介绍了 Lucene 的基本概念，在本篇文章我们将深入 Lucene 中最核心的类之一 IndexWriter，来探索 Lucene 中数据写入和索引构建的整个过程。

## IndexWriter

```
// initialization
Directory index = new NIOFSDirectory(Paths.get("/index"));
IndexWriterConfig config = new IndexWriterConfig();
IndexWriter writer = new IndexWriter(index, config);
// create a document
Document doc = new Document();
doc.add(new TextField("title", "Lucene - IndexWriter", Field.Store.YES));
doc.add(new StringField("content", "招人，求私信", Field.Store.YES));
// index the document
writer.addDocument(doc);
writer.commit();
```

先看下 Lucene 中如何使用 IndexWriter 来写入数据，上面是一段精简的调用示例代码，整个过程主要有三个步骤：

1.  初始化：初始化 IndexWriter 必要的两个元素是 Directory 和 IndexWriterConfig，Directory 是 Lucene 中数据持久层的抽象接口，通过这层接口可以实现很多不同类型的数据持久层，例如本地文件系统、网络文件系统、数据库或者是分布式文件系统。IndexWriterConfig 内提供了很多可配置的高级参数，提供给高级玩家进行性能调优和功能定制，它提供的几个关键参数后面会细说。
2.  构造文档：Lucene 中文档由 Document 表示，Document 由 Field 构成。Lucene 提供多种不同类型的 Field，其 FiledType 决定了它所支持的索引模式，当然也支持自定义 Field，具体方式可参考上一篇文章。
3.  写入文档：通过 IndexWriter 的 addDocument 函数写入文档，写入时同时根据 FieldType 创建不同的索引。文档写入完成后，还不可被搜索，最后需要调用 IndexWriter 的 commit，在 commit 完后 Lucene 才保证文档被持久化并且是 searchable 的。

以上就是 Lucene 的一个简明的数据写入流程，核心是 IndexWriter，整个过程被抽象的非常简洁明了。一个设计优良的库的最大特点，就是可以让普通玩家以非常小的代价学习和使用，同时又照顾高级玩家能够提供可调节的性能参数和功能定制能力。

## IndexWriterConfig

IndexWriterConfig 内提供了一些供高级玩家做性能调优和功能定制的核心参数，我们列几个主要的看下：

*   **IndexDeletionPolicy**：Lucene 开放对 commit point 的管理，通过对 commit point 的管理可以实现例如 snapshot 等功能。Lucene 默认配置的 DeletionPolicy，只会保留最新的一个 commit point。
*   **Similarity**：搜索的核心是相关性，Similarity 是相关性算法的抽象接口，Lucene 默认实现了 TF-IDF 和 BM25 算法。相关性计算在数据写入和搜索时都会发生，数据写入时的相关性计算称为 Index-time boosting，计算 Normalizaiton 并写入索引，搜索时的相关性计算称为 query-time boosting。
*   **MergePolicy**：Lucene 内部数据写入会产生很多 Segment，查询时会对多个 Segment 查询并合并结果。所以 Segment 的数量一定程度上会影响查询的效率，所以需要对 Segment 进行合并，合并的过程就称为 Merge，而何时触发 Merge 由 MergePolicy 决定。
*   **MergeScheduler**：当 MergePolicy 触发 Merge 后，执行 Merge 会由 MergeScheduler 来管理。Merge 通常是比较耗 CPU 和 IO 的过程，MergeScheduler 提供了对 Merge 过程定制管理的能力。
*   **Codec**：Codec 可以说是 Lucene 中最核心的部分，定义了 Lucene 内部所有类型索引的 Encoder 和 Decoder。Lucene 在 Config 这一层将 Codec 配置化，主要目的是提供对不同版本数据的处理能力。对于 Lucene 用户来说，这一层的定制需求通常较少，能玩 Codec 的通常都是顶级玩家了。
*   **IndexerThreadPool**：管理 IndexWriter 内部索引线程（DocumentsWriterPerThread）池，这也是 Lucene 内部定制资源管理的一部分。
*   **FlushPolicy**：FlushPolicy 决定了 In-memory buffer 何时被 flush，默认的实现会根据 RAM 大小和文档个数来判断 Flush 的时机，FlushPolicy 会在每次文档 add/update/delete 时调用判定。
*   **MaxBufferedDoc**：Lucene 提供的默认 FlushPolicy 的实现 FlushByRamOrCountsPolicy 中允许 DocumentsWriterPerThread 使用的最大文档数上限，超过则触发 Flush。
*   **RAMBufferSizeMB**：Lucene 提供的默认 FlushPolicy 的实现 FlushByRamOrCountsPolicy 中允许 DocumentsWriterPerThread 使用的最大内存上限，超过则触发 flush。
*   **RAMPerThreadHardLimitMB**：除了 FlushPolicy 能决定 Flush 外，Lucene 还会有一个指标强制限制 DocumentsWriterPerThread 占用的内存大小，当超过阈值则强制 flush。
*   **Analyzer**：即分词器，这个通常是定制化最多的，特别是针对不同的语言。

## 核心操作

IndexWriter 提供很简单的几种操作接口，这一章节会做一个简单的功能和用途解释，下一个章节会对其内部实现做一个详细的剖析。IndexWrite 的提供的核心 API 如下：

*   addDocument：比较纯粹的一个 API，就是向 Lucene 内新增一个文档。Lucene 内部没有主键索引，所有新增文档都会被认为一个新的文档，分配一个独立的 docId。
*   updateDocuments：更新文档，但是和数据库的更新不太一样。数据库的更新是查询后更新，Lucene 的更新是查询后删除再新增。流程是先 delete by term，后 add document。但是这个流程又和直接先调用 delete 后调用 add 效果不一样，只有 update 能够保证在 Thread 内部删除和新增保证原子性，详细流程在下一章节会细说。
*   deleteDocument：删除文档，支持两种类型删除，by term 和 by query。在 IndexWriter 内部这两种删除的流程不太一样，在下一章节再细说。
*   flush：触发强制 flush，将所有 Thread 的 In-memory buffer flush 成 segment 文件，这个动作可以清理内存，强制对数据做持久化。
*   prepareCommit/commit/rollback：commit 后数据才可被搜索，commit 是一个二阶段操作，prepareCommit 是二阶段操作的第一个阶段，也可以通过调用 commit 一步完成，rollback 提供了回滚到 last commit 的操作。
*   maybeMerge/forceMerge：maybeMerge 触发一次 MergePolicy 的判定，而 forceMerge 则触发一次强制 merge。

## 数据路径

上面几个章节介绍了 IndexWriter 的基本流程、配置和核心接口，非常简单和易理解。这一章节，我们将深入 IndexWriter 内部，来探索其内核实现。

![](https://pic4.zhimg.com/v2-7a9ee709ca224ac0b32ab3a901cd6577_r.jpg)

如上是 IndexWriter 内部核心流程架构图，接下来我们将以 add/update/delete/commit 这些主要操作来讲解 IndexWriter 内部的数据路径。

## 并发模型

IndexWriter 提供的核心接口都是线程安全的，并且内部做了特殊的并发优化来优化多线程写入的性能。IndexWriter 内部为每个线程都会单独开辟一个空间来写入，这块空间由 DocumentsWriterPerThread 来控制。整个多线程数据处理流程为：

1.  多线程并发调用 IndexWriter 的写接口，在 IndexWriter 内部具体请求会由 DocumentsWriter 来执行。DocumentsWriter 内部在处理请求之前，会先根据当前执行操作的 Thread 来分配 DocumentsWriterPerThread。
2.  每个线程在其独立的 DocumentsWriterPerThread 空间内部进行数据处理，包括分词、相关性计算、索引构建等。
3.  数据处理完毕后，在 DocumentsWriter 层面执行一些后续动作，例如触发 FlushPolicy 的判定等。

引入了 DocumentsWriterPerThread（后续简称为 DWPT）后，Lucene 内部在处理数据时，整个处理步骤只需要对以上第一步和第三步进行加锁，第二步完全不用加锁，每个线程都在自己独立的空间内处理数据。而通常来说，第一步和第三步都是非常轻量级的，而第二步是对计算和内存资源消耗最大的。所以这样做之后，能够将加锁的时间大大缩短，提高并发的效率。每个 DWPT 内单独包含一个 In-memory buffer，这个 buffer 最终会 flush 成不同的独立的 segment 文件。

这种方案下，对多线程并发写入性能有很大的提升。特别是针对纯新增文档的场景，所有数据写入都不会有冲突，所以非常适合这种空间隔离式的数据写入方式。但对于删除文档的场景，一次删除动作可能会涉及删除不同线程空间内的数据，这里 Lucene 也采取了一种特殊的交互方式来降低锁的开销，在剖析 delete 操作时会细说。

在搜索场景中，全量构建索引的阶段，基本是纯新增文档式的写入，而在后续增量索引阶段（特别是数据源是数据库时），会涉及大量的 update 和 delete 操作。从原理上来分析，一个最佳实践是包含相同唯一主键 Term 的文档分配相同的线程来处理，使数据更新发生在一个独立线程空间内，避免跨线程。

## add & update

add 接口用于新增文档，update 接口用于更新文档。但 Lucene 的 update 和数据库的 update 不太一样。数据库的更新是查询后更新，Lucene 的更新是查询后删除再新增，不支持更新文档内部分列。流程是先 delete by term，后 add document。

IndexWriter 提供的 add 和 update 接口，都会映射到 DocumentsWriter 的 udpate 接口，看下接口定义：

```
long updateDocument(final Iterable<? extends IndexableField> doc, final Analyzer analyzer,
    final Term delTerm) throws IOException, AbortingException
```

这个函数内的处理流程是：

1.  根据 Thread 分配 DWPT
2.  在 DWPT 内执行 delete
3.  在 DWPT 内执行 add

关于 delete 操作的细节在下一小结详细说，add 操作会直接将文档写入 DWPT 内的 In-memory buffer。

## delete

delete 相对 add 和 update 来说，是完全不同的一个数据路径。而且 update 和 delete 虽然内部都会执行数据删除，但这两者又是不同的数据路径。文档删除不会直接影响 In-memory buffer 内的数据，而是会有另外的方式来达到删除的目的。

![](https://pic1.zhimg.com/v2-5e6185d2780e39a1149d415afe10a830_r.jpg)

在 Delete 路径上关键的数据结构就是 Deletion queue，在 IndexWriter 内部会有一个全局的 Deletion Queue，称为 Global Deletion Queue，而在每个 DWPT 内部，还会有一个独立的 Deletion Queue，称为 Pending Updates。DWPT Pending Updates 会与 Global Deletion Queue 进行双向同步，因为文档删除是全局范围的，不应该只发生在 DWPT 范围内。

Pending Updates 内部会按发生顺序记录每个删除动作，并且标记该删除影响的文档范围，文档影响范围通过记录当前已写入的最大 DocId（DocId Upto）来标记，即代表这个删除动作只删除小于等于该 DocId 的文档。

update 接口和 delete 接口都可以进行文档删除，但是有一些差异：

*   update 只能进行 by term 的文档删除，而 delete 除了 by term，还支持 by query。
*   update 的删除会先作用于 DWPT 内部，后作用于 Global，再由 Global 同步到其他 DWPT。
*   delete 的删除会作用在 Global 级别，后异步同步到 DWPT 级别。

update 和 delete 流程上的差异也决定了他们行为上的一些差异，update 的删除操作会先发生在 DWPT 内部，并且是和 add 同时发生，所以能够保证该 DWPT 内部的 delete 和 add 的原子性，即保证在 add 之前的所有符合条件的文档一定被删除。

DWPT Pending Updates 里的删除操作什么时候会真正作用于数据呢？在 Lucene Segment 内部，数据实际上并不会被真正删除。Segment 中有一个特殊的文件叫 live docs，内部是一个位图的数据结构，记录了这个 Segment 内部哪些 DocId 是存活的，哪些 DocId 是被删除的。所以删除的过程就是构建 live docs 标记位图的过程，数据实际上不会被真正删除，只是在 live docs 里会被标记删除。Term 删除和 Query 删除会在不同阶段构建 live docs，Term 删除要求先根据 Term 查询出它关联的所有 doc，所以很明显这个会发生在倒排索引构建时。而 Query 删除要求执行一次完整的查询后才能拿到其对应的 docId，所以会发生在 segment 被 flush 完成后，基于 flush 后的索引文件构建 IndexReader 后执行搜索才能完成。

还有一点要注意的是，live docs 只影响倒排，所以在 live docs 里被标记删除的文档没有办法通过倒排索引检索出，但是还能够通过 doc id 查询到 store fields。当然文档数据最终是会被真正物理删除，这个过程会发生在 merge 时。

## flush

flush 是将 DWPT 内 In-memory buffer 里的数据持久化到文件的过程，flush 会在每次新增文档后由 FlushPolicy 判定自动触发，也可以通过 IndexWriter 的 flush 接口手动触发。

每个 DWPT 会 flush 成一个 segment 文件，flush 完成后这个 segment 文件是不可被搜索的，只有在 commit 之后，所有 commit 之前 flush 的文件才可被搜索。

## commit

commit 时会触发数据的一次强制 flush，commit 完成后再此之前 flush 的数据才可被搜索。commit 动作会触发生成一个 commit point，commit point 是一个文件。Commit point 会由 IndexDeletionPolicy 管理，lucene 默认配置的策略只会保留 last commit point，当然 lucene 提供其他多种不同的策略供选择。

## merge

merge 是对 segment 文件合并的动作，合并的好处是能够提高查询的效率以及回收一些被删除的文档。Merge 会在 segment 文件 flush 时触发 MergePolicy 来判定自动触发，也可通过 IndexWriter 进行一次 force merge。

## IndexingChain

前面几个章节主要介绍了 IndexWriter 内部各个关键操作的流程，本小节会介绍最核心的 DWPT 内部对文档进行索引构建的流程。Lucene 内部索引构建最关键的概念是 IndexingChain，顾名思义，链式的索引构建。为啥是链式的？这个和 Lucene 的整个索引体系结构有关系，Lucene 提供了各种不同类型的索引类型，例如倒排、正排（列存）、StoreField、DocValues 等。每个不同的索引类型对应不同的索引算法、数据结构以及文件存储，有些是列级别的，有些是文档级别的。所以一个文档写入后，需要被这么多种不同索引处理，有些索引会共享 memory-buffer，有些则是完全独立的。基于这个架构，理论上 Lucene 是提供了扩展其他类型索引的可能性，顶级玩家也可以去尝试。

![](https://pic2.zhimg.com/v2-df5fdb63221a8c686e75f9c6d4b4f749_r.jpg)

在 IndexWriter 内部，indexing chain 上索引构建顺序是 invert index、store fields、doc values 和 point values。有些索引类型处理文档后会将索引内容直接写入文件（主要是 store field 和 term vector），而有些索引类型会先将文档内容写入 memory buffer，最后在 flush 的时候再写入文件。能直接写入文件的索引，通常是文档级的索引，索引构建可以文档级的增量构建。而不能写入文件的索引，例如倒排，则必须等 Segment 内所有文档全部写入完毕后，会先对 Term 进行一个全排序，之后才能构建索引，所以必须要有一个 memory-buffer 先缓存所有文档。

前面提到，IndexWriterConfig 支持配置 Codec，Codec 就是对应每种类型索引的 Encoder 和 Decoder。在上图可以看到，在 Lucene 7.2.1 版本中，主要有这么几种 Codec：

*   BlockTreeTermsWriter：倒排索引对应的 Codec，其中倒排表部分使用 Lucene50PostingsWriter(Block 方式写入倒排链) 和 Lucene50SkipWriter(对 Block 的 SkipList 索引)，词典部分则是使用 FST（针对倒排表 Block 级的词典索引）。
*   CompressingTermVectorsWriter：对应 Term vector 索引的 Writer，底层是压缩 Block 格式。
*   CompressingStoredFieldsWriter：对应 Store fields 索引的 Writer，底层是压缩 Block 格式。
*   Lucene70DocValuesConsumer：对应 Doc values 索引的 Writer。
*   Lucene60PointsWriter：对应 Point values 索引的 Writer。

这一章节主要了解 IndexingChain 内部的文档索引处理流程，核心是链式分阶段的索引，并且不同类型索引支持 Codec 可配置。

## 总结

这篇文章主要从一个全局视角来讲解 IndexWriter 的配置、接口、并发模型、核心操作的数据路径以及索引链，在之后的文章中，会再深入索引链中每种不同类型索引的构建流程，探索其 memory-buffer 的实现、索引算法以及数据存储格式。