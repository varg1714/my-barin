---
source: https://zhuanlan.zhihu.com/p/47951652
create: 2023-11-13 19:56
---
## 前言

Elasticsearch 是一个很火的分布式搜索系统，提供了非常强大而且易用的查询和分析能力，包括全文索引、模糊查询、多条件组合查询、地理位置查询等等，而且具有一定的分析聚合能力。因为其查询场景非常丰富，所以如果泛泛的分析其查询性能是一个非常复杂的事情，而且除了场景之外，还有很多影响因素，包括机型、参数配置、集群规模等等。本文主要是针对几种主要的查询场景，从查询原理的角度分析这个场景下的查询开销，并给出一个大概的性能数字，供大家参考。

## Lucene 查询原理

本节主要是一些 Lucene 的背景知识，了解这些知识的同学可以略过。

## Lucene 的数据结构和查询原理

Elasticsearch 的底层是 Lucene，可以说 Lucene 的查询性能就决定了 Elasticsearch 的查询性能。关于 Lucene 的查询原理大家可以参考以下这篇文章：

[本专栏文章：Lucene 查询原理](https://zhuanlan.zhihu.com/p/35814539)

Lucene 中最重要的就是它的几种数据结构，这决定了数据是如何被检索的，本文再简单描述一下几种数据结构：

1.  FST：保存 term 字典，可以在 FST 上实现单 Term、Term 范围、Term 前缀和通配符查询等。
2.  倒排链：保存了每个 term 对应的 docId 的列表，采用 skipList 的结构保存，用于快速跳跃。
3.  BKD-Tree：BKD-Tree 是一种保存多维空间点的数据结构，用于数值类型 (包括空间点) 的快速查找。
4.  DocValues：基于 docId 的列式存储，由于列式存储的特点，可以有效提升排序聚合的性能。

## 组合条件的结果合并

了解了 Lucene 的数据结构和基本查询原理，我们知道：

1.  对单个词条进行查询，Lucene 会读取该词条的倒排链，倒排链中是一个有序的 docId 列表。
2.  对字符串范围 / 前缀 / 通配符查询，Lucene 会从 FST 中获取到符合条件的所有 Term，然后就可以根据这些 Term 再查找倒排链，找到符合条件的 doc。
3.  对数字类型进行范围查找，Lucene 会通过 BKD-Tree 找到符合条件的 docId 集合，但这个集合中的 docId 并非有序的。

现在的问题是，如果给一个组合查询条件，Lucene 怎么对各个单条件的结果进行组合，得到最终结果。简化的问题就是如何求两个集合的交集和并集。

## 1. 对 N 个倒排链求交集

上面 Lucene 原理分析的文章中讲过，N 个倒排链求交集，可以采用 skipList，有效的跳过无效的 doc。

## 2. 对 N 个倒排链求并集

处理方式一：仍然保留多个有序列表，多个有序列表的队首构成一个优先队列 (最小堆)，这样后续可以对整个并集进行 iterator(堆顶的队首出堆，队列里下一个 docID 入堆)，也可以通过 skipList 的方式向后跳跃(各个子列表分别通过 skipList 跳)。这种方式适合倒排链数量比较少(N 比较小) 的场景。

处理方式二：倒排链如果比较多 (N 比较大)，采用方式一就不够划算，这时候可以直接把结果合并成一个有序的 docID 数组。

处理方式三：方式二中，直接保存原始的 docID，如果 docID 非常多，很消耗内存，所以当 doc 数量超过一定值时 (32 位 docID 在 BitSet 中只需要一个 bit，BitSet 的大小取决于 segments 里的 doc 总数，所以可以根据 doc 总数和当前 doc 数估算是否 BitSet 更加划算)，会采用构造 BitSet 的方式，非常节约内存，而且 BitSet 可以非常高效的取交 / 并集。

## 3. BKD-Tree 的结果怎么跟其他结果合并

通过 BKD-Tree 查找到的 docID 是无序的，所以要么先转成有序的 docID 数组，或者构造 BitSet，然后再与其他结果合并。

## 查询顺序优化

如果采用多个条件进行查询，那么先查询代价比较小的，再从小结果集上进行迭代，会更优一些。Lucene 中做了很多这方面的优化，在查询前会先估算每个查询的代价，再决定查询顺序。

## 结果排序

默认情况下，Lucene 会按照 Score 排序，即算分后的分数值，如果指定了其他的 Sort 字段，就会按照指定的字段排序。那么，排序会非常影响性能吗？首先，排序并不会对所有命中的 doc 进行排序，而是构造一个堆，保证前 (Offset+Size) 个数的 doc 是有序的，所以排序的性能取决于 (Size+Offset) 和命中的文档数，另外就是读取 docValues 的开销。因为 (Size+Offset) 并不会太大，而且 docValues 的读取性能很高，所以排序并不会非常的影响性能。

## 各场景查询性能分析

上一节讲了一些查询相关的理论知识，那么本节就是理论结合实践，通过具体的一些测试数字来分析一下各个场景的性能。测试采用单机单 Shard、64 核机器、SSD 磁盘，主要分析各个场景的计算开销，不考虑操作系统 Cache 的影响，测试结果仅供参考。

## 单 Term 查询

ES 中建立一个 Index，一个 shard，无 replica。有 1000 万行数据，每行只有几个标签和一个唯一 ID，现在将这些数据写入这个 Index 中。其中 Tag1 这个标签只有 a 和 b 两个值，现在要从 1000 万行中找到一条 Tag1=a 的数据(约 500 万)。给出以下查询，那么它耗时如何呢：

```
请求：
{
  "query": {
    "constant_score": {
      "filter": {
        "term": {
          "Tag1": "a"
        }
      }
    }
  },
  "size": 1
}'
响应：
{"took":233,"timed_out":false,"_shards":{"total":1,"successful":1,"skipped":0,"failed":0},"hits":{"total":5184867,"max_score":1.0,"hits":...}
```

这个请求耗费了 233ms，并且返回了符合条件的数据总数：5184867 条。

对于 Tag1="a" 这个查询条件，我们知道是查询 Tag1="a" 的倒排链，这个倒排链的长度是 5184867，是非常长的，主要时间就花在扫描这个倒排链上。其实对这个例子来说，扫描倒排链带来的收益就是拿到了符合条件的记录总数，因为条件中设置了 constant_score，所以不需要算分，随便返回一条符合条件的记录即可。对于要算分的场景，Lucene 会根据词条在 doc 中出现的频率来计算分值，并取分值排序返回。

目前我们得到一个结论，233ms 时间至少可以扫描 500 万的倒排链，另外考虑到单个请求是单线程执行的，可以粗略估算，一个 CPU 核在一秒内扫描倒排链内 doc 的速度是千万级的。

我们再换一个小一点的倒排链，长度为 1 万，总共耗时 3ms。

```
{"took":3,"timed_out":false,"_shards":{"total":1,"successful":1,"skipped":0,"failed":0},"hits":{"total":10478,"max_score":1.0,"hits":...}
```

## Term 组合查询

首先考虑两个 Term 查询求交集：

```
对于一个Term的组合查询，两个倒排链分别为1万和500万，合并后符合条件的数据为5000，查询性能如何呢？
请求：
{
  "size": 1,
  "query": {
    "constant_score": {
      "filter": {
        "bool": {
          "must": [
            {
              "term": {
                "Tag1": "a"  // 倒排链长度500万
              }
            },
            {
              "term": {
                "Tag2": "0" // 倒排链长度1万
              }
            }
          ]
        }
      }
    }
  }
}
响应：
{"took":21,"timed_out":false,"_shards":{"total":1,"successful":1,"skipped":0,"failed":0},"hits":{"total":5266,"max_score":2.0,"hits":...}
```

这个请求耗时 21ms，主要是做两个倒排链的求交操作，因此我们主要分析 skipList 的性能。

这个例子中，倒排链长度是 1 万、500 万，合并后仍有 5000 多个 doc 符合条件。对于 1 万的倒排链，基本上不进行 skip，因为一半的 doc 都是符合条件的，对于 500 万的倒排链，平均每次 skip1000 个 doc。因为倒排链在存储时最小的单位是 BLOCK，一个 BLOCK 一般是 128 个 docID，BLOCK 内不会进行 skip 操作。所以即使能够 skip 到某个 BLOCK，BLOCK 内的 docID 还是要顺序扫描的。所以这个例子中，实际扫描的 docID 数粗略估计也有几十万，所以总时间花费了 20 多 ms 也符合预期。

对于 Term 查询求并集呢，将上面的 bool 查询的 must 改成 should，查询结果为：

```
{"took":393,"timed_out":false,"_shards":{"total":1,"successful":1,"skipped":0,"failed":0},"hits":{"total":5190079,"max_score":1.0,"hits":...}
```

花费时间 393ms，所以求并集的时间是多于其中单个条件查询的时间。

## 字符串范围查询

```
RecordID是一个UUID，1000万条数据，每个doc都有一个唯一的uuid，从中查找0～7开头的uuid，大概结果有500多万个，性能如何呢？
请求：
{
  "query": {
    "constant_score": {
      "filter": {
        "range": {
          "RecordID": {
            "gte": "0",
            "lte": "8"
          }
        }
      }
    }
  },
  "size": 1
}
响应：
{"took":3001,"timed_out":false,"_shards":{"total":1,"successful":1,"skipped":0,"failed":0},"hits":{"total":5185663,"max_score":1.0,"hits":...}

查询a开头的uuid，结果大概有60多万，性能如何呢？

请求：
{
  "query": {
    "constant_score": {
      "filter": {
        "range": {
          "RecordID": {
            "gte": "a",
            "lte": "b"
          }
        }
      }
    }
  },
  "size": 1
}
响应：
{"took":379,"timed_out":false,"_shards":{"total":1,"successful":1,"skipped":0,"failed":0},"hits":{"total":648556,"max_score":1.0,"hits":...}
```

这个查询我们主要分析 FST 的查询性能，从上面的结果中我们可以看到，FST 的查询性能相比扫描倒排链要差许多，同样扫描 500 万的数据，倒排链扫描只需要不到 300ms，而 FST 上的扫描花费了 3 秒，基本上是慢十倍的。对于 UUID 长度的字符串来说，FST 范围扫描的性能大概是每秒百万级。

## 字符串范围查询加 Term 查询

```
字符串范围查询(符合条件500万)，加上两个Term查询(符合条件5000)，最终符合条件数目2600，性能如何？
请求：
{
  "query": {
    "constant_score": {
      "filter": {
        "bool": {
          "must": [
            {
              "range": {
                "RecordID": {
                  "gte": "0",
                  "lte": "8"
                }
              }
            },
            {
              "term": {
                "Tag1": "a"
              }
            },
            {
              "term": {
                "Tag2": "0"
              }
            }
          ]
        }
      }
    }
  },
  "size": 1
}
结果：
{"took":2849,"timed_out":false,"_shards":{"total":1,"successful":1,"skipped":0,"failed":0},"hits":{"total":2638,"max_score":1.0,"hits":...}
```

这个例子中，查询消耗时间的大头还是在扫描 FST 的部分，通过 FST 扫描出符合条件的 Term，然后读取每个 Term 对应的 docID 列表，构造一个 BitSet，再与两个 TermQuery 的倒排链求交集。

## 数字 Range 查询

```
对于数字类型，我们同样从1000万数据中查找500万呢？
请求：
{
  "query": {
    "constant_score": {
      "filter": {
        "range": {
          "Number": {
            "gte": 100000000,
            "lte": 150000000
          }
        }
      }
    }
  },
  "size": 1
}
响应：
{"took":567,"timed_out":false,"_shards":{"total":1,"successful":1,"skipped":0,"failed":0},"hits":{"total":5183183,"max_score":1.0,"hits":...}
```

这个场景我们主要测试 BKD-Tree 的性能，可以看到 BKD-Tree 查询的性能还是不错的，查找 500 万个 doc 花费了 500 多 ms，只比扫描倒排链差一倍，相比 FST 的性能有了很大的提升。地理位置相关的查询也是通过 BKD-Tree 实现的，性能很高。

## 数字 Range 查询加 Term 查询

```
这里我们构造一个复杂的查询场景，数字Range范围数据500万，再加两个Term条件，最终符合条件数据2600多条，性能如何？
请求：
{
  "query": {
    "constant_score": {
      "filter": {
        "bool": {
          "must": [
            {
              "range": {
                "Number": {
                  "gte": 100000000,
                  "lte": 150000000
                }
              }
            },
            {
              "term": {
                "Tag1": "a"
              }
            },
            {
              "term": {
                "Tag2": "0"
              }
            }
          ]
        }
      }
    }
  },
  "size": 1
}
响应：
{"took":27,"timed_out":false,"_shards":{"total":1,"successful":1,"skipped":0,"failed":0},"hits":{"total":2638,"max_score":1.0,"hits":...}
```

这个结果出乎我们的意料，竟然只需要 27ms！因为在上一个例子中，数字 Range 查询耗时 500 多 ms，而我们增加两个 Term 条件后，时间竟然变为 27ms，这是为何呢？

实际上，Lucene 在这里做了一个优化，底层有一个查询叫做 IndexOrDocValuesQuery，会自动判断是查询 Index(BKD-Tree) 还是 DocValues。在这个例子中，查询顺序是先对两个 TermQuery 求交集，得到 5000 多个 docID，然后读取这个 5000 多个 docID 对应的 docValues，从中筛选符合数字 Range 条件的数据。因为只需要读 5000 多个 doc 的 docValues，所以花费时间很少。

## 简单结论

1.  总体上讲，扫描的 doc 数量越多，性能肯定越差。
2.  单个倒排链扫描的性能在每秒千万级，这个性能非常高，如果对数字类型要进行 Term 查询，也推荐建成字符串类型。
3.  通过 skipList 进行倒排链合并时，性能取决于最短链的扫描次数和每次 skip 的开销，skip 的开销比如 BLOCK 内的顺序扫描等。
4.  FST 相关的字符串查询要比倒排链查询慢很多 (通配符查询更是性能杀手, 本文未做分析)。
5.  基于 BKD-Tree 的数字范围查询性能很好，但是由于 BKD-Tree 内的 docID 不是有序的，不能采用类似 skipList 的向后跳的方式，如果跟其他查询做交集，必须先构造 BitSet，这一步可能非常耗时。Lucene 中通过 IndexOrDocValuesQuery 对一些场景做了优化。

最后结尾再放一个彩蛋，既然扫描数据越多，性能越差，那么能否获取到足够数据就提前终止呢，下一篇文章我会介绍一种这方面的技术，可以极大的提高很多场景下的查询性能。