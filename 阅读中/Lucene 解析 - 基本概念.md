---
create: 2023-11-13 01:20
---
## 0.1. 前言

Apache Lucene 是一个开源的高性能、可扩展的信息检索引擎，提供了强大的数据检索能力。Lucene 已经发展了很多年，其功能越来越强大，架构也越来越精细。它目前不仅仅能支持全文索引，也能够提供多种其他类型的索引方式，来满足不同类型的查询需求。

基于 Lucene 的开源项目有很多，最知名的要属 Elasticsearch 和 Solr，如果说 Elasticsearch 和 Solr 是一辆设计精美、性能卓越的跑车，那 Lucene 就是为其提供强大动力的引擎。为了驾驭这辆跑车让它跑的更快更稳定，我们需要对它的引擎研究透彻。

在此之前我们在专栏已经发表了多篇文章来剖析 Elasticsearch 的数据模型、读写路径、分布式架构以及 Data/Meta 一致性等问题，这篇文章之后我们会陆续发表一系列的关于 Lucene 的原理和源码解读，来全面解析 Lucene 的数据模型和数据读写路径。

Lucene 官方对自己的优势总结为几点：

1.  Scalable, High-Performance Indexing
2.  Powerful, Accurate and Efficient Search Algorithms

希望通过我们的系列文章，能够让读者理解 Lucene 是如何达到这些目标的。

整个分析会基于 Lucene 7.2.1 版本，在读这篇文章之前，需要有一定的知识基础，例如了解基本的搜索和索引原理，知道什么是倒排、分词、相关性等基本概念，了解 Lucene 的基本使用，例如 Directory、IndexWriter、IndexSearcher 等。

## 0.2. 基本概念

在深入解读 Lucene 之前，先了解下 Lucene 的几个基本概念，以及这几个概念背后隐藏的一些东西。

![](https://pic2.zhimg.com/v2-89d424acd24ab21954774b9391353ca5_r.jpg)

**Index（索引）**

类似数据库的表的概念，但是与传统表的概念会有很大的不同。传统关系型数据库或者 NoSQL 数据库的表，在创建时至少要定义表的 Scheme，定义表的主键或列等，会有一些明确定义的约束。而 Lucene 的 Index，则完全没有约束。Lucene 的 Index 可以理解为一个文档收纳箱，你可以往内部塞入新的文档，或者从里面拿出文档，但如果你要修改里面的某个文档，则必须先拿出来修改后再塞回去。这个收纳箱可以塞入各种类型的文档，文档里的内容可以任意定义，Lucene 都能对其进行索引。

**Document（文档）**

类似数据库内的行或者文档数据库内的文档的概念，一个 Index 内会包含多个 Document。写入 Index 的 Document 会被分配一个唯一的 ID，即 Sequence Number（更多被叫做 DocId），关于 Sequence Number 后面会再细说。

**Field（字段）**

一个 Document 会由一个或多个 Field 组成，Field 是 Lucene 中数据索引的最小定义单位。Lucene 提供多种不同类型的 Field，例如 StringField、TextField、LongFiled 或 NumericDocValuesField 等，Lucene 根据 Field 的类型（FieldType）来判断该数据要采用哪种类型的索引方式（Invert Index、Store Field、DocValues 或 N-dimensional 等），关于 Field 和 FieldType 后面会再细说。

**Term 和 Term Dictionary**

Lucene 中索引和搜索的最小单位，一个 Field 会由一个或多个 Term 组成，Term 是由 Field 经过 Analyzer（分词）产生。Term Dictionary 即 Term 词典，是根据条件查找 Term 的基本索引。

**Segment**

一个 Index 会由一个或多个 sub-index 构成，sub-index 被称为 Segment。Lucene 的 Segment 设计思想，与 LSM 类似但又有些不同，继承了 LSM 中数据写入的优点，但是在查询上只能提供近实时而非实时查询。

Lucene 中的数据写入会先写内存的一个 Buffer（类似 LSM 的 MemTable，但是不可读），当 Buffer 内数据到一定量后会被 flush 成一个 Segment，每个 Segment 有自己独立的索引，可独立被查询，但数据永远不能被更改。这种模式避免了随机写，数据写入都是 Batch 和 Append，能达到很高的吞吐量。Segment 中写入的文档不可被修改，但可被删除，删除的方式也不是在文件内部原地更改，而是会由另外一个文件保存需要被删除的文档的 DocID，保证数据文件不可被修改。Index 的查询需要对多个 Segment 进行查询并对结果进行合并，还需要处理被删除的文档，为了对查询进行优化，Lucene 会有策略对多个 Segment 进行合并，这点与 LSM 对 SSTable 的 Merge 类似。

Segment 在被 flush 或 commit 之前，数据保存在内存中，是不可被搜索的，这也就是为什么 Lucene 被称为提供近实时而非实时查询的原因。读了它的代码后，发现它并不是不能实现数据写入即可查，只是实现起来比较复杂。原因是 Lucene 中数据搜索依赖构建的索引（例如倒排依赖 Term Dictionary），Lucene 中对数据索引的构建会在 Segment flush 时，而非实时构建，目的是为了构建最高效索引。当然它可引入另外一套索引机制，在数据实时写入时即构建，但这套索引实现会与当前 Segment 内索引不同，需要引入额外的写入时索引以及另外一套查询机制，有一定复杂度。

**Sequence Number**

Sequence Number（后面统一叫 DocId）是 Lucene 中一个很重要的概念，数据库内通过主键来唯一标识一行，而 Lucene 的 Index 通过 DocId 来唯一标识一个 Doc。不过有几点要特别注意：

1.  DocId 实际上并不在 Index 内唯一，而是 Segment 内唯一，Lucene 这么做主要是为了做写入和压缩优化。那既然在 Segment 内才唯一，又是怎么做到在 Index 级别来唯一标识一个 Doc 呢？方案很简单，Segment 之间是有顺序的，举个简单的例子，一个 Index 内有两个 Segment，每个 Segment 内分别有 100 个 Doc，在 Segment 内 DocId 都是 0-100，转换到 Index 级的 DocId，需要将第二个 Segment 的 DocId 范围转换为 100-200。
2.  DocId 在 Segment 内唯一，取值从 0 开始递增。但不代表 DocId 取值一定是连续的，如果有 Doc 被删除，那可能会存在空洞。
3.  一个文档对应的 DocId 可能会发生变化，主要是发生在 Segment 合并时。

Lucene 内最核心的倒排索引，本质上就是 Term 到所有包含该 Term 的文档的 DocId 列表的映射。所以 Lucene 内部在搜索的时候会是一个两阶段的查询，第一阶段是通过给定的 Term 的条件找到所有 Doc 的 DocId 列表，第二阶段是根据 DocId 查找 Doc。Lucene 提供基于 Term 的搜索功能，也提供基于 DocId 的查询功能。

DocId 采用一个从 0 开始底层的 Int32 值，是一个比较大的优化，同时体现在数据压缩和查询效率上。例如数据压缩上的 Delta 策略、ZigZag 编码，以及倒排列表上采用的 SkipList 等，这些优化后续会详述。

## 0.3. 索引类型

Lucene 中支持丰富的字段类型，每种字段类型确定了支持的数据类型以及索引方式，目前支持的字段类型包括 LongPoint、TextField、StringField、NumericDocValuesField 等。

![](https://pic1.zhimg.com/v2-47e85d4edb8a1b786d3f1371cf24ee54_r.jpg)

如图是 Lucene 中对于不同类型 Field 定义的一个基本关系，所有字段类都会继承自 [Field](https://link.zhihu.com/?target=http%3A//lucene.apache.org/core/7_2_1/core/org/apache/lucene/document/Field.html) 这个类，Field 包含 3 个重要属性：name(String)、fieldsData(BytesRef) 和 type(FieldType)。name 即字段的名称，fieldsData 即字段值，所有类型的字段的值最终都会转换为二进制字节流来表示。type 是字段类型，确定了该字段被索引的方式。

FieldType 是一个很重要的类，包含多个重要属性，这些属性的值决定了该字段被索引的方式。

Lucene 提供的多种不同类型的 Field，本质区别就两个：一是不同类型值到 fieldData 定义了不同的转换方式；二是定义了 FieldType 内不同属性不同取值的组合。这种模式下，你也能够通过自定义数据以及组合 FieldType 内索引参数来达到定制类型的目的。

要理解 Lucene 能够提供哪些索引方式，只需要理解 FieldType 内每个属性的具体含义，我们来一个一个看：

*   **stored**: 代表是否需要保存该字段，如果为 false，则 lucene 不会保存这个字段的值，而搜索结果中返回的文档只会包含保存了的字段。
*   **tokenized**: 代表是否做分词，在 lucene 中只有 TextField 这一个字段需要做分词。
*   **termVector**: 这篇[文章](https://link.zhihu.com/?target=http%3A//makble.com/what-is-term-vector-in-lucene)很好的解释了 term vector 的概念，简单来说，term vector 保存了一个文档内所有的 term 的相关信息，包括 Term 值、出现次数（frequencies）以及位置（positions）等，是一个 per-document inverted index，提供了根据 docid 来查找该文档内所有 term 信息的能力。对于长度较小的字段不建议开启 term verctor，因为只需要重新做一遍分词即可拿到 term 信息，而针对长度较长或者分词代价较大的字段，则建议开启 term vector。Term vector 的用途主要有两个，一是关键词高亮，二是做文档间的相似度匹配（more-like-this）。
*   **omitNorms**: Norms 是 normalization 的缩写，lucene 允许每个文档的每个字段都存储一个 normalization factor，是和搜索时的相关性计算有关的一个系数。Norms 的存储只占一个字节，但是每个文档的每个字段都会独立存储一份，且 Norms 数据会全部加载到内存。所以若开启了 Norms，会消耗额外的存储空间和内存。但若关闭了 Norms，则无法做 index-time boosting（elasticsearch 官方[建议](https://link.zhihu.com/?target=https%3A//www.elastic.co/guide/en/elasticsearch/reference/5.2/mapping-boost.html)使用 query-time boosting 来替代）以及 [length normalization](https://link.zhihu.com/?target=https%3A//www.coursera.org/learn/text-retrieval/lecture/RnXhr/lesson-2-3-doc-length-normalization)。
*   **indexOptions**: Lucene 提供倒排索引的 5 种可选参数（NONE、DOCS、DOCS_AND_FREQS、DOCS_AND_FREQS_AND_POSITIONS、DOCS_AND_FREQS_AND_POSITIONS_AND_OFFSETS），用于选择该字段是否需要被索引，以及索引哪些内容。
*   **docValuesType**: DocValue 是 Lucene 4.0 引入的一个正向索引（docid 到 field 的一个列存），大大优化了 sorting、faceting 或 aggregation 的效率。DocValues 是一个强 schema 的存储结构，开启 DocValues 的字段必须拥有严格一致的类型，目前 Lucene 只提供 NUMERIC、BINARY、SORTED、SORTED_NUMERIC 和 SORTED_SET 五种类型。
*   **dimension**：Lucene 支持多维数据的索引，采取特殊的索引来优化对多维数据的查询，这类数据最典型的应用场景是地理位置索引，一般经纬度数据会采取这个索引方式。

来看下 Lucene 中对 StringField 的一个定义：

![](https://pic4.zhimg.com/v2-aa5a8f88ade925450922e3d24a500493_r.jpg)

StringFiled 有两种类型索引定义，TYPE_NOT_STORED 和 TYPE_STORED，唯一的区别是这个 Field 是否需要 Store。从其他的几个属性也可以解读出，StringFiled 选择 omitNorms，需要进行倒排索引并且不需要被分词。

## 0.4. Elasticsearch 数据类型

Elasticsearch 内对用户输入文档内 Field 的索引，也是按照 Lucene 能提供的几种模式来提供。除了用户能自定义的 Field，Elasticsearch 还有自己预留的系统字段，用作一些特殊的目的。这些字段映射到 Lucene 本质上也是一个 Field，与用户自定义的 Field 无任何区别，只不过 Elasticsearch 根据这些系统字段不同的使用目的，定制有不同的索引方式。

![](https://pic3.zhimg.com/v2-b1e649d9421f6e3aa08fc099c613f682_r.jpg)

举个例子，上图是 Elasticsearch 内两个系统字段_version 和_uid 的 FieldType 定义，我们来解读下它们的索引方式。Elasticsearch 通过_uid 字段唯一标识一个文档，通过_version 字段来记录该文档当前的版本。从这两个字段的 FieldType 定义上可以看到，_uid 字段会做倒排索引，不需要分词，需要被 Store。而_version 字段则不需要被倒排索引，也不需要被 Store，但是需要被正排索引。很好理解，因为_uid 需要被搜索，而_version 不需要。但_version 需要通过 docId 来查询，而且 Elasticsearch 内 versionMap 内需要通过 docId 做大量查询且只需要查询出_version 字段，所以_version 最合适的是被正排索引。

关于 Elasticsearch 内系统字段全面的解析，可以看下这篇[文章](https://zhuanlan.zhihu.com/p/34680841)。

## 0.5. 总结

1. 这篇文章主要介绍了 Lucene 的一些基本概念以及提供的索引类型。后续我们会有一系列文章来解析 Lucene 提供的 IndexWriter 的写入流程，其 In-Memory Buffer 的结构以及持久化后的索引文件结构，来了解 Lucene 为何能达到如此高效的数据索引性能。也会去解析 IndexSearcher 的查询流程，以及一些特殊的查询优化的数据结构，来了解为何 Lucene 能提供如此高效的搜索和查询。
2. 这是内容很长的一段话
    看看内容呢？
    1. 呵呵
        - 呵呵，这是很长的第一段话哦嘿嘿嘿送来樊笼看法拉萨开梦阿赛蓝光阿塞看光爱三表阿赛激光阿塞看光啊空单光阿塞感阿龙蛋光卡赛激光阿三开干阿龙蛋光阿空等级阿龙等
            这里这个里面的内容哦
        - 哈哈
    1. 哈哈 cene 提供的 IndexWriter 的写入流程，其 In-Memory Buffer 的结构以及持久化后的索引文件结构，来了解 Lucene 为何能达到如此高效的数据索引性能。也会去解析 IndexSearcher 的查询流程
        这个里面也有内容