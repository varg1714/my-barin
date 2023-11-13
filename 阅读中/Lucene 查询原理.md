---
create: 2023-11-13 01:20
---
## 前言

Lucene 是一个基于 Java 的全文信息检索工具包，目前主流的搜索系统 Elasticsearch 和 solr 都是基于 lucene 的索引和搜索能力进行。想要理解搜索系统的实现原理，就需要深入 lucene 这一层，看看 lucene 是如何存储需要检索的数据，以及如何完成高效的数据检索。

在数据库中因为有索引的存在，也可以支持很多高效的查询操作。不过对比 lucene，数据库的查询能力还是会弱很多，本文就将探索下 lucene 支持哪些查询，并会重点选取几类查询分析 lucene 内部是如何实现的。为了方便大家理解，我们会先简单介绍下 lucene 里面的一些基本概念，然后展开 lucene 中的几种数据存储结构，理解了他们的存储原理后就可以方便知道如何基于这些存储结构来实现高效的搜索。本文重点关注是 lucene 如何做到传统数据库较难做到的查询，对于分词，打分等功能不会展开介绍。

本文具体会分以下几部分：

1.  介绍 lucene 的数据模型，细节可以参阅 lucene 数据模型一文。
2.  介绍 lucene 中如何存储需要搜索的 term。
3.  介绍 lucene 的倒排链的如何存储以及如何实现 docid 的快速查找。
4.  介绍 lucene 如何实现倒排链合并。
5.  介绍 lucene 如何做范围查询和前缀匹配。
6.  介绍 lucene 如何优化数值类范围查询。

## Lucene 数据模型

Lucene 中包含了四种基本数据类型，分别是：

Index：索引，由很多的 Document 组成。  
Document：由很多的 Field 组成，是 Index 和 Search 的最小单位。  
Field：由很多的 Term 组成，包括 Field Name 和 Field Value。  
Term：由很多的字节组成。一般将 Text 类型的 Field Value 分词之后的每个最小单元叫做 Term。

在 lucene 中，读写路径是分离的。写入的时候创建一个 IndexWriter，而读的时候会创建一个 IndexSearcher，  
下面是一个简单的代码示例，如何使用 lucene 的 IndexWriter 建索引以及如何使用 indexSearch 进行搜索查询。

```
Analyzer analyzer = new StandardAnalyzer();
    // Store the index in memory:
    Directory directory = new RAMDirectory();
    // To store an index on disk, use this instead:
    //Directory directory = FSDirectory.open("/tmp/testindex");
    IndexWriterConfig config = new IndexWriterConfig(analyzer);
    IndexWriter iwriter = new IndexWriter(directory, config);
    Document doc = new Document();
    String text = "This is the text to be indexed.";
    doc.add(new Field("fieldname", text, TextField.TYPE_STORED));
    iwriter.addDocument(doc);
    iwriter.close();

    // Now search the index:
    DirectoryReader ireader = DirectoryReader.open(directory);
    IndexSearcher isearcher = new IndexSearcher(ireader);
    // Parse a simple query that searches for "text":
    QueryParser parser = new QueryParser("fieldname", analyzer);
    Query query = parser.parse("text");
    ScoreDoc[] hits = isearcher.search(query, 1000).scoreDocs;
    //assertEquals(1, hits.length);
    // Iterate through the results:
    for (int i = 0; i < hits.length; i++) {
         Document hitDoc = isearcher.doc(hits[i].doc);
         System.out.println(hitDoc.get("fieldname"));
    }
    ireader.close();
    directory.close();
```

从这个示例中可以看出，lucene 的读写有各自的操作类。本文重点关注读逻辑，在使用 IndexSearcher 类的时候，需要一个 DirectoryReader 和 QueryParser，其中 DirectoryReader 需要对应写入时候的 Directory 实现。QueryParser 主要用来解析你的查询语句，例如你想查 “A and B"，lucene 内部会有机制解析出是 term A 和 term B 的交集查询。在具体执行 Search 的时候指定一个最大返回的文档数目，因为可能会有过多命中，我们可以限制单词返回的最大文档数，以及做分页返回。

下面会详细介绍一个索引查询会经过几步，每一步 lucene 分别做了哪些优化实现。

## Lucene 查询过程

在 lucene 中查询是基于 segment。每个 segment 可以看做是一个独立的 subindex，在建立索引的过程中，lucene 会不断的 flush 内存中的数据持久化形成新的 segment。多个 segment 也会不断的被 merge 成一个大的 segment，在老的 segment 还有查询在读取的时候，不会被删除，没有被读取且被 merge 的 segement 会被删除。这个过程类似于 LSM 数据库的 merge 过程。下面我们主要看在一个 segment 内部如何实现高效的查询。

为了方便大家理解，我们以人名字，年龄，学号为例，如何实现查某个名字（有重名）的列表。

![](https://pic4.zhimg.com/v2-a543029b09b870d87633a86f7daaba6b_r.jpg)

在 lucene 中为了查询 name=XXX 的这样一个条件，会建立基于 name 的倒排链。以上面的数据为例，倒排链如下：  
姓名

![](https://pic4.zhimg.com/80/v2-5f11f09b2d5432a4ef4395583be1920f_1440w.webp)

  
如果我们还希望按照年龄查询，例如想查年龄 = 18 的列表，我们还可以建立另一个倒排链：

![](https://pic4.zhimg.com/80/v2-77088bb5916540ed699fe3d83d6bd53b_1440w.webp)

在这里，Alice，Alan，18，这些都是 term。所以倒排本质上就是基于 term 的反向列表，方便进行属性查找。到这里我们有个很自然的问题，如果 term 非常多，如何快速拿到这个倒排链呢？在 lucene 里面就引入了 term dictonary 的概念，也就是 term 的字典。term 字典里我们可以按照 term 进行排序，那么用一个二分查找就可以定为这个 term 所在的地址。这样的复杂度是 logN，在 term 很多，内存放不下的时候，效率还是需要进一步提升。可以用一个 hashmap，当有一个 term 进入，hash 继续查找倒排链。这里 hashmap 的方式可以看做是 term dictionary 的一个 index。 从 lucene4 开始，为了方便实现 rangequery 或者前缀，后缀等复杂的查询语句，lucene 使用 FST 数据结构来存储 term 字典，下面就详细介绍下 FST 的存储结构。

## FST

我们就用 Alice 和 Alan 这两个单词为例，来看下 FST 的构造过程。首先对所有的单词做一下排序为 “Alice”，“Alan”。

1.  插入 “Alan”  
    

![](https://pic3.zhimg.com/v2-3d65c9a5d838696aec95519bd5d86e3e_r.jpg)

1.  插入 “Alice”  
    

![](https://pic1.zhimg.com/v2-34b4346b6bc63b6fc7654c6e67f0e180_r.jpg)

这样你就得到了一个有向无环图，有这样一个数据结构，就可以很快查找某个人名是否存在。FST 在单 term 查询上可能相比 hashmap 并没有明显优势，甚至会慢一些。但是在范围，前缀搜索以及压缩率上都有明显的优势。

在通过 FST 定位到倒排链后，有一件事情需要做，就是倒排链的合并。因为查询条件可能不止一个，例如上面我们想找 name="alan" and age="18" 的列表。lucene 是如何实现倒排链的合并呢。这里就需要看一下倒排链存储的数据结构

## SkipList

为了能够快速查找 docid，lucene 采用了 SkipList 这一数据结构。SkipList 有以下几个特征：

1.  元素排序的，对应到我们的倒排链，lucene 是按照 docid 进行排序，从小到大。
2.  跳跃有一个固定的间隔，这个是需要建立 SkipList 的时候指定好，例如下图以间隔是 3
3.  SkipList 的层次，这个是指整个 SkipList 有几层  
    

![](https://pic2.zhimg.com/v2-2aad54b99982a1242362cde872ad60d5_r.jpg)

有了这个 SkipList 以后比如我们要查找 docid=12，原来可能需要一个个扫原始链表，1，2，3，5，7，8，10，12。有了 SkipList 以后先访问第一层看到是然后大于 12，进入第 0 层走到 3，8，发现 15 大于 12，然后进入原链表的 8 继续向下经过 10 和 12。  
有了 FST 和 SkipList 的介绍以后，我们大体上可以画一个下面的图来说明 lucene 是如何实现整个倒排结构的：  

![](https://pic3.zhimg.com/v2-926c47383573f70da40475d2ae2777ce_r.jpg)

有了这张图，我们可以理解为什么基于 lucene 可以快速进行倒排链的查找和 docid 查找，下面就来看一下有了这些后如何进行倒排链合并返回最后的结果。

## 倒排合并

假如我们的查询条件是 name = “Alice”，那么按照之前的介绍，首先在 term 字典中定位是否存在这个 term，如果存在的话进入这个 term 的倒排链，并根据参数设定返回分页返回结果即可。这类查询，在数据库中使用二级索引也是可以满足，那 lucene 的优势在哪呢。假如我们有多个条件，例如我们需要按名字或者年龄单独查询，也需要进行组合 name = "Alice" and age = "18" 的查询，那么使用传统二级索引方案，你可能需要建立两张索引表，然后分别查询结果后进行合并，这样如果 age = 18 的结果过多的话，查询合并会很耗时。那么在 lucene 这两个倒排链是怎么合并呢。  
假如我们有下面三个倒排链需要进行合并。

![](https://pic4.zhimg.com/v2-128de2f49b2e40369c10eb33911935fb_r.jpg)

在 lucene 中会采用下列顺序进行合并：

1.  在 termA 开始遍历，得到第一个元素 docId=1
2.  Set currentDocId=1
3.  在 termB 中 search(currentDocId) = 1 (返回大于等于 currentDocId 的一个 doc),  
    

1.  因为 currentDocId ==1，继续
2.  如果 currentDocId 和返回的不相等，执行 2，然后继续

1.  到 termC 后依然符合，返回结果
2.  currentDocId = termC 的 nextItem
3.  然后继续步骤 3 依次循环。直到某个倒排链到末尾。

整个合并步骤我可以发现，如果某个链很短，会大幅减少比对次数，并且由于 SkipList 结构的存在，在某个倒排中定位某个 docid 的速度会比较快不需要一个个遍历。可以很快的返回最终的结果。从倒排的定位，查询，合并整个流程组成了 lucene 的查询过程，和传统数据库的索引相比，lucene 合并过程中的优化减少了读取数据的 IO，倒排合并的灵活性也解决了传统索引较难支持多条件查询的问题。

## BKDTree

在 lucene 中如果想做范围查找，根据上面的 FST 模型可以看出来，需要遍历 FST 找到包含这个 range 的一个点然后进入对应的倒排链，然后进行求并集操作。但是如果是数值类型，比如是浮点数，那么潜在的 term 可能会非常多，这样查询起来效率会很低。所以为了支持高效的数值类或者多维度查询，lucene 引入类 BKDTree。BKDTree 是基于 KDTree，对数据进行按照维度划分建立一棵二叉树确保树两边节点数目平衡。在一维的场景下，KDTree 就会退化成一个二叉搜索树，在二叉搜索树中如果我们想查找一个区间，logN 的复杂度就会访问到叶子结点得到对应的倒排链。如下图所示：  

![](https://pic3.zhimg.com/v2-d9e0a158bf01d926e90f73d13fd5c4a2_r.jpg)

如果是多维，kdtree 的建立流程会发生一些变化。  
比如我们以二维为例，建立过程如下：

1.  确定切分维度，这里维度的选取顺序是数据在这个维度方法最大的维度优先。一个直接的理解就是，数据分散越开的维度，我们优先切分。
2.  切分点的选这个维度最中间的点。
3.  递归进行步骤 1，2，我们可以设置一个阈值，点的数目少于多少后就不再切分，直到所有的点都切分好停止。

下图是一个建立例子：  

![](https://pic2.zhimg.com/v2-e49807c7e849c8a1bf0ee300b349e871_r.jpg)

BKDTree 是 KDTree 的变种，因为可以看出来，KDTree 如果有新的节点加入，或者节点修改起来，消耗还是比较大。类似于 LSM 的 merge 思路，BKD 也是多个 KDTREE，然后持续 merge 最终合并成一个。不过我们可以看到如果你某个 term 类型使用了 BKDTree 的索引类型，那么在和普通倒排链 merge 的时候就没那么高效了所以这里要做一个平衡，一种思路是把另一类 term 也作为一个维度加入 BKDTree 索引中。

## 如何实现返回结果进行排序聚合

通过之前介绍可以看出 lucene 通过倒排的存储模型实现 term 的搜索，那对于有时候我们需要拿到另一个属性的值进行聚合，或者希望返回结果按照另一个属性进行排序。在 lucene4 之前需要把结果全部拿到再读取原文进行排序，这样效率较低，还比较占用内存，为了加速 lucene 实现了 fieldcache，把读过的 field 放进内存中。这样可以减少重复的 IO，但是也会带来新的问题，就是占用较多内存。新版本的 lucene 中引入了 DocValues，DocValues 是一个基于 docid 的列式存储。当我们拿到一系列的 docid 后，进行排序就可以使用这个列式存储，结合一个堆排序进行。当然额外的列式存储会占用额外的空间，lucene 在建索引的时候可以自行选择是否需要 DocValue 存储和哪些字段需要存储。

## Lucene 的代码目录结构

介绍了 lucene 中几个主要的数据结构和查找原理后，我们在来看下 lucene 的代码结构，后续可以深入代码理解细节。lucene 的主要有下面几个目录：

1.  analysis 模块主要负责词法分析及语言处理而形成 Term。
2.  codecs 模块主要负责之前提到的一些数据结构的实现，和一些编码压缩算法。包括 skiplist，docvalue 等。
3.  document 模块主要包括了 lucene 各类数据类型的定义实现。
4.  index 模块主要负责索引的创建，里面有 IndexWriter。
5.  store 模块主要负责索引的读写。
6.  search 模块主要负责对索引的搜索。
7.  geo 模块主要为 geo 查询相关的类实现
8.  util 模块是 bkd，fst 等数据结构实现。

## 最后

本文介绍了 lucene 中的一些主要数据结构，以及如何利用这些数据结构实现高效的查找。我们希望通过这些介绍可以加深理解倒排索引和传统数据库索引的区别，数据库有时候也可以借助于搜索引擎实现更丰富的查询语意。除此之外，做为一个搜索库，如何进行打分，query 语句如何进行 parse 这些我们没有展开介绍，有兴趣的同学可以深入 lucene 的源码进一步了解。

【转载自云栖社区】