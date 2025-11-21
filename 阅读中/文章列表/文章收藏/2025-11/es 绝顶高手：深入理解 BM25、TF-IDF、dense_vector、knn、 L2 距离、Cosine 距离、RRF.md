---
source: https://mp.weixin.qq.com/s/A51bcpZW8fYvmAYl5KJVbg
create: 2025-11-05 17:34
read: false
knowledge: false
---
FSAC 未来超级架构师

架构师总动员  
实现架构转型，再无中年危机

## 尼恩说在前面

在 40 岁老架构师 尼恩的**读者交流群** (50+) 中，最近有小伙伴拿到了一线互联网企业如阿里、滴滴、极兔、有赞、希音、百度、网易、美团的面试资格，遇到很多很重要的 es 面试题：

**(1) 核心算法对比分析题**

请从算法原理、适用场景、性能特点三个维度对比： L2 距离与余弦距离的数学本质区别？dense_vector 与传统倒排索引的工作原理差异?

**(2) 完整搜索流程设计题**

为一个内容平台设计搜索功能，支持标题关键词搜索和内容语义搜索。请描述从文档处理、向量生成、索引构建到查询响应的完整技术流程。()

**(3) 高维向量性能优化题**

当向量维度从 384 维增加到 1536 维时，KNN 搜索性能显著下降。请分析根本原因，并从算法层和工程层各提出两种优化方案。

**(4) 混合搜索架构设计题**

设计一个电商搜索系统，要求同时支持关键词搜索（商品名称）和语义搜索（商品描述相似度）。请给出具体的索引 mapping 设计，并详细说明 BM25 与 KNN 的分数融合策略。

**(5) 大规模混合搜索系统设计题**

设计一个日查询量千万级的混合搜索系统，需要同时处理文本搜索和向量搜索。请描述集群架构、索引分片策略、缓存设计和性能监控方案。

最近又有小伙伴在面试大厂，都遇到了相关的面试题。虽然 回答了一些边边角角，但是回答不全面不体系，面试官不满意，面试挂了。

主要的原因，是对 es 的  BM25、TF-IDF、dense_vector、knn、 L2 距离、Cosine 距离、RRF 底层原理理解不够。

接下来，尼恩搞一篇文章，帮助大家深入理解，成为 es 顶尖高手。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WNFEKtEYFAR6BSwUicWTYuVS7SaR2cCkq7Rfrt6Z3owbUrdDPFLicwwriczmKb67vejXfCkuh3IrBzVQ/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=0)

借着此文，尼恩给大家做一下系统化、体系化的梳理，使得大家内力猛增，展示一下雄厚的 “技术肌肉、技术实力”，**让面试官爱到 “不能自已、口水直流”**，然后实现”offer 直提，offer 自由”。

当然，这道面试题，以及参考答案，也会收入咱们的 《尼恩 Java 面试宝典 PDF》V140 版本，供后面的小伙伴参考，提升大家的 3 高 架构、设计、开发水平。

《尼恩 架构笔记》《尼恩高并发三部曲》《尼恩 Java 面试宝典》的 PDF，请到文末公号【技术自由圈】获取

## Elasticsearch   BM25 到混合搜索 底层原理

在信息检索领域，Elasticsearch 作为领先的搜索引擎，提供了从传统关键词搜索到现代语义搜索的完整解决方案。

本文将深入解析 BM25、TF-IDF、dense_vector、kNN、L2 距离和余弦距离这六大核心概念，并重点探讨如何在实践中实现传统搜索与向量搜索的协同工作。

## 第一部分：传统文本检索算法

想象一下，你要从 1000 篇文章里找出关于 “人工智能” 的好文章。

如果只是看有没有出现 “人工智能” 这个词，那很容易被刷屏干扰——有些文章可能通篇堆砌这个词，但其实啥也没说清楚。

所以，我们需要一个标准：**既能反映一个词在某篇文章里的出现频率，又能看出它是 “稀有关键词” 还是“谁都能用的口水词”**。

### 1.1 TF-IDF：经典的关键词权重算法

这就催生了 **TF-IDF**（读作 “T-F-I-D-F”），全称是 _Term Frequency - Inverse Document Frequency_，翻译过来就是：“词频 × 逆文档频率”。

它的核心思想很简单：

一个词越是在这篇文档里频繁出现，同时又很少出现在其他文档中，那它就越能代表这篇文档的特点。

就像你在一群人里听到有人说 “量子纠缠”，你会觉得这个人很特别；但如果所有人都在谈这个话题，那这个词就没那么“亮眼” 了。

**TF-IDF** 简单说就是：

*   某个词在一个文档里出现得越多（TF 高），说明这篇文档越相关；
    
*   但如果这个词在所有文档里都常见（比如 “的”、“是” 这类停用词），那它就不重要（IDF 低）。
    

所以 TF-IDF 就是综合考虑 “出现次数” 和“稀缺性”，给每个词打分，最后加起来判断相关性。

优点：简单有效，适合基础搜索场景  
 缺点：无法处理同义词、近义词；容易被关键词堆砌欺骗

#### 核心原理

TF-IDF（Term Frequency-Inverse Document Frequency）是传统文本检索中衡量 "词对文档重要性" 的经典算法。

TF-IDF 核心思想是：一个词对文档的重要性，与它在文档中出现的频率成正比，与它在所有文档中出现的频率成反比。

**TF（词频）计算**：

```
TF = 词在文档中出现的次数


```

实际应用中通常会对词频进行归一化处理，如除以文档总词数，以避免长文档的偏差。

**IDF（逆文档频率）计算**：

```
IDF = log(总文档数 / (包含该词的文档数 + 1))


```

其中的 +1 是为了避免分母为 0 的情况。

**最终得分**：

```
TF-IDF = TF × IDF


```

分数越高，说明该词对当前文档的 "区分度" 越强。

#### 在 ES 中的历史地位

TF-IDF 是 ES 早期版本（5.x 之前）的默认相似度算法，主要用于关键词搜索的相关性排序。

虽然算法简单直观，但存在明显局限性：

*   仅关注词的 "出现频率"，无法理解语义（如 "电脑" 和 "计算机" 被视为完全不同的词）
    
*   对词频的增长没有限制，当词频过高时，分数可能不合理
    
*   缺乏对文档长度的考虑，长文档天然具有优势
    

### 1.2 BM25：改进的最佳匹配算法

#### 算法演进：从 “朴素统计” 到“精细调控”

后来工程师们发现 TF-IDF 还不够用， 在真实的大规模文本中，比如网页、商品、日志等。于是就有了 **BM25**（Best Matching 25），它是目前 Elasticsearch 默认的评分算法。

如果说 TF-IDF 是个老实的学生，只会死记硬背数字，那 BM25 就是个会思考的老手，懂得 “适可而止” 和“因地制宜”。

BM25 全称是 **Best Match 25**，名字听着有点神秘，其实是上世纪 90 年代英国布里斯托大学的研究成果编号（第 25 个实验模型）。没想到几十年后，它成了现代搜索引擎的基石之一。

从 **Elasticsearch 5.x 开始**，BM25 正式取代 TF-IDF 成为默认相似度算法。

BM25 其实是在 TF-IDF 的基础上做了几个关键优化：

*   **对词频做饱和处理**：以前一个词出现 100 次就比出现 10 次强 10 倍？BM25 不这么认为。它觉得重复太多也没必要无限加分，就像一个人喊破喉咙也不能代表他说得对。所以它用了数学函数让分数增长变慢（叫 “词频饱和”）。BM25 引入了一个 “天花板效应”：**词频越高，新增一次出现带来的收益越小**。
    
*   **考虑文档长度**：短文档里某个词出现几次就很突出；长文档里即使出现多次也可能只是正常描述。BM25 会自动调整权重，避免短文档吃亏。这部分的作用是：**当文档比平均水平长时，适当降低每个词的权重；短文档则略有加分。**
    
*   **可调参数更多**：比如 `k1` 控制词频影响程度，`b` 控制文档长度归一化强度，可以根据业务微调。
    

总结：BM25 更贴近现实，不容易被刷榜，效果稳定，因此成了 Elasticsearch 的默认选择。

#### 算法演进

BM25（Best Match 25）是对 TF-IDF 的重大改进，自 ES 5.x 版本起成为默认相似度算法。

BM25（Best Match 25） 通过引入 "词频饱和" 和 "文档长度归一化" 机制，解决了 TF-IDF 的主要缺陷。

```
// 示例：使用 BM25 进行搜索
GET /products/_search
{
  "query": {
    "match": {
      "description": "轻薄透气 连衣裙"
    }
  }
}


```

这个查询背后，Elasticsearch 就已经在用 BM25 计算每条记录的相关性得分 `_score` 了。

#### 数学公式

BM25 的完整评分公式为：

```
score(d, q) = Σ [ IDF(t) × (TF(t,d) × (k1 + 1)) / (TF(t,d) + k1 × (1 - b + b × |d| / avg_dl)) ) ]


```

其中：

*   `d`代表当前文档，`q`代表查询，`t`是查询中的词项
    
*   `k1`：控制词频饱和速度的参数（默认值 1.2）
    
*   `b`：控制文档长度影响的参数（默认值 0.75）
    
*   `|d|`：当前文档的长度
    
*   `avg_dl`：所有文档的平均长度
    

#### 核心改进

**1. 词频饱和（Term Frequency Saturation）**

BM25 通过 `k1`参数控制词频对得分的影响。

当词频达到一定阈值后，其对得分的贡献会趋于饱和，避免单个高频词过度影响最终结果。

BM25 引入了一个 “天花板效应”：**词频越高，新增一次出现带来的收益越小**。

这就像吃包子：

*   第一个包子让你很满足；
    
*   第五个就开始饱了；
    
*   第十个已经想吐了。
    

BM25 中的 `k1` 参数就是控制这个 “饱腹感” 的开关：

*   `k1` 越小 → 饱得越快（词频很快饱和）
    
*   `k1` 越大 → 更能忍受高频词（允许更多贡献）
    

默认值 `1.2` 是经过大量实验得出的经验值，在大多数场景下表现均衡。

**2. 文档长度归一化（Document Length Normalization）**

通过 `b`参数调整文档长度对得分的影响：

*   `b=0`：完全忽略文档长度的影响
    
*   `b=1`：完全考虑文档长度的影响
    
*   默认值 `b=0.75`在两者之间取得平衡
    

长文档中出现的词项会获得适当的权重惩罚，避免长文档在搜索结果中天然占优。

这部分的作用是：**当文档比平均水平长时，适当降低每个词的权重；短文档则略有加分。**

举个例子：

*   一篇 5000 字的技术综述 vs 一篇 800 字的精炼指南。同样出现 “卷积神经网络”5 次，你觉得哪个更有价值？
    
*   大概率是那篇短的——因为在有限篇幅里还提到了关键术语，说明主题更聚焦。
    

#### 实际应用

```
// 在字段映射中自定义 BM25 参数
PUT /my_index
{
  "mappings": {
    "properties": {
      "content": {
        "type": "text",
        "similarity": {
          "type": "BM25",
          "k1": 1.3,
          "b": 0.8
        }
      }
    }
  }
}


```

## 第二部分：现代向量搜索技术

在传统的搜索引擎中，比如你输入 “苹果手机”，系统会把这句话拆成“苹果” 和“手机”两个词，然后去查找哪些文档里包含了这些关键词。

这种做法叫**关键词匹配**，它依赖的是词语是否出现、出现了多少次。

关键词匹配， 是基于 “词是否出现” 来判断相关性的，属于**稀疏向量模型**（Sparse Vector Model）。

但问题是——语言是有 “意思”、语境的。

“苹果”可以是水果，也可以是公司；“手机”和 “智能手机” 虽然用词不同，但意思接近。

**关键词匹配**，很难理解这些语义上的细微差别。 没法 很好 解 决 同义词、语义相似性 的问题。

有！这就是近年来大火的 **“稠密” 向量**技术。

##### 问题背景分析：我们为什么需要 “稠密” 向量？

简单说，就是把一句话变成一串数字（比如 768 个浮点数），这串数字能捕捉到它的 “含义”。

这个过程叫做**嵌入（embedding）**，而结果就是所谓的**稠密向量（dense vector）**。

所以，“稠密” 的意思就是：每个维度都有信息，几乎没有零值，不像老式的稀疏向量那样大部分都是 0。

### 2.1 dense_vector：稠密向量存储

现在的 AI 模型（比如 BERT、Sentence-BERT）可以把一句话压缩成一个固定长度的数字数组，叫做**嵌入向量**（embedding），存放在 Elasticsearch 中对应的数据类型就是 `dense_vector`。

举个例子：

<table><thead><tr><td><span><strong><span leaf="">文本</span></strong></span></td><td><span><strong><span leaf="">向量（简化示意）</span></strong></span></td></tr></thead><tbody><tr><td><section><span><span leaf="">“我喜欢吃苹果”</span></span></section></td><td><section><span><span leaf="">[0.8, 0.3, -0.1, 0.9]</span></span></section></td></tr><tr><td><section><span><span leaf="">“我爱吃水果”</span></span></section></td><td><section><span><span leaf="">[0.7, 0.4, 0.0, 0.85]</span></span></section></td></tr><tr><td><section><span><span leaf="">“今天天气真好”</span></span></section></td><td><section><span><span leaf="">[-0.2, 0.6, 0.9, 0.1]</span></span></section></td></tr></tbody></table>

你会发现前两句语义接近，它们的向量也更相似；第三句完全不同，向量差距大。这样机器就能通过 “算距离” 来判断语义相似度。

#### 基本概念

`dense_vector`是 ES 中用于存储稠密向量的专用字段类型。

标识一个词 “词是否出现” 的 稀疏向量模型 （Sparse Vector Model） 不同，dense_vector 稠密向量的每个维度都包含有意义的浮点数值。

dense_vector  可以高效地保存由 AI 模型生成的向量，比如：

*   文本经过 BERT 模型后输出的 768 维向量
    
*   图片通过 ResNet 提取出的 512 维特征
    
*   音频片段转换成的声音指纹
    

dense_vector  数据不再是简单的文字或数字，而是承载了深层语义的信息载体。

#### 技术特性

*   **存储格式**：固定长度的浮点型数组（如 `[0.12, 0.34, -0.56, ...]`）
    
*   **维度定义**：在映射时必须明确指定向量维度（如 `dims: 768`）
    
*   **应用场景**：存储文本、图像、音频等内容的嵌入向量
    

### 支持多种相似度计算方式：

*   `cosine`：看方向是不是一致（常用在文本上）
    
*   `l2_norm`：算欧氏距离（适合图像等空间分布明显的场景）
    
*   `dot_product`：点积，衡量两个向量 “共线程度”
    

注意：`dense_vector` 不支持全文检索！你不能对它做 “模糊搜索” 或者 “分词查询”。它是为“语义相似性” 服务的，不是为 “关键词查找” 设计的。

#### 映射配置示例

```
PUT /my_vector_index
{
  "mappings": {
    "properties": {
      "text_embedding": {
        "type": "dense_vector",
        "dims": 768,
        "similarity": "cosine"
      },
      "image_embedding": {
        "type": "dense_vector", 
        "dims": 512,
        "similarity": "l2_norm"
      }
    }
  }
}


```

解释一下上面这段配置：

*   创建了一个叫 `my_vector_index` 的索引。
    
*   它有两个向量字段：
    
*   `text_embedding`：用来存文本向量，长度 768，使用余弦相似度比较。
    
*   `image_embedding`：存图片特征，512 维，用 L2 距离判断远近。
    

##### 重要限制（别踩坑！）

但 `dense_vector` 也有自己的短板：

**(1) 不能倒排索引：没法像普通字段那样快速定位某个值是否存在。**

**(2) 不能分词处理：它就是一个整体的数组，不能拆开查其中某一部分。**

**(3) 不能单独用于全文搜索：你得配合其他字段一起用，比如原始文本字段做关键词过滤。**

**(4) 必须搭配 kNN 使用：只有通过向量相似性搜索才能发挥它的价值。**

所以记住一句话：**`dense_vector` 是为 “找相似” 而生的，不是为 “找关键词” 准备的。**

### 2.2 kNN：k 近邻搜索算法

##### 问题背景分析：怎么从 “一堆意义” 里找到最像的那个？

假设你现在有一万个句子都被转成了向量，存在 ES 里。用户输入一句：“我想买一部拍照好的手机”，你也把它变成一个向量。

现在的问题是：**在这 1 万条记录中，哪几条和这句话最 “像”？**

如果一个个去比对，算 1 万次距离，当然能得到最准确的结果。但如果数据变成百万级、千万级呢？每次都全表扫描，速度就太慢了。

这就引出了 **kNN（k-Nearest Neighbors）算法**——它的目标就是在高维空间里，快速找出和查询向量最接近的 k 个邻居。

#### 算法原理

kNN（k-Nearest Neighbors）是一种基于向量相似度的搜索算法，用于在向量空间中找到与查询向量最相似的 k 个文档。

**工作流程**：

**(1) 输入：查询向量（如用户查询文本通过嵌入模型生成的向量）**

**(2) 计算：计算查询向量与文档库中所有向量的相似度**

**(3) 排序：按相似度从高到低排序**

**(4) 输出：返回最相似的前 k 个文档**

这里的 “距离” 不一定是几何上的远近，而是语义上的差异。常见的有：

*   余弦距离（Cosine Distance）：看两个向量方向夹角有多大
    
*   欧氏距离（Euclidean Distance）：两点之间的直线距离
    
*   内积（Dot Product）：越大表示越相似
    

#### 实际应用示例

```
GET /my_vector_index/_search
{
  "knn": {
    "field": "text_embedding",
    "query_vector": [0.1, 0.2, ..., 0.768],
    "k": 10,
    "num_candidates": 100
  }
}


```

参数说明：

*   `field`：要搜索的 dense_vector 字段名
    
*   `query_vector`：查询向量
    
*   `k`：返回的最相似文档数量
    
*   `num_candidates`：每分片考虑的候选向量数，影响精度和性能
    

##### 实际应用示例

```
GET /my_vector_index/_search
{
  "knn": {
    "field": "text_embedding",
    "query_vector": [0.1, 0.2, ..., 0.768],
    "k": 10,
    "num_candidates": 100
  }
}


```

参数详解：

<table><thead><tr><td><span><strong><span leaf="">参数</span></strong></span></td><td><span><strong><span leaf="">含义</span></strong></span></td></tr></thead><tbody><tr><td><section><span><code><span leaf="">field</span></code></span></section></td><td><section><span><span leaf="">要搜索的向量字段名，必须是&nbsp;</span><code><span leaf="">dense_vector</span></code><span leaf="">&nbsp;类型</span></span></section></td></tr><tr><td><section><span><code><span leaf="">query_vector</span></code></span></section></td><td><section><span><span leaf="">用户查询对应的向量，必须和字段维度一致（这里是 768）</span></span></section></td></tr><tr><td><section><span><code><span leaf="">k</span></code></span></section></td><td><section><span><span leaf="">最终返回多少个最相似的结果（比如 top 10）</span></span></section></td></tr><tr><td><section><span><code><span leaf="">num_candidates</span></code></span></section></td><td><section><span><span leaf="">每个分片最多参与比较的候选数量，影响精度与速度</span></span></section></td></tr></tbody></table>

### 2.3 ES 中的 kNN 实现方式

Elasticsearch 提供了两种 kNN 实现方式，各有适用场景。

**1. 暴力搜索（Brute-force kNN）**

*   计算查询向量与所有文档向量的距离
    
*   优点：结果精确，100% 召回率
    
*   缺点：性能随数据量线性增长，只适合小规模数据集
    

**2. 近似 kNN（Approximate kNN）**

*   使用 HNSW（Hierarchical Navigable Small World）等算法建立索引
    
*   优点：搜索速度快，适合大规模数据
    
*   缺点：牺牲少量精度以换取性能提升
    

#### 1. 暴力搜索（Brute-force kNN）

顾名思义，就是 “硬刚”——逐个计算每一个向量和查询向量的距离。

优点：

*   结果绝对准确，召回率 100%
    
*   实现简单，无需额外索引
    

缺点：

*   性能随数据量线性增长
    
*   数据量超过几万条时，延迟明显上升
    

适用场景：小规模测试、验证模型效果、数据量小于 1 万条的情况。

#### 2. 近似 kNN（Approximate kNN）

这才是大规模应用的主力方案。它采用一种叫 **HNSW（Hierarchical Navigable Small World 分层小世界导航）** 的图算法来建立索引。

HNSW 是什么？

想象你在一座迷宫里找出口。如果没有地图，你就只能一间房一间房试过去（暴力搜索）。

但如果你有一张简化的 “高层导航图”，先跳到大致区域，再精细查找，就能快很多。

HNSW 就是这样一张 “多层导航图”：

*   最上层：节点少，连接跨度大，用于快速跳跃
    
*   越往下层：节点越多，连接更细，用于精确逼近
    

这样就可以在极短时间内找到 “差不多最近” 的几个点，牺牲一点点精度换来巨大的性能提升。

优点：

*   搜索速度快，百万级数据也能毫秒响应
    
*   支持实时插入新向量（动态更新）
    

缺点：

*   返回的结果是 “近似最优”，可能漏掉个别真正最近的点
    
*   建立索引需要时间和内存资源
    

适用场景：生产环境、数据量大、要求低延迟的应用，如推荐系统、语义搜索、图像检索等。

### 2.4 暴力搜索（Brute-force kNN） 和近似 kNN（HNSW）  DSL 示例

Elasticsearch 中**暴力搜索（Brute-force kNN）** 和**近似 kNN（HNSW）** 对标演示，通过字段映射和查询语句的差异体现两种搜索方式的区别：

#### 1. 暴力搜索（Brute-force kNN）示例

暴力搜索不需要为 `dense_vector` 字段建立索引（依赖全量向量计算），适用于小规模数据。

步骤 1：创建索引（不构建向量索引）

```
# 创建索引，dense_vector 字段不设置 index: true（默认不建立索引）
PUT brute_force_knn_index
{
  "mappings": {
    "properties": {
      "text_embedding": {
        "type": "dense_vector",
        "dims": 384  # 向量维度为 384
        # 不设置 index: true，即不构建 HNSW 索引，查询时会触发全量计算（暴力搜索）
      },
      "content": {
        "type": "text"
      }
    }
  }
}


```

不设置 index: true，即不构建 HNSW 索引，查询时会触发全量计算（暴力搜索）

步骤 2：插入测试数据（假设插入 5000 条文档，向量为随机生成）

```
POST brute_force_knn_index/_doc
{
  "content": "这是一篇测试文档",
  "text_embedding": [0.12, 0.34, -0.56, ..., 0.78]  # 384 维向量
}


```

步骤 3：执行暴力 kNN 搜索

由于 `text_embedding` 没有索引，ES 会遍历所有文档，逐个计算与查询向量的距离（暴力搜索）：

```
GET brute_force_knn_index/_search
{
  "knn": {
    "field": "text_embedding",  # 目标向量字段
    "query_vector": [0.11, 0.35, -0.55, ..., 0.77],  # 查询向量（与文档向量同维度）
    "k": 10,  # 返回 top 10 相似文档
    "num_candidates": 5000  # 候选文档数（等于总数据量，全量计算）
  },
  "fields": ["content"]  # 返回内容字段
}


```

**说明**：

*   因未建立索引，`num_candidates` 需设为总文档数（或更大），确保遍历所有向量；
    
*   数据量超过 1 万时，此查询延迟会明显上升（如 10 万条数据可能需要几秒）。
    

#### 2. 近似 kNN（HNSW）示例

近似 kNN 需要为 `dense_vector` 字段提前建立 HNSW 索引，适用于大规模数据。

步骤 1：创建索引（构建 HNSW 索引）

```
# 创建索引，dense_vector 字段设置 index: true（自动构建 HNSW 索引）
PUT approximate_knn_index
{
  "mappings": {
    "properties": {
      "text_embedding": {
        "type": "dense_vector",
        "dims": 384,
        "index": true,  # 关键：开启索引，ES 会自动用 HNSW 算法构建索引
        "similarity": "cosine"  # 指定相似度度量（余弦距离，适合语义向量）
      },
      "content": {
        "type": "text"
      }
    }
  }
}


```

关键：开启索引，ES 会自动用 HNSW 算法构建索引

步骤 2：插入大规模数据（假设插入 100 万条文档）

```
POST approximate_knn_index/_doc
{
  "content": "这是一篇生产环境文档",
  "text_embedding": [0.23, -0.45, 0.67, ..., -0.89]  # 384 维向量
}


```

步骤 3：执行近似 kNN 搜索

利用 HNSW 索引加速，无需遍历所有向量：

```
GET approximate_knn_index/_search
{
  "knn": {
    "field": "text_embedding",
    "query_vector": [0.22, -0.44, 0.66, ..., -0.88],  # 查询向量
    "k": 10,  # 返回 top 10 相似文档
    "num_candidates": 100  # 候选文档数（从 HNSW 索引中快速筛选 100 个候选，再精排）
  },
  "fields": ["content"]
}


```

**说明**：

*   `index: true` 触发 HNSW 索引构建，插入数据时会同步更新索引；
    
*   `num_candidates` 远小于总数据量（如 100 万数据设为 100-500），通过索引快速缩小范围，耗时可控制在毫秒级；
    
*   结果为近似最优（召回率通常 95%+），牺牲少量精度换取速度。
    

#### 暴力搜索（Brute-force kNN） 和近似 kNN（HNSW）  核心区别总结

<table><thead><tr><td><span><strong><span leaf="">维度</span></strong></span></td><td><span><strong><span leaf="">暴力搜索（Brute-force）</span></strong></span></td><td><span><strong><span leaf="">近似 kNN（HNSW）</span></strong></span></td></tr></thead><tbody><tr><td><section><span><span leaf="">映射配置</span></span></section></td><td><section><span><code><span leaf="">dense_vector.index: false</span></code><span leaf="">（默认）</span></span></section></td><td><section><span><code><span leaf="">dense_vector.index: true</span></code></span></section></td></tr><tr><td><section><span><span leaf="">索引依赖</span></span></section></td><td><section><span><span leaf="">无（全量计算）</span></span></section></td><td><section><span><span leaf="">依赖 HNSW 索引</span></span></section></td></tr><tr><td><section><span><code><span leaf="">num_candidates</span></code></span></section></td><td><section><span><span leaf="">需等于总数据量（全量遍历）</span></span></section></td><td><section><span><span leaf="">远小于总数据量（索引筛选）</span></span></section></td></tr><tr><td><section><span><span leaf="">性能（百万级数据）</span></span></section></td><td><section><span><span leaf="">秒级延迟</span></span></section></td><td><section><span><span leaf="">毫秒级延迟</span></span></section></td></tr><tr><td><section><span><span leaf="">结果精度</span></span></section></td><td><section><span><span leaf="">100% 准确</span></span></section></td><td><section><span><span leaf="">近似准确（95%+ 召回率）</span></span></section></td></tr></tbody></table>

通过映射中 `index` 参数的设置，可在 ES 中灵活切换两种 kNN 搜索方式，适配不同数据规模和精度需求。

## 第三部分：向量相似度度量方法

### 3.1. L2 距离（欧氏距离）

#### 数学定义

那它们之间的直线距离是多少？初中数学就学过——勾股定理！

$$\text{距离} = \sqrt{(4 - 1)^2 + (6 - 2)^2} = \sqrt{9 + 16} = \sqrt{25} = 5$$

这个算法推广到高维空间，就是所谓的 **L2 距离**，也叫**欧氏距离**。它的公式长这样：

```
L2(v1, v2) = √[(v1₁ - v2₁)² + (v1₂ - v2₂)² + ... + (v1ₙ - v2ₙ)²]


```

也就是说，把每个维度上的差值平方加起来，再开根号。

听起来有点抽象？其实它就是 “三维世界中两点间直线距离” 的高维版本。

哪怕你的向量有 384 个数（比如一个句子被编码成 384 维的向量），也能算出它们之间的 “直线距离”。

#### 特点分析

*   **对模长敏感**：向量的绝对大小直接影响距离计算结果
    
*   **几何直观**：符合人类对 "距离" 的直观理解
    
*   **适用场景**：图像特征匹配、用户行为统计向量等模长有实际意义的场景
    

### 3.2. 余弦距离（Cosine Distance）

想象两个向量从原点出发，指向不同的方向。它们夹角越小，说明方向越一致，也就越 “相似”。

这就是余弦相似度的核心思想：**只看方向，不看长短**。

#### 数学基础

余弦距离基于余弦相似度推导而来，衡量的是两个向量的方向一致性：

余弦相似度：

```
cosine_similarity(v1, v2) = (v1 · v2) / (||v1|| × ||v2||)


```

其中：

*   `v1 · v2` 是点积（对应元素相乘再求和）
    
*   `||v1||` 和 `||v2||` 分别是两个向量的长度（模）
    

结果范围在 [-1, 1] 之间：

*   1：完全同方向（非常相似）
    
*   0：垂直（毫无关联）
    
*   -1：完全相反
    

而**余弦距离**则是用 1 减去相似度，让它变成 “距离” 形式，越小越好：

```
cosine_distance = 1 - cosine_similarity


```

所以余弦距离的结果在 [0, 2] 范围内，0 表示完全相同方向，2 表示完全相反。

#### 核心特性

**1 方向敏感性**：这才是重点！无论向量多长，只要方向接近，就算相似。比如下面这三种情况：

*   “我喜欢猫”
    
*   “我真的很喜欢猫，每天都想撸它”
    
*   “I love cats”
    

这些句子长度不同、表达方式不同，但如果用了好的语言模型转成向量，它们的方向会非常接近。这时用余弦距离就能准确捕捉这种 “语义相似”。

**2 文本处理优势**：正是因为文本中经常出现长短不一但意思相近的情况，余弦距离成了 NLP 任务中的首选。不管是搜索引擎、问答系统还是推荐系统，只要你是在比 “意思像不像”，基本都在用它。

**3  归一化效果**：余弦计算自带 “标准化” 功能。相当于先把两个向量都压缩成单位长度（长度为 1），然后再比较角度。这样一来，就不会因为某个向量数值大就被误判为“更突出”。

L2 像是拿尺子量两个物体之间的直线距离；

而余弦更像是拿量角器看它们的 “朝向差异”。

### 3.3. 点积（Dot Product，也叫内积）

点积（Dot Product，也叫内积）是另一种衡量向量相关性的常用指标，它与余弦相似度、L2 距离的计算逻辑不同，核心是通过 “向量对应维度的乘积之和” 来反映相关性。

**点积的定义与计算**

两个向量 `v1 = [a1, a2, ..., an]` 和 `v2 = [b1, b2, ..., bn]` 的点积公式为：

```
点积(v1, v2) = a1b1 + a2b2 + ... + an*bn


```

简单说，就是把两个向量对应位置的元素相乘，再把所有乘积加起来。

**示例**：

*   向量 `v1 = [2, 3]`，`v2 = [4, 5]`，点积 = 2_4 + 3_5 = 8 + 15 = 23；
    
*   向量 `v3 = [1, 0]`（水平向右），`v4 = [0, 1]`（垂直向上），点积 = 1_0 + 0_1 = 0（垂直向量点积为 0）；
    
*   向量 `v5 = [3, 4]`（模长 5），`v6 = [6, 8]`（模长 10，与 v5 同方向），点积 = 3_6 + 4_8 = 18 + 32 = 50（同方向向量点积为正且较大）。
    

**点积如何反映 “相似性”？**

点积的结果大小与两个向量的**方向**和**模长**都相关：

*   **方向影响**：若两个向量方向相同（夹角 θ < 90°），点积为**正数**，且夹角越小（越相似），点积越大；
    
*   若方向垂直（θ = 90°），点积为 **0**（无相关性）；
    
*   若方向相反（θ > 90°），点积为**负数**（负相关）。
    
*   **模长影响**：即使两个向量方向相同，模长越大，点积也越大（比如上面的 v5 和 v6，同方向但 v6 模长更大，点积也更大）。
    

**点积与余弦相似度的关系**

余弦相似度的公式是：

```
余弦相似度(v1, v2) = 点积(v1, v2) / (||v1|| * ||v2||)


```

其中 `||v1||` 是 v1 的模长（L2 范数），`||v2||` 是 v2 的模长。

可以发现：

*   当两个向量**被归一化**（模长都为 1）时，`||v1|| = ||v2|| = 1`，此时 **点积 = 余弦相似度**。
    
*   这是一个非常重要的特性：在实际场景中（如语义向量），如果提前将向量归一化（模长为 1），点积可以直接替代余弦相似度，且计算更快（省去了除以模长的步骤）。
    

**点积的适用场景**

**(1) 向量已归一化的场景：**

若向量经过预处理（模长为 1，如 BERT 等模型的输出向量常被归一化），点积与余弦相似度等价，且计算成本更低（少了模长乘积的除法），适合高并发的向量检索（如推荐系统、语义搜索）。

**(2) 需要考虑模长的场景：**

当向量的模长有实际业务意义时（比如 “用户对商品的点击次数向量”，模长越大表示用户活跃度越高），点积可以同时体现 “方向相似性” 和 “模长强度”，此时比余弦相似度更合适。

### 3.4 距离度量选择策略

#### 选择指南

*   **向量已归一化**：优先选择 cosine 或 dot_product（计算速度最快）
    
*   **向量未归一化**：选择 l2 距离
    
*   **不确定状态**：cosine 是最通用的选择，特别适合文本语义搜索
    

#### mapping 配置

```
// 在 mapping 阶段确定 similarity 类型
"title_vector": {
  "type": "dense_vector",
  "dims": 384,
  "similarity": "cosine"  // 一旦设定无法修改
}


```

## 第四部分：混合搜索实战指南

**怎么让搜索引擎既懂 “关键词”，又懂 “语义”？**

传统的搜索引擎靠的是关键词匹配，比如你搜 “苹果手机”，它会去找文档里是否包含“苹果” 和“手机”这两个词。

这种技术叫 **BM25**，它是 Elasticsearch 的默认检索方式，效果不错，但有个缺点——它不懂语义。

比如你搜 “iPhone”，它可能就找不到标题写的是“苹果手机” 的文章，因为关键词不完全一样。

而现在的 AI 技术可以用向量来表示文本的意思。

比如把 “苹果手机” 和“Iphone”都转成一串数字（向量），虽然文字不同，但它们的向量很接近。

通过计算向量之间的相似度，就能找出语义上相关的内容。

这种方式叫 **KNN（K-Nearest Neighbors）** 检索，也就是找最相近的几个向量。

那有没有办法把这两种能力结合起来？

当然有！这就是我们说的 **混合索引（Hybrid Index）** ——在一个索引里，既能做关键词搜索，又能做语义搜索。

## 4.1 索引设计：同时支持 BM25 和 KNN

下面就是一个典型的混合索引配置，适用于 Elasticsearch + 向量插件（如 ES-KNN 插件或 OpenSearch）的场景。

#### 混合索引映射

```
PUT my-hybrid-index
{
  "settings": {
    "index": {
      "knn": true,
      "knn.algo_param.ef_construction": 128,
      "number_of_shards": 1
    }
  },
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "ik_max_word"
      },
      "content": {
        "type": "text",
        "analyzer": "ik_max_word"
      },
      "title_vector": {
        "type": "dense_vector",
        "dims": 384,
        "similarity": "cosine"
      },
      "content_vector": {
        "type": "dense_vector", 
        "dims": 768,
        "similarity": "cosine"
      }
    }
  }
}


```

#### 设置部分（settings）

*   `"knn": true`
    

这句话告诉 Elasticsearch：这个索引要支持向量检索功能。不开这个开关，后面定义的向量字段就没法用于近邻搜索。

*   `"knn.algo_param.ef_construction": 128`
    

这是构建向量索引时的一个参数，影响索引的速度和精度。简单理解就是：值越大，构建的向量索引越精确，但占用内存也越多。128 是一个比较平衡的选择。

*   `"number_of_shards": 1`
    

分片数量设为 1，主要是为了测试方便。真实生产环境可以根据数据量调整，比如几十万条数据可以保持为 1，上百万建议增加分片。

#### 映射部分（mappings）

这是最关键的部分，决定了每个字段怎么存储和使用。

*   `title` 和 `content`
    

类型是 `text`，用了 `ik_max_word` 分词器。这是中文搜索的关键！IK 分词器能把 “中国人民解放军” 拆成 “中国”“人民”“解放军” 等多个词，提升关键词匹配效果。

*   `title_vector`
    

存储标题的向量，维度是 384。为什么是 384？因为很多轻量级语义模型（比如 `paraphrase-multilingual-MiniLM-L12-v2`）输出的就是 384 维向量。适合短文本。

*   `content_vector`
    

存储正文的向量，维度是 768。通常对应像 `bert-base-chinese` 这类模型的输出，适合处理较长、更复杂的文本。

*   `"similarity": "cosine"`
    

表示用余弦相似度来计算向量之间的距离。这是语义搜索中最常用的衡量方式，值越接近 1，说明两个句子意思越像。

### 实际使用中要注意什么？

**(1) 向量得提前算好**

Elasticsearch 本身不会自动把文本变成向量。你需要在外面用模型（比如 BERT、Sentence-BERT）先把 `title` 和 `content` 转成向量，再写入 `title_vector` 和 `content_vector` 字段。

**(2) 资源消耗更高**

开启 KNN 功能后，内存消耗会上升，尤其是向量维度高、数据量大的时候。建议给节点配上足够的内存，并考虑专用的数据节点。

**(3) 查询时要组合两种方式**

后续做搜索的时候，不能只查向量，也不能只查关键词。要用 `bool` 查询把 BM25 和 KNN 结果融合起来，比如：

*   一部分分数来自关键词匹配（BM25）
    
*   一部分分数来自向量相似度（KNN）
    
*   最终按加权总分排序
    

**(4) 分词器选择很重要**

中文必须用 IK 这样的中文分词器，否则 “我喜欢机器学习” 会被当成一个整词，没法拆解，严重影响关键词检索效果。

### 索引设计 小结

<table><thead><tr><td><span><strong><span leaf="">能力</span></strong></span></td><td><span><strong><span leaf="">技术</span></strong></span></td><td><span><strong><span leaf="">作用</span></strong></span></td></tr></thead><tbody><tr><td><section><span><span leaf="">关键词匹配</span></span></section></td><td><section><span><span leaf="">BM25 + IK 分词</span></span></section></td><td><section><span><span leaf="">找出含有用户输入关键词的文档</span></span></section></td></tr><tr><td><section><span><span leaf="">语义理解</span></span></section></td><td><section><span><span leaf="">KNN + dense_vector</span></span></section></td><td><section><span><span leaf="">找出意思相近但关键词不同的文档</span></span></section></td></tr><tr><td><section><span><span leaf="">综合判断</span></span></section></td><td><section><span><span leaf="">混合查询</span></span></section></td><td><section><span><span leaf="">把两种结果融合，提升整体准确性</span></span></section></td></tr></tbody></table>

通过上面这个索引设计，我们就打造了一个 “既看得懂字，又读得懂意” 的搜索引擎基础框架。

接下来的任务，就是在查询阶段如何巧妙地把这两股力量拧成一股绳。

## 4.2 三种搜索模式详解

在实际的搜索系统中，我们常常会遇到不同的用户需求：

*   有的用户希望输入几个关键词就能快速找到完全匹配的结果（比如查文档标题），
    
*   有的则希望系统能 “理解” 他的意思，哪怕他用词不准确，也能推荐出相关内容（比如想找 “神经网络” 的资料，但搜了“AI 模型训练工具”）。
    

为了应对这些多样化的场景，现代搜索引擎提供了多种搜索方式。

下面我们来详细聊聊 Elasticsearch 中最常见的三种搜索模式——**纯 BM25 搜索、纯 KNN 搜索 和 混合搜索**，并解释它们各自适合什么情况，以及背后的逻辑是怎么回事。

#### 模式一：纯 BM25 搜索

```
GET my-hybrid-index/_search
{
  "query": {
    "match": {
      "title": "深度学习框架"
    }
  }
}


```

BM25 这种基于关键词的搜索，在以下场景依然不可替代：

*   精确查找已有信息（如产品名、人名、术语）
    
*   对响应速度要求极高（毫秒级返回）
    
*   数据本身结构清晰、关键词明确
    

**优点**：速度快、结果可解释性强、资源消耗低  
**缺点**：无法处理同义词、近义表达、语义相似等问题（比如搜 “手机” 不会返回 “智能手机” 或“移动设备”）

**适用场景总结**：适用于需要**快速、精准匹配关键词**的场景，比如电商商品搜索、内部知识库检索、日志查询等。

#### 模式二：纯 KNN 搜索

```
GET my-hybrid-index/_search
{
  "knn": {
    "field": "title_vector",
    "query_vector": [0.1, -0.2, ..., 0.384],
    "k": 10,
    "num_candidates": 100
  }
}


```

这就是所谓的 “语义搜索”。

**优点**：

*   支持语义理解和模糊匹配
    
*   可用于推荐系统、问答系统、跨语言 / 跨模态检索（比如图搜文）
    
*   用户即使表达不清也能得到合理结果
    

**缺点**：

*   计算开销大，尤其是数据量大的时候
    
*   结果有时 “太发散”，不够精确（比如搜“猫” 返回一堆动物图片）
    
*   需要额外部署模型生成向量，增加系统复杂度
    

**适用场景总结**：适合做**语义层面的理解和发现**，典型应用包括：

*   相似内容推荐（“你还可能感兴趣”）
    
*   客服机器人自动匹配 FAQ
    
*   多模态搜索（图文互搜）
    
*   新内容冷启动推荐（没有点击行为数据时）
    

#### 模式三：混合搜索（Hybrid Search）

**1. RRF 融合（ES 8.x+ 推荐）**

```
GET my-hybrid-index/_search
{
  "size": 20,
  "query": {
    "match": {
      "title": "深度学习"
    }
  },
  "knn": {
    "field": "title_vector",
    "query_vector": [0.1, -0.2, ..., 0.384],
    "k": 20,
    "num_candidates": 100
  },
  "rank": {
    "rrf": {
      "window_size": 100,
      "rank_constant": 20
    }
  }
}


```

RRF（Reciprocal Rank Fusion）是一种非常实用的融合策略。

它的基本思路是：

不管你是关键词搜出来的，还是向量搜出来的，只要你在某一边排得比较靠前，就有机会被最终选中。

具体做法是：

*   分别运行 BM25 和 KNN 搜索，各自得到一个排序列表；
    
*   对每个文档，计算它在这两个列表中的排名；
    
*   使用公式：`score = 1 / (rank + 常数)` 来融合分数（排名越靠前，贡献越大）；
    
*   最后统一排序，输出综合结果。
    

举个生活化的比喻：

就像评选优秀员工，HR 看绩效（关键词匹配），同事看口碑（语义相关），最后用一个公平规则把两方面评价加起来决定谁上榜。

参数说明：

*   `window_size`: 表示只考虑每种搜索前 N 名的结果进行融合（节省性能）
    
*   `rank_constant`: 平滑参数，防止排名第一的得分过高，压制其他候选
    

**优势**：

*   无需调参权重，自动平衡两种模式
    
*   Elasticsearch 原生支持，配置简单
    
*   在多数场景下表现稳定且鲁棒
    

**推荐使用版本 ES 8.8+**，因为早期版本对 RRF 支持不完善。

**2. 手动权重融合**

```
GET my-hybrid-index/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "match": {
            "title": {
              "query": "深度学习",
              "boost": 0.4
            }
          }
        },
        {
          "script_score": {
            "query": {
              "exists": {
                "field": "title_vector"
              }
            },
            "script": {
              "source": """
                // 自定义分数融合逻辑
                double knn_score = ...;
                return params.bm25_weight * _score + params.knn_weight * knn_score;
              """,
              "params": {
                "bm25_weight": 0.4,
                "knn_weight": 0.6
              }
            }
          }
        }
      ]
    }
  }
}


```

###### 这种方式更像是 “自己动手丰衣足食”

如果你对自己的业务非常了解，想精细控制关键词和语义的比重，就可以选择手动融合。

比如：

*   在电商搜索中，你可能更相信标题匹配（设 BM25 权重高一点）；
    
*   在内容推荐平台，你更看重语义关联（KNN 权重更高）；
    

这种方式允许你写脚本来自定义打分逻辑，灵活性极高。

但也带来问题：

*   需要不断调试权重（0.4 + 0.6 真的是最优吗？）
    
*   脚本性能较差，尤其数据量大时影响响应速度
    
*   维护成本高，后续换人难接手
    

## 4.3 融合策略深度解析

在 Elasticsearch（ES）中，**RRF（Reciprocal Rank Fusion， reciprocal rank 融合）** 是一种轻量、高效的结果融合算法

RRF 主要用于将多个不同来源的搜索结果（如关键词搜索的 BM25 结果、语义搜索的 kNN 结果）合并为一个综合排名，核心优势是**无需对不同来源的分数进行标准化**，极大降低了多策略融合的复杂度。

### RRF（Reciprocal Rank Fusion）原理

RRF 的核心思想是：**根据文档在各个独立结果列表中的 “排名” 而非 “原始分数” 来计算综合得分**，排名越靠前的文档在融合时权重越高。

文档 `d` 的 RRF 综合得分公式为：

```
RRF_score(d) = Σ [ 1 / (k + rank_i(d)) ]


```

其中：

*   `rank_i(d)`：文档 `d` 在第 `i` 个独立结果列表中的排名（排名从 1 开始，第 1 名表示最相关）；
    
*   `k`：平滑常数（经验值通常设为 60，作用是降低排名靠前文档的权重占比，避免单一结果列表的 Top 文档过度主导）；
    
*   求和符号 `Σ`：对所有参与融合的结果列表（如 BM25 列表、kNN 列表）的贡献值求和。
    

### 搞个例子  直观理解

例如，假设我们要融合两个结果列表（BM25 和 kNN）。

第一个 文档 `d` 的表现如下：

*   在 BM25 结果中排名第 2（`rank_1(d)=2`）；
    
*   在 kNN 结果中排名第 3（`rank_2(d)=3`）；
    
*   取 `k=60`。
    

则 `d` 的 RRF 得分为：`1/(60+2) + 1/(60+3) ≈ 0.0161 + 0.0159 ≈ 0.032`。

另一个文档 `d'` 在 BM25 中排名第 10，在 kNN 中排名第 1。

则 `d'` 得分是：`1/(60+10) + 1/(60+1) ≈ 0.0143 + 0.0164 ≈ 0.0307`。

因此 `d` 会排在 `d'` 前面。

可见，**在多个列表中排名都较好的文档，综合得分更高**，这符合 “多维度相关的文档更可能是用户需要的” 的直觉。

### RRF 的核心优势

**(1) 无需分数标准化**

不同搜索策略的分数体系差异极大（如 BM25 分数通常在 0-10 之间，kNN 的余弦距离可能在 0-2 之间），直接相加或加权毫无意义。

而 RRF 基于 “排名” 计算，避开了标准化难题，对输入来源的兼容性极强。

**(2) 对噪音不敏感**

单个结果列表中可能存在异常排名（如某文档因关键词匹配偶然排前），但 RRF 依赖多个列表的排名综合计算，单一列表的噪音影响被稀释，结果更稳健。

**(3) 计算轻量**

仅需知道文档在各列表中的排名，无需存储或处理原始分数，计算成本低，适合高并发场景。

**(4) 天然支持多源融合**

可同时融合 2 个及以上结果列表（如关键词搜索、语义搜索、用户行为过滤结果等），灵活扩展搜索策略。

### ES 中的 RRF 应用场景

RRF 在 ES 中最典型的用途是**混合搜索（Hybrid Search）**—— 融合 “关键词搜索（BM25）” 和 “语义搜索（kNN）” 的结果，兼顾 “精确匹配” 和 “语义相关”。

例如：

*   关键词搜索（BM25）擅长捕捉 “字面上的匹配”（如 “苹果手机” 匹配包含 “苹果” 和 “手机” 的文档）；
    
*   语义搜索（kNN）擅长捕捉 “意义上的相似”（如 “苹果手机” 匹配 “iPhone 设备”）；
    
*   用 RRF 融合两者，既保证字面相关的文档不丢失，又能补充语义相似的结果，提升搜索召回率和相关性。
    

### ES 中实现 RRF 的方式

ES 从 8.7 版本开始原生支持 RRF 融合（通过 `rank` 查询），也可通过自定义脚本实现。

以下是原生实现示例：

```
GET my_index/_search
{
  "query": {
    "rank": {  // RRF 融合查询
      "window_size": 50,  // 每个子查询取前 50 名参与融合（平衡精度和性能）
      "query": {
        "bool": {
          "should": [
            {  // 子查询 1：BM25 关键词搜索
              "match": {
                "title": "苹果手机"
              }
            },
            {  // 子查询 2：kNN 语义搜索
              "knn": {
                "title_embedding": {
                  "vector": [0.1, 0.2, ...],  // "苹果手机"的嵌入向量
                  "k": 50
                }
              }
            }
          ]
        }
      },
      "rank_script": {  // RRF 评分脚本
        "source": "double score = 0; for (int i = 0; i < params._ranks.length; i++) { score += 1.0 / (60 + params._ranks[i]); } return score;",
        "params": {
          "k": 60  // 平滑常数
        }
      }
    }
  }
}


```

*   `window_size`：控制每个子查询取多少文档参与融合（如 50 表示每个子查询取前 50 名），过小可能丢失优质文档，过大则增加计算成本；
    
*   `params._ranks`：ES 自动传入的数组，存储文档在每个子查询中的排名（如 `_ranks[0]` 是在 BM25 中的排名，`_ranks[1]` 是在 kNN 中的排名）。
    

### RRF 调优关键：平滑常数 k 的选择

`k` 是 RRF 中唯一需要调优的参数，其值直接影响排名的权重分配：

*   **`k` 越小**（如 10）：排名靠前的文档（如第 1 名）权重越高（`1/(10+1)≈0.09`），排名靠后的文档权重衰减快，适合 “优先突出单一列表中极相关的文档”；
    
*   **`k` 越大**（如 100）：排名的权重差异被拉平（第 1 名 `1/(100+1)≈0.0099`，第 10 名 `1/(100+10)≈0.0091`），适合 “均衡多个列表的贡献，避免单一列表主导”。
    

实际业务中，`k=60` 是经大量实践验证的经验值，可在此基础上通过 A/B 测试（结合用户点击率、转化率等指标）调整。

### 总结

RRF 是 ES 中实现多策略搜索结果融合的 “利器”，尤其适合关键词搜索与语义搜索的混合场景。其核心价值在于**无需标准化分数、实现简单、鲁棒性强**，通过合理设置 `k` 和 `window_size`，可在几乎不增加系统复杂度的前提下，显著提升搜索结果的相关性。

## 第五部分：高级应用与最佳实践

### 5.1 性能优化指南

#### 索引优化

```
{
  "settings": {
    "index": {
      "knn": true,
      "knn.algo_param.ef_construction": 128,    // 构建质量
      "knn.algo_param.m": 16,                   // 层间连接数
      "number_of_shards": 3,
      "number_of_replicas": 1
    }
  }
}


```

#### 查询优化参数

*   `num_candidates`：影响召回率和性能的关键参数
    
*   `k`：根据实际需求设置，避免不必要的计算
    
*   使用过滤器缩小搜索范围
    

### 5.2 常见问题解答

#### Q1：可以一个字段同时支持 BM25 和 KNN 吗？

**答案**：不可以。

BM25 需要 text/keyword 类型字段，KNN 需要 dense_vector 类型字段。必须建立两个独立的字段。

#### Q2：similarity 参数可以修改吗？

**答案**：不可以。similarity 参数在 mapping 阶段设定后无法修改，需要重新创建索引。

#### Q3：BM25 参数需要调优吗？

**答案**：大多数场景使用默认参数即可。

特殊场景下可以调整：

*   `k1`：控制词频饱和程度（1.2-2.0）
    
*   `b`：控制文档长度影响（0.0-1.0）
    

#### Q4：如何选择距离函数？

**答案**：

*   **文本向量**：优先选择 cosine
    
*   **归一化向量**：cosine 或 dot_product
    
*   **非归一化向量**：l2_norm
    
*   **不确定时**：cosine 最安全
    

### 5.3 实战场景示例

#### 电商搜索场景

```
// 商品搜索：关键词匹配 + 语义相似
GET products/_search
{
  "size": 50,
  "query": {
    "match": {
      "product_name": {
        "query": "智能手机",
        "boost": 0.6
      }
    }
  },
  "knn": {
    "field": "product_vector",
    "query_vector": [...],
    "k": 50,
    "num_candidates": 200,
    "boost": 0.4
  },
  "rank": {
    "rrf": {
      "window_size": 100
    }
  }
}


```

#### 内容推荐系统

```
// 基于内容相似性的推荐
GET articles/_search
{
  "knn": {
    "field": "content_vector", 
    "query_vector": [...],  // 当前文章向量
    "k": 10,
    "num_candidates": 100
  },
  "post_filter": {
    "bool": {
      "must_not": {
        "term": {
          "article_id": "current_article_id"
        }
      }
    }
  }
}


```

## 第六部分：总结与展望

### 6.1 技术选型矩阵

<table><thead><tr><td><span><strong><span leaf="">场景类型</span></strong></span></td><td><span><strong><span leaf="">推荐方案</span></strong></span></td><td><span><strong><span leaf="">配置要点</span></strong></span></td><td><span><strong><span leaf="">适用案例</span></strong></span></td></tr></thead><tbody><tr><td><section><span><strong><span leaf="">精确关键词搜索</span></strong></span></section></td><td><section><span><span leaf="">纯 BM25</span></span></section></td><td><section><span><span leaf="">使用默认参数</span></span></section></td><td><section><span><span leaf="">商品搜索、日志查询</span></span></section></td></tr><tr><td><section><span><strong><span leaf="">语义相似搜索</span></strong></span></section></td><td><section><span><span leaf="">纯 KNN</span></span></section></td><td><section><span><span leaf="">similarity=cosine</span></span></section></td><td><section><span><span leaf="">内容推荐、问答系统</span></span></section></td></tr><tr><td><section><span><strong><span leaf="">混合需求</span></strong></span></section></td><td><section><span><span leaf="">Hybrid Search</span></span></section></td><td><section><span><span leaf="">RRF 融合</span></span></section></td><td><section><span><span leaf="">智能搜索、知识库</span></span></section></td></tr><tr><td><section><span><strong><span leaf="">高性能要求</span></strong></span></section></td><td><section><span><span leaf="">优化 KNN 参数</span></span></section></td><td><section><span><span leaf="">调整 num_candidates</span></span></section></td><td><section><span><span leaf="">大规模实时搜索</span></span></section></td></tr></tbody></table>

### 6.2 核心原则总结

**(1) 职责分离：BM25 负责文字相关性，KNN 负责向量相似性**

**(2) 提前规划：距离函数在 mapping 阶段确定，无法修改**

**(3) 灵活组合：根据业务需求选择合适的搜索模式**

**(4) 持续优化：通过监控和测试不断调整参数配置**

### 6.3 未来发展趋势

随着大语言模型和多模态技术的发展，Elasticsearch 的混合搜索能力将进一步加强：

*   更智能的自动融合策略
    
*   实时向量更新和索引
    
*   多模态统一向量表示
    
*   与生成式 AI 的深度集成
    

通过深入理解这些核心技术原理和实践方法，开发者可以构建出更加智能、高效的搜索系统，满足不同业务场景下的复杂需求。

## 说在最后：有问题找老架构取经‍

按照此文的套路去回答，一定会 **吊打面试官，让面试官爱到 “不能自已、口水直流”**，然后实现”offer 直提”。

在面试之前，建议大家系统化的刷一波 5000 页《[尼恩 Java 面试宝典 PDF](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247497474&idx=1&sn=54a7b194a72162e9f13695443eabe186&scene=21#wechat_redirect)》，里边有大量的大厂真题、面试难题、架构难题。

很多小伙伴刷完后， 吊打面试官， 大厂横着走。

在刷题过程中，如果有啥问题，大家可以来 找 40 岁老架构师尼恩交流。

另外，如果没有面试机会， 可以找尼恩来改简历、做帮扶。前段时间，指导一个 **[被裁 1 年的 28 岁 / 6 年女程序员，收 3 大厂 offer  成 大厂 皇后 。2 本学历 惊天 逆涨，涨薪 2 倍 51W 年薪](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486433&idx=1&sn=0f630e8fc7620b3bf0e054827b48d91e&scene=21#wechat_redirect)**

跟着 尼恩  狠狠卷，实现 “offer 自由” 很容易的。

很多跟着 尼恩 **卷 硬核技术 的小伙伴 ， offer 拿到手软**， 实现真正的 “offer 自由” 。

## Java+AI  弯道超车： 跟着尼恩  卷 最新技术， 占据 技术领先地位，  没有危机 

 [会 AI 的程序员，工资暴涨 50%！](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486827&idx=1&sn=db3f268c6752ddb074a98c7b13a84011&scene=21#wechat_redirect)

  

[28 岁 / 6 年 / 被裁 1 年，收 3 大厂 offer ， 成 大厂 皇后 。2 本学历 51W 年薪，惊天 逆涨，涨薪 2 倍](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486987&idx=1&sn=ff977f450dd242446f228d3a6585e258&scene=21#wechat_redirect)，大厂皇后

  

[涨薪传奇： 18k->38K , 单月暴 20K，32 岁小伙伴 2 个月时间年薪 翻 1.5 倍 ，一步登天 + 逆天改命](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486960&idx=1&sn=f57253a448694c32e834207381c42284&scene=21#wechat_redirect)

  

[低学历 传奇：29 岁 6 年专套本，受够了外包，狠卷 3 个月逆袭大厂 涨 1 倍， 逆天改命](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486945&idx=1&sn=f6ff6231ccf3585624161c2ef1f50fdf&scene=21#wechat_redirect)

  

[极速上岸： 被裁 后， 8 天 拿下 京东，狠涨 一倍 年薪 48W， 小伙伴 就是 做对了一件事](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486913&idx=1&sn=edd6774e9d17c39327e8dcb95fdd68d9&scene=21#wechat_redirect)

  

[外包 + 二本 进 美团： 26 岁小 2 本 一步登天， 进了顶奢大厂（ 美团） ， 太爽了](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486904&idx=1&sn=e7c878f99ed101ebfbd1d80ec0e07401&scene=21#wechat_redirect)

  

[超牛的 Java+Al 双栖架构： 34 岁无路可走，一个月翻盘，拿 3 个架构 offer，靠  Java+Al   逆天改命！！！](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486848&idx=1&sn=ff43d058271b0801f84aba3f3d957855&scene=21#wechat_redirect)

java+AI 逆袭 2：[：3 年 程序媛 被裁， 25W-》40W 上岸， 逆涨 60%。 Java+AI 太神了， 架构小白 2 个月逆天改命](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486858&idx=1&sn=9cd122c18d166433b43d0952d3a7a7a8&scene=21#wechat_redirect)

  

  

[Java+AI 逆袭 3 ： 36 岁 / 失业 7 个月 / 彻底绝望 。狠卷 3 个月 Java+AI ，终于逆风翻盘，顺利 上岸](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486870&idx=1&sn=990a540a155cdb8254ebce6c75816882&scene=21#wechat_redirect)

[Java+AI 逆袭 ： 闲了一年，41 岁 / 失业 12 个月 / 彻底绝望 。狠卷 2 个月 Java+AI ，终于逆风翻盘](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486880&idx=1&sn=a37033157c766ac5ae7c9195fe776693&scene=21#wechat_redirect)

[Java+AI 逆袭 5：1 个月大涨 2.5W，37 岁 脱坑外包， 入了正编，GO+AI 要逆天了](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486885&idx=1&sn=4e26fbb093f45d437dedf14ea9b9e6c5&scene=21#wechat_redirect)

**职业救助站**

实现职业转型，极速上岸

![](https://mmbiz.qpic.cn/mmbiz_jpg/9qY2Gt3PBs3icE6WcM4nBOdRRcczOW4wkNDgCMZhBfqeet9suwGHfgQduP33YYqqZLOppecMI72Pv5ibNkPeIk6g/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1#imgIndex=1)

关注**职业救助站**公众号，获取每天职业干货  
助您实现**职业转型、职业升级、极速上岸**  
---------------------------------

**技术自由圈**

实现架构转型，再无中年危机

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/xlgvgPaib7WMtyFcJx45oIeWL1d6AENcdoAvuTPuq46E9n8pPe4SHXoSicuEVvhibHRSIzmEBPlC7POcDaqD6ID7w/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1#imgIndex=2)

关注**技术自由圈**公众号，获取每天技术千货  
一起成为牛逼的**未来超级架构师**

**几十篇架构笔记、5000 页面试宝典、20 个技术圣经  
请加尼恩个人微信 免费拿走**

**暗号，请在 公众号后台 发送消息：领电子书**

如有收获，请点击底部的 " 在看 "和" 赞 "，谢谢