---
source: https://mp.weixin.qq.com/s/wlh2AHpNLrz9dHxPw9UrkQ
create: 2024-08-16 16:30
read: false
---

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naKoTzV6Pb6MRQcWglvtKDZvFJYBLLDquB36QGuST9YGcbUFHW9OV1J43qHt860sUwK2iazokm6c7bw/640?wx_fmt=jpeg&from=appmsg)

阿里妹导读

本文主要围绕 ES 核心特性：分布式存储特性和分析检索能力，介绍了概念、原理与实践案例，希望让读者快速理解 ES 的核心特性与应用场景。

Elasticsearch 入门

Elasticsearch(ES) 是一种基于分布式存储的搜索和分析引擎，目前在许多场景得到了广泛使用，比如维基百科和 github 的检索，使用的就是 ES。ES 中不乏纷繁冗余的细节，而本文将关注其核心特性：分布式存储特性和分析检索能力。围绕这两大核心特性，本文将介绍其中的概念、原理与实践案例，希望让读者快速理解 ES 的核心特性与应用场景。

**核心概念**

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naJADJIfbibnekdJ4GzONb5VFibKgWszkI7eJfvbN56a1184CNmrpVh4bocQaG5c2o7vske5vV16OfVg/640?wx_fmt=other&from=appmsg)

## 分布式存储特性相关概念：

### 节点（Node）

节点就是单个的 Elasticsearch 实例，该实例运行的载体可以是物理服务器或虚拟机器。

### 集群（Cluster）

集群是节点的集合。

集群是一组运行在不同载体（物理或虚拟机器）上的一个或多个 Elasticsearch 节点的集合。在集群内，节点间相互合作，对数据进行存储和管理。

### 分片（Shards）

分片是索引的片段。

受制于单个 ES 节点的性能上限（内存、磁盘 IO 速度），如果数据以整块形式进行存储与管理，则无法足够快速地响应客户端的请求，因此 ES 将索引拆分为更小块的分片，以便分布式存储和并行处理数据。

### 副本（Replicas）

副本是分片的拷贝。

每个分片可以有零个或者多个副本，副本与分片对外都可以提供数据查询服务。副本的存在可以增加整个 ES 系统的高可用性，高并发性，因为分片能做到的事情，副本也能做到。计算机世界里没有银弹，副本的开销主要体现在数据同步成本的增加（每次数据更新时，都需要把分片上的数据变更同步到其他副本中）。

## 数据模型相关概念

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naJADJIfbibnekdJ4GzONb5VF12Sqe9QUM3p3KiaqicV3zK3aDtPavomqkh4j2MZN3sfC0zQibT6BtXibCw/640?wx_fmt=other&from=appmsg)

### 索引（Index）

索引由一个或多个分片组成。索引是 Elasticsearch 中的顶层数据容器，对应关系型数据库中的数据库模型。

### 类别（Type）

类别在较早的 Elasticsearch 版本中，一个索引可以包含多个类别，每个类别用于存储不同种类的文档。对应关系型数据库中的数据表。然而，在 Elasticsearch 7.0 以后，类别逐渐被弃用。原因是同一索引下，不同 type 的数据存储其他 type 的 field，包含大量空值，造成资源浪费。

### 文档（Document）

索引中的每一条数据叫作一个文档，它是一个 JSON 格式的数据对象。对应关系型数据库中的数据行。这一点与非关系型数据库 MongoDB 很类似，MongoDB 属于文档数据库，每条数据也是一个 BSON 文档（BinaryJSON）。非关系型数据库的文档相比关系型数据库的数据行，优势在于提供了更高的自由度，文档中可以方便地新增减字段，多个文档间也不要求字段完全一致。同时，文档也保留了一部分结构化存储的特性，对存储的数据进行了一定的结构化封装，而没有像 K-V 非关系型数据库那样完全抛弃数据的结构化。

## 分析检索能力相关概念

### 倒排索引（Inverted Index）

倒排索引是 Elasticsearch 中用于高效检索文档的关键数据结构。它是将文档中的每个单词映射到包含它的文档上。这种数据结构使得 Elasticsearch 能够高效地处理文本信息这类非结构化数据，相比传统关系型数据库的正排索引遍历整个数据表，它能够高效地进行文本检索与分析。

正排索引是从文档到关键字的映射（已知文档求关键字），倒排索引是从关键字到文档的映射（已知关键字求文档）。倒排索引由两个主要部分组成：词汇表（Vocabulary）和倒排列表（Inverted List）。词汇表存储了所有不同的单词，而倒排列表存储了每个单词文档中的分布情况。

### 分析器（Analyzer）

分析器是 Elasticsearch 用于进行文本预处理的组件。它的主要作用是将文本转化为可被倒排索引的单词（term）。分析通常由以下几个步骤组成：

*   分词（Tokenization）：将文本拆分成单词，对于英文，以空格为分界线来拆分单词。
    
*   标准化（Normalization）：对单词进行规范化，通常包括大小转小写、去除停用词等。
    
*   过滤（Filtering）：过滤掉特殊字符，例如移除特定字符、删除数字替换等。

在阿里云 DTS 数据同步工具中，可以选择一系列 ES 内置的分析器，但是 Elasticsearch 的内置分析器对于中文的支持较差，采取了暴力拆分每个中文单字的策略。如果希望对中文进行合适的分词，可以选择第三方分词器，比如 jieba 分词器。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naJADJIfbibnekdJ4GzONb5VFSpUBB4nicGpFzNb9SLe9MkzdRUqq1lYAGfVMTCcDPqklz5KhKf0ia0icQ/640?wx_fmt=other&from=appmsg)

**主要查询类型**

基于以上的核心概念，Elasticsearch 通过分布式存储结构和分析检索能力，支持并提供了多种不同类型的查询能力，用于满足各种检索需求。以下是 ES 中主要的查询类型：

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naJADJIfbibnekdJ4GzONb5VFSJy4vgD1X9DK3Ak5flSWEbJyg2KLribprBbwJ8JgKdsfwIYg0ibXnmtw/640?wx_fmt=other&from=appmsg)

## 单词级别查询

万丈高楼平地起，优秀的全文索引能力是由基础的单词查询能力支撑的。

*   Term Query（精确）
*   最基础的 ES 查询，把输入字符串全部看作一个完整的单词，然后去倒排索引表里面找。
*   Fuzzy Query（模糊）
*   带编辑距离的 term 查询。具体实现：给定一个模糊度（编辑距离），ES 会根据这个编辑距离，对原始的单词进行拓展，生成一系列候选的新单词。对每一个编辑距离内的新单词，做 term 查询。

## 全文级别查询

像使用 match 和 match_phrase 这样的高层查询都属于全文级别查询，全文级别查询是对多个 / 多种单词级别查询的封装。

*   match
*   match 是自适应的：
*   如果给定了模糊度参数 fuzziness，match 在单词级别查询上会调用 fuzzy querry；如果未给定此参数，则 match 在单词级别上会走 term query；
    
*   如果 analyzed，match 会对输入进行分词，把输入 "service_123456" 看成 "service" 和 "123456"；如果 not_analyzed，match 走完全匹配，把输入 "service_123456" 看成 "service_123456"。
*   match 查询的主要步骤：

i. 检查字段类型，查看字段是 analyzed 还是 not_analyzed；

1. 如果 analyzed，说明该字段已经被分析器处理过，match 会对输入进行分词；
2. 如果 not_analyzed，说明该字段未被分析器处理过，match 走完全匹配；

ii. 分析查询字符串，将输入字符串进行分词，对分出来的每个单词，根据是否设置了模糊度参数 fuzziness，选择走 term query 或者 fuzzy query；

iii. 文档评分计算。

*   match_phrase
*   在 match 查询的基础上，保证输入的单词之间的顺序不变才会命中，性能相比 match 会差一些。

## Bool 查询

用于实现复杂的组合查询逻辑，具体有四种：

*   should：或
    
*   must：且
    
*   must _not：非
    
*   filter：可以用于作为查询中的前置过滤条件，must 类似，好处是它不会参与计算相关性分数。

逻辑完备性：足够数量的或且非，可以实现任何逻辑。

## Term Query 的文档相关度得分计算方式

利用倒排索引，对于输入的单词，考虑每个文档的以下指标：

*   TFIDF
*   目的：用文档中的一个单词，在一堆文档中区分出该文档；
    
*   TFIDF = TF * IDF；
    
*   TF（term frequency）：词频。表示单词在该文本中出现的频率（单词在该文本中出现的多不多）；
    
*   IDF（inverse document frequency）：反向文档频率。 表示单词在整个文本集合中出现的频率（有多少文本包含了这个词）的倒数，IDF 越大表示该词的重要性越高，反映了单词是否具有 distinguish 其所在文本的能力。
*   字段的长度
*   字段越短相关度越高；

综合这两个指标得出每个文档的相关度评分_score。

Elasticsearch 实践

**Elasticsearch 实现 nextToken 分页**

在服务端开发的实践中，由于数据量大，不可能一次请求一次查询就返回全部数据。因此，对数据进行分页查询是一种常见的工程实践。而由于 ES 方便处理非结构化字段的能力，常常被用作搜索框 API 中的主力分页查询。

在 ES 中，内置的分页机制为 sort+Search After 分页。它会对每次请求生成一个游标字段，这就相当于标记了上一页的结束位置，因此下次请求只要从上一次的游标字段开始，就能够方便地查找下一页。这实际上是 ES 官方提供的一种 nextToken 分页实现，它省略掉了构建游标这一过程，只需要使用者在查询条件中给定排序字段：

```
GET /service_version_index/service_version_type/_search
{
  "size": 100,
  "sort": [
    {"gmt_modified": "desc"},
    {"score": "desc"},
    {"id": "desc"}
  ],
  ...
}
```

sort+Search After 就能在查询结果中方便地生成每一页在特定排序上的游标。

```
{
  "sort" : [
  1614561419000,
  "6FxZJXgBE6QbUWetnarH"
  ]
}
```

下次查询带上这个游标，就可以快速定位到上一页的结束位置，开始下一页的查询：

```
GET /service_version_index/service_version_type/_search
{
  "size": 100,
  "sort": [
    {"gmt_modified": "desc"},
    {"score": "desc"},
    {"id": "desc"}
    ],
  "query": {
    ...
    ...
  },
  "search_after": [
    1614561419000,
    "6FxZJXgBE6QbUWetnarH"
  ]
}
```

我们也可以手动构建查询条件，手动实现 nextToken 分页条件。即使你不熟悉 nextToken 分页和 ES 查询的具体用法，你也应该能做出以下判断：你一定可以用 ES 的查询条件实现任意的 nextToken 分页逻辑。理由是 ES 的 Bool 查询具有逻辑完备性。

具体地，一个简单的 if-else 逻辑，可以用 ES 的基础查询来复现：

if A then B else C => (A must B) should (must not A must C)

类似地，任意复杂的查询条件，都可以实现了。

这样做的好处是，如果项目中涉及到多种数据库的分页，则后端代码的分页逻辑可以共用，只需要在不同的数据库中实现相同的 nextToken 条件：

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naJADJIfbibnekdJ4GzONb5VFW3fia2f0iak1plXjkAvH3qUxlCMhtUnvK7HT2AF6TQGw9wsatsDGHMZA/640?wx_fmt=other&from=appmsg)

在上述项目中，一个 API 服务使用到了两条查询链路：一条纯 MySQL 的查询链路，另一条 ES 分页 + MySQL 补字段的查询链路。由于在 ES 分页中，我们已经手动复现了 MySQL 中的 nextToken 分页条件。因此，在查询结束后，封装 nextToken 分页请求的这部分后端逻辑就可以实现复用。如果 MySQL 基于自实现的 nextToken 分页而 ES 使用官方推荐的 Sort 分页，则复用性较差，需要两套分页逻辑。

**Elasticsearch 关联查询与数据同步**

## 关联查询方案：

ES 与非关系型的文档数据库类似，基于文档存储数据，没有固定的表结构。关系型数据库以二维表结构的形式来组织数据，并擅长提供对数据表间关系的管理。而 ES 以文档为数据的组织形式，进行扁平化存储，它不擅长进行关系管理而擅长对扁平化的文档进行文本检索。

在 ES 中，由于其分布式存储特性和非关系性数据模型，类似关系型数据库中 JOIN 联表查询这样的操作将非常不便。ES 内置了类似 MySQL 的 JOIN 的关联查询实现：父子文档，但它存在功能和性能上的限制：父子文档需要在在同一个分片中，额外实现关系管理需要的成本。ES 官方通常不建议使用这种方式。

在 ES 中，如果要实现关联查询，最佳实践一般为构建宽表或采取服务端 JOIN 这种折中方案。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naJADJIfbibnekdJ4GzONb5VFqlnoyZDHnW0XVk0RIiajfpKFXnF2KOzYOQrnicGqKdodsvaAv8rXjO4Q/640?wx_fmt=other&from=appmsg)

### 服务端 JOIN

ES 中分两个索引来存储数据，查询时在服务端的业务代码内进行两次查询，将第一次查询的结果作为第二次查询的条件。

好处：实现容易，数据量少时用户体验好。

坏处：数据量大时，两次查询会带来额外的开销，因为每次查询都需要建立连接、发送请求......

拓展：如果涉及不同数据库之间的关联查询，也可以采用此方案，比如用 ES 处理有限的文本字段，查得一个 id 列表，然后把这个 id 列表给 MySQL 的完整查询作为条件，补齐剩下的字段。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naJADJIfbibnekdJ4GzONb5VFW3fia2f0iak1plXjkAvH3qUxlCMhtUnvK7HT2AF6TQGw9wsatsDGHMZA/640?wx_fmt=other&from=appmsg)

### 宽表冗余存储：

宽表：通俗地讲就是字段很多的数据库表。指的是把特定的查询业务需求所需要的全部字段都关联在一起的一张数据库表。由于关联了大量冗余字段，宽表已经不符合数据库设计的三范式，而因此获得的好处就是查询性能的提高与关联查询的简化（避开了查询时 JOIN）。这是一种典型的空间换时间的优化思路。但宽表不便扩展，如果业务需求有变化，哪怕是需要新增一个字段，都需要变更宽表。

窄表：严格按照数据库设计三范式设计的数据表。这种表的设计形式减少了数据冗余，但是实现一个复杂查询要使用很多张表，涉及多表 JOIN 问题，可能会影响性能。其特点是方便扩展，多个窄表可以组合并适应多种业务场景，无论有多少不同的场景，都不用修改原本的表结构。但在查询逻辑和代码逻辑上需要进行封装。

如果对查询速度性能要求较高，建议选择宽表。

## 数据同步

由于 ES 擅长检索而不是存储，业务场景中很少会以 ES 作为主力做数据存储，而是使用关系型数据库进行存储，在需要 ES 时再构建需要的数据并进行同步。具体来说，有手动写入和数据同步工具等方案。

### 手动写入

在已有的业务逻辑中，同步或异步地增加对 ES 的增删改查。实现简单，但不利于扩展，耦合性较强。

### 数据同步工具

阿里云 DTS：

是阿里云提供的一种云服务产品，基于 binLog 模拟主从复制实现数据同步。一对一数据同步方便高效，但对多表 JOIN 场景无法支持。如果表结构出现变更，则需要手动删除目标 ES 库，重建同步任务。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naJADJIfbibnekdJ4GzONb5VFhLF2qcVzStLm2ibVqib3o3JZ3iakUspQtZicfcU142zDHPVJtAzmRvBJibQ/640?wx_fmt=other&from=appmsg)

如果我们搭建一个中间层，将多表 JOIN 结果先写入一个冗余的 MySQL 宽表，再同步到 ES，则数据同步可以使用简单高效的 DTS。代价是整个同步链路环节增多，不稳定性增加了。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naJADJIfbibnekdJ4GzONb5VFoXFKWQBe3fCvzaY2ErJfo7pbyAqs1dcNiaAFibIbMJdtrAYszpmeGXrQ/640?wx_fmt=other&from=appmsg)

ES 套件 Logstash：

是 ES 官方套件系列中的数据同步工具。支持在 config 配置文件中写入需要的 SQL 逻辑并存储为视图，并通过视图来写入多表查询的结果到 ES。这种方式自由度较高，能够方便地构建所需要的数据。但是数据同步的性能略差（秒级别）。

```
CREATE VIEW my_view AS
SELECT sv.*, s.score, sc.category
FROM service_version sv
JOIN service s ON sv.service_id = s.service_id
JOIN service_category sc ON s.service_id = sc.service_id;
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naJADJIfbibnekdJ4GzONb5VFLSpnic9VJhD1SOLT4r66GtSKibVCeVzd1j34jg20EY2NbSMfSdn8B6Cg/640?wx_fmt=other&from=appmsg)

其他数据同步工具选型指南：

 https://help.aliyun.com/zh/es/use-cases/select-a-synchronization-method 

## 参考：

[1] 网易基于 Elasticsearch 构建通用搜索系统的实践 - 分享 - Elastic 中文社区﻿：https://elasticsearch.cn/slides/243#page=20

[2] RDSMySQL 同步方案_检索分析服务 Elasticsearch 版 - 阿里云帮助中心﻿：https://help.aliyun.com/zh/es/use-cases/select-a-synchronization-method

[3] Elasticsearch Guide [8.9] | Elastic﻿：https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html

**移动开发秘籍：云上高效构建 App** 

本方案使用阿里云多端低代码开发平台魔笔低代码快速搭建适配于微信、支付宝等多平台的小程序，帮助您提升开发效率、降低维护成本。

点击阅读原文查看详情。