---
source: https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247505798&idx=1&sn=e641ea3d294abc3f5cd8e2800ebff2a1&scene=21&poc_token=HLVuxWijYQtcKnXTX5IJTIEZ6Sdn3SgsV0Z75czK
create: 2025-09-13 21:16
read: true
knowledge: true
knowledge-date: 2025-09-14
tags:
  - 数据库
  - 系统架构
summary: "[[ClickHouse 介绍]]"
---
尼恩架构团队@技术自由圈 2025年09月01日 21:14

FSAC未来超级架构师

架构师总动员  
实现架构转型，再无中年危机

  

## 尼恩说在前面

在45岁老架构师 尼恩的**读者交流群**(50+)中，通过 **Java+AI双驱架构** 帮助很多小伙伴拿到了一线企业如 字节、得物、阿里、滴滴、极兔、有赞、希音、百度、网易、美团、蚂蚁、得物的面试资格，遇到很多很重要的面试题：

> 说说 ClickHouse   Group By 执行流程  ？
> 
> ClickHouse   支持 **高吞吐写入**与**低延迟查询**，可支撑亿级乃至十亿级数据的实时分析，原因是什么？
> 
> 为啥mysql 不行？

最近有小伙伴在 面试希音，遇到 到了这个的面试题。 

小伙伴没有 回答好，到手到 大厂 offer就 这样溜走了。  非常可惜。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP2NQkTh3ibu7ZKibTiceKHchEUMIn0IydBBoyQtVD7ALowIibVhIXtr6yfiaB6oZiaCorzJHNLlicGmMVOw/640?from=appmsg&watermark=1#imgIndex=0)

小伙伴面试完了之后，来求助尼恩。 那么，遇到 这个问题 该如何， 才能回答得很漂亮，才能 让面试官刮目相看、口水直流。

所以，尼恩给大家做一下系统化、体系化的梳理，使得大家内力猛增，可以充分展示一下大家雄厚的 “技术肌肉”，**让面试官爱到 “不能自已、口水直流”**，然后实现”offer直提”。

当然，这道面试题，以及参考答案，也会收入咱们的 《[尼恩Java面试宝典](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247497474&idx=1&sn=54a7b194a72162e9f13695443eabe186&scene=21#wechat_redirect)》V175版本PDF集群，供后面的小伙伴参考，提升大家的 3高 架构、设计、开发水平。

> 最新《尼恩 架构笔记》《尼恩高并发三部曲》《尼恩Java面试宝典》的PDF，请关注本公众号【技术自由圈】获取，后台回复：领电子书

# 一、ClickHouse 高性能分析型数据库核心解析

## 1、ClickHouse 概述

ClickHouse 核心优势是**高吞吐写入**与**低延迟查询**，可支撑亿级乃至十亿级数据的实时分析需求。

ClickHouse 与其他数据工具的定位差异，可通过以下图 区分：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP2NQkTh3ibu7ZKibTiceKHchEjVc1FiaicFNIvhTfRtCgiacKptOEtWKZhSA2Wn4TLUSsSa4lbgtZagPVQ/640?from=appmsg&watermark=1#imgIndex=1)

## 2、ClickHouse 整体架构

ClickHouse 的架构设计是其性能的核心基础，核心差异在于“存储与计算的关系”和“MPP 并行能力”。

ClickHouse 是面向联机分析处理（OLAP）场景的分布式列式数据库，其核心架构包含两大模块：

*   **存储层**：负责数据持久化与压缩优化
    
*   **计算层**：处理查询执行与分布式计算
    

ClickHouse  采用MPP（大规模并行处理）架构，各节点对等协作，通过数据分片与副本机制实现水平扩展。

MPP 与传统计算引擎不同，ClickHouse采用存储计算一体化设计，使存储层能针对查询需求进行深度优化。

### 2.1 架构核心：存储服务于计算

传统大数据引擎（如 Spark）是 “计算 存储 分离 ”模式——计算层需从外部存储（HDFS/S3）读取原始数据，再进行计算；

而 ClickHouse 是 “计算 存储 一体化  ”模式 —— 自带存储层，可在存储阶段为计算做优化（如列存储、预排序），即“存储服务于计算”，大幅减少计算时的 I/O 开销。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP2NQkTh3ibu7ZKibTiceKHchEAMy63HwiahUd41ZjlW5D89ibXHOv2MH8eTbKmWzGufl4SJ9M82MqocGg/640?from=appmsg&watermark=1#imgIndex=2)

### 2.2 MPP 架构：并行计算提效

ClickHouse 采用**大规模并行处理（MPP）** 架构，集群中所有节点对等，无主从之分。

MPP 架构中，查询 任务会被拆分为多个子任务，并行分发到不同节点执行，最后汇总结果，充分利用多核 CPU 与多节点资源。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP2NQkTh3ibu7ZKibTiceKHchEAk7WRibhtnr38w2dkKiaFEXbdN9m6adB5xBy7hpiccwIm2QKGnjf89wCg/640?from=appmsg&watermark=1#imgIndex=3)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP2NQkTh3ibu7ZKibTiceKHchEmOt4ict8QJmaic0tqx1TEXEddgnLFAib5su1Gf4Jcel2Rhvt12fpNy3Fw/640?from=appmsg&watermark=1#imgIndex=4)

## 3、 ClickHouse 核心性能优化特性

ClickHouse 的极速查询能力，源于三大核心优化：

*   列式存储
    
*   向量化执行
    
*   数据预排序。
    

### 3.1 列式存储：减少无效数据读取

传统行存储按“行”存储数据（一行所有列存在一个文件），查询时需读取整行数据；

而列式存储按“列”拆分存储（每列单独一个文件），查询时仅读取所需列，同时同列数据特征相似，压缩率更高（生产中常达 8:1）。

  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP2NQkTh3ibu7ZKibTiceKHchEMrUDCpU4XJicER5s5U9ibTkPbsnWmISwciaG6eRjAlblNXjVeibr0uo9jQ/640?from=appmsg&watermark=1#imgIndex=5)

### 3.2 向量化执行：提升 CPU 效率

传统计算引擎“单条数据处理”（一次处理 1 条数据，CPU 缓存命中率低）；

ClickHouse 采用“批量处理”，将数据分成固定大小的块（如 1024 条），借助 CPU 的 **SIMD（单指令多数据）** 指令集，1 条指令处理多个数据，大幅提升 CPU 利用率。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP2NQkTh3ibu7ZKibTiceKHchEnicK3kgtycCESYCiaQZzJEV7VXicUViaob5icwc90EeGVosqdFia8o3Giaf5A/640?from=appmsg&watermark=1#imgIndex=6)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP2NQkTh3ibu7ZKibTiceKHchExDrKz7ylyd4IWzIH4dWOXIibP2hzJxib65kzuFLibDssHxOkmrQRog01A/640?from=appmsg&watermark=1#imgIndex=7)

### 3.3 数据预排序：加速范围查询

ClickHouse 基于类 LSM 算法，数据写入时先在内存排序，刷盘后通过后台 Compaction 合并为有序大分区，确保磁盘数据始终有序。

查询时可利用有序性快速定位范围（如“时间>2024-01-01”），减少扫描的数据量。

MergeTree （本地表） 建表语句的  示例

```
`  
-- 示例：创建 MergeTree 表（核心表引擎，支持预排序）  
CREATE TABLE IF NOT EXISTS user_behavior (  
 user_id UInt64,          -- 用户ID（无符号整数）  
 behavior_type String,    -- 行为类型（点击/购买）  
 create_time DateTime,    -- 行为时间  
 product_id UInt64        -- 商品ID  
) ENGINE = MergeTree()      -- MergeTree：支持预排序、Compaction  
ORDER BY (user_id, create_time)  -- 排序键：决定数据存储顺序（核心配置）  
PARTITION BY toYYYYMM(create_time)  -- 按年月分区（优化时间范围查询）  
PRIMARY KEY user_id;        -- 主键：需为排序键前缀（此处user_id是排序键首字段）  
-- 关键解释：  
-- 1. ORDER BY 是预排序的核心，查询常按 user_id+时间范围过滤，有序数据可快速定位  
-- 2. PARTITION BY 按时间分区，查询时先定位分区，再扫描分区内数据  
`
```

## 4、 ClickHouse 表引擎

表引擎决定了数据的存储方式、查询支持能力、并发控制等

### 4.1 ClickHouse 提供多种引擎

ClickHouse 提供多种引擎，适配不同场景，常用引擎如下：

<table style="line-height: 1.75em;padding: 6px 13px;border-color: rgb(198, 203, 209) rgb(223, 226, 229) rgb(223, 226, 229);background-color: rgb(246, 248, 250);border-top-width: 1px;border-top-style: solid;"><thead><tr><td style="line-height: 1.75em;padding: 6px 13px;border-color: rgb(198, 203, 209) rgb(223, 226, 229) rgb(223, 226, 229);background-color: rgb(246, 248, 250);border-top-width: 1px;border-top-style: solid;"><span style="font-size: 15px;"><strong><span leaf="">表引擎类型</span></strong></span></td><td style="line-height: 1.75em;padding: 6px 13px;border-color: rgb(198, 203, 209) rgb(223, 226, 229) rgb(223, 226, 229);background-color: rgb(246, 248, 250);border-top-width: 1px;border-top-style: solid;"><span style="font-size: 15px;"><strong><span leaf="">代表引擎</span></strong></span></td><td style="line-height: 1.75em;padding: 6px 13px;border-color: rgb(198, 203, 209) rgb(223, 226, 229) rgb(223, 226, 229);background-color: rgb(246, 248, 250);border-top-width: 1px;border-top-style: solid;"><span style="font-size: 15px;"><strong><span leaf="">核心用途</span></strong></span></td><td style="line-height: 1.75em;padding: 6px 13px;border-color: rgb(198, 203, 209) rgb(223, 226, 229) rgb(223, 226, 229);background-color: rgb(246, 248, 250);border-top-width: 1px;border-top-style: solid;"><span style="font-size: 15px;"><strong><span leaf="">适用场景</span></strong></span></td></tr></thead><tbody><tr style="background-color: rgb(246, 248, 250);border-top-width: 1px;border-top-style: solid;border-top-color: rgb(198, 203, 209);"><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">核心存储引擎</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">MergeTree</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">海量结构化数据存储（支持预排序、分区）</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">业务核心表（如用户行为、订单表）</span></span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top-width: 1px;border-top-style: solid;border-top-color: rgb(198, 203, 209);"><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">分布式引擎</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">Distributed</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">管理分片集群（不存储数据，仅路由请求）</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">分布式查询入口（对外提供统一表名）</span></span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top-width: 1px;border-top-style: solid;border-top-color: rgb(198, 203, 209);"><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">内存引擎</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">Memory</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">内存存储（数据重启丢失）</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">临时计算、小表缓存</span></span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top-width: 1px;border-top-style: solid;border-top-color: rgb(198, 203, 209);"><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">日志引擎</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">Log</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">轻量存储（无索引）</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">日志数据（如审计日志、访问日志）</span></span></section></td></tr></tbody></table>

Distributed   关键解释：

*   Distributed 表本身不存数据，仅作为“路由入口”
    
*   写入时：按分片键计算分片ID，路由到对应节点的本地表
    
*   查询时：向所有分片发送子查询，汇总结果后返回
    

Distributed 表建表示例（分片管理）

```
`  
-- 前提：已在 config.xml 配置集群 my_cluster（2个分片，每个分片1个副本）  
CREATE TABLE IF NOT EXISTS user_behavior_dist (  
 user_id UInt64,  
 behavior_type String,  
 create_time DateTime,  
 product_id UInt64  
) ENGINE = Distributed(  
 my_cluster,          -- 集群名称（配置文件中定义）  
 default,             -- 数据库名（与本地表一致）  
 user_behavior,       -- 本地表名（每个分片节点上的表）  
 user_id              -- 分片键（按user_id哈希分片，保证数据均匀）  
);  
  
`
```

### 4.2  Distributed 与 MergeTree 引擎在 ClickHouse 中的协作关系

在 ClickHouse 的高性能列式数据库架构中，Distributed 引擎和 MergeTree 系列引擎有着密切的协同关系，共同构建了 ClickHouse 的分布式处理能力。

#### Distributed 与 MergeTree 引擎 对比

<table style="line-height: 1.75em;padding: 6px 13px;border-color: rgb(198, 203, 209) rgb(223, 226, 229) rgb(223, 226, 229);background-color: rgb(246, 248, 250);border-top-width: 1px;border-top-style: solid;"><thead><tr><td style="line-height: 1.75em;padding: 6px 13px;border-color: rgb(198, 203, 209) rgb(223, 226, 229) rgb(223, 226, 229);background-color: rgb(246, 248, 250);border-top-width: 1px;border-top-style: solid;"><span style="font-size: 15px;"><strong><strong><span leaf="">特性</span></strong></strong></span></td><td style="line-height: 1.75em;padding: 6px 13px;border-color: rgb(198, 203, 209) rgb(223, 226, 229) rgb(223, 226, 229);background-color: rgb(246, 248, 250);border-top-width: 1px;border-top-style: solid;"><span style="font-size: 15px;"><strong><strong><span leaf="">Distributed 引擎</span></strong></strong></span></td><td style="line-height: 1.75em;padding: 6px 13px;border-color: rgb(198, 203, 209) rgb(223, 226, 229) rgb(223, 226, 229);background-color: rgb(246, 248, 250);border-top-width: 1px;border-top-style: solid;"><span style="font-size: 15px;"><strong><strong><span leaf="">MergeTree 引擎</span></strong></strong></span></td></tr></thead><tbody><tr style="background-color: rgb(246, 248, 250);border-top-width: 1px;border-top-style: solid;border-top-color: rgb(198, 203, 209);"><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><strong><span leaf="">本质</span></strong></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">逻辑引擎（无实际存储）</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">物理存储引擎</span></span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top-width: 1px;border-top-style: solid;border-top-color: rgb(198, 203, 209);"><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><strong><span leaf="">作用</span></strong></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">分布式查询与写入协调</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">本地数据存储与处理</span></span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top-width: 1px;border-top-style: solid;border-top-color: rgb(198, 203, 209);"><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><strong><span leaf="">存储</span></strong></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">不存储实际数据</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">管理本地磁盘数据</span></span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top-width: 1px;border-top-style: solid;border-top-color: rgb(198, 203, 209);"><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><strong><span leaf="">数据分布</span></strong></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">跨节点/分片</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">分区内有序存储</span></span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top-width: 1px;border-top-style: solid;border-top-color: rgb(198, 203, 209);"><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><strong><span leaf="">依赖关系</span></strong></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">依赖底层的 MergeTree 表</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">作为 Distributed 表基础</span></span></section></td></tr></tbody></table>

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP2NQkTh3ibu7ZKibTiceKHchEyyYu96IQbianhqQqRHiaPJia6ib92QaT6naQarZVzCfUSfgBQ3WMaIfIoQ/640?from=appmsg&watermark=1#imgIndex=8)

### 4.3  Distributed 与 MergeTree 引擎协作机制说明

Distributed 和 MergeTree 引擎在 ClickHouse 中各司其职：

*   •**MergeTree 引擎**提供高性能本地存储和处理， 作为物理存储基础，每个分片上有独立的MergeTree实例
    
*   •**Distributed 引擎**实现透明分布式访问层， 提供 分布式访问入口
    
*   •**协作关系**：Distributed 表是逻辑入口，MergeTree 表是物理基础
    

### 4.3.1. 数据写入流程

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP2NQkTh3ibu7ZKibTiceKHchEGy6cJG1sUsUKOqbLHjFZfuoO6BXSGVYQOTcb3eaE8fjMJ4WhJ6EJibw/640?from=appmsg&watermark=1#imgIndex=9)

### 4.3.2. 查询处理流程

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP2NQkTh3ibu7ZKibTiceKHchEvBCicd2GiaiaicF49RT4XGu2emicLia20lUZ9MLjMNeVeakDRJtib7B4jEbYQ/640?from=appmsg&watermark=1#imgIndex=10)

### 4.3.3. 实际应用场景

```
`  
-- 创建本地存储表  
CREATE TABLE events_local ON CLUSTER main_cluster  
(  
 event_date Date,  
 event_type String,  
 user_id UInt64,  
 value Float64  
)  
ENGINE = ReplicatedMergeTree('/clickhouse/tables/{shard}/events_local', '{replica}')  
PARTITION BY toYYYYMM(event_date)  
ORDER BY (event_date, event_type);  
-- 创建分布式访问表  
CREATE TABLE events_distributed AS events_local  
ENGINE = Distributed('main_cluster', 'default', 'events_local', rand());  
`
```

## 5、 ClickHouse 数据类型

ClickHouse 支持 100+ 数据类型，设计上兼顾“存储效率”与“计算性能”，核心类型分类及实践如下：

核心数据类型表

<table style="line-height: 1.75em;padding: 6px 13px;border-color: rgb(198, 203, 209) rgb(223, 226, 229) rgb(223, 226, 229);background-color: rgb(246, 248, 250);border-top-width: 1px;border-top-style: solid;"><thead><tr><td style="line-height: 1.75em;padding: 6px 13px;border-color: rgb(198, 203, 209) rgb(223, 226, 229) rgb(223, 226, 229);background-color: rgb(246, 248, 250);border-top-width: 1px;border-top-style: solid;"><span style="font-size: 15px;"><strong><span leaf="">类型分类</span></strong></span></td><td style="line-height: 1.75em;padding: 6px 13px;border-color: rgb(198, 203, 209) rgb(223, 226, 229) rgb(223, 226, 229);background-color: rgb(246, 248, 250);border-top-width: 1px;border-top-style: solid;"><span style="font-size: 15px;"><strong><span leaf="">具体类型</span></strong></span></td><td style="line-height: 1.75em;padding: 6px 13px;border-color: rgb(198, 203, 209) rgb(223, 226, 229) rgb(223, 226, 229);background-color: rgb(246, 248, 250);border-top-width: 1px;border-top-style: solid;"><span style="font-size: 15px;"><strong><span leaf="">用途说明</span></strong></span></td><td style="line-height: 1.75em;padding: 6px 13px;border-color: rgb(198, 203, 209) rgb(223, 226, 229) rgb(223, 226, 229);background-color: rgb(246, 248, 250);border-top-width: 1px;border-top-style: solid;"><span style="font-size: 15px;"><strong><span leaf="">示例</span></strong></span></td></tr></thead><tbody><tr style="background-color: rgb(246, 248, 250);border-top-width: 1px;border-top-style: solid;border-top-color: rgb(198, 203, 209);"><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">基本数值类型</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">UInt8/UInt64</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">无符号整数（存储ID、计数，节省空间）</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">user_id UInt64</span></span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top-width: 1px;border-top-style: solid;border-top-color: rgb(198, 203, 209);"><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">Int8/Int64</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">有符号整数（存储带正负的数值）</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">score Int32</span></span></section></td><td><section><span leaf=""><br></span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top-width: 1px;border-top-style: solid;border-top-color: rgb(198, 203, 209);"><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">Decimal(P,S)</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">高精度小数（金额、汇率，避免浮点误差）</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">amount Decimal(18,2)</span></span></section></td><td><section><span leaf=""><br></span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top-width: 1px;border-top-style: solid;border-top-color: rgb(198, 203, 209);"><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">日期时间类型</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">Date</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">日期（YYYY-MM-DD，仅占2字节）</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">create_date Date</span></span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top-width: 1px;border-top-style: solid;border-top-color: rgb(198, 203, 209);"><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">DateTime64(N)</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">带精度的时间（N为小数位数，如毫秒）</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">pay_time DateTime64(3)</span></span></section></td><td><section><span leaf=""><br></span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top-width: 1px;border-top-style: solid;border-top-color: rgb(198, 203, 209);"><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">复杂类型</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">Array(T)</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">同类型数组（存储多值，如商品分类）</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">tags Array(String)</span></span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top-width: 1px;border-top-style: solid;border-top-color: rgb(198, 203, 209);"><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">Nested(k1 T1, k2 T2)</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">嵌套结构（类似JSON数组对象）</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">addr Nested(street String, city String)</span></span></section></td><td><section><span leaf=""><br></span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top-width: 1px;border-top-style: solid;border-top-color: rgb(198, 203, 209);"><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">优化类型</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">LowCardinality(T)</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">低基数优化（值数量&lt;1万，如状态）</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">status LowCardinality(String)</span></span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top-width: 1px;border-top-style: solid;border-top-color: rgb(198, 203, 209);"><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">Nullable(T)</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">支持空值（需额外存储空标记，慎用）</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">email Nullable(String)</span></span></section></td><td><section><span leaf=""><br></span></section></td></tr></tbody></table>

复杂类型使用示例

```
`  
-- 示例：含复杂类型的商品表  
CREATE TABLE IF NOT EXISTS product_info (  
 product_id UInt64,  
 name String,  
 category Array(String),  -- 数组：商品所属多个分类  
 price Decimal(18,2),  
 status LowCardinality(String),  -- 低基数：状态仅“在售/下架/预售”  
 update_time DateTime64(3),  
 remark Nullable(String)  -- 可空：备注可能为空  
) ENGINE = MergeTree()  
ORDER BY product_id;  
-- 插入数据（数组用[]，空值用NULL）  
INSERT INTO product_info VALUES (  
 1001,  
 "5G手机",  
 ["电子设备", "手机", "5G"],  -- 数组值  
 3999.00,  
 "在售",  -- 低基数值  
 now64(3),  -- 当前毫秒时间  
 NULL       -- 空值  
);  
-- 关键解释：  
-- 1. LowCardinality：将“在售”编码为0、“下架”为1，存储时用编码值，减少空间  
-- 2. Array：查询时用 HAS 关键字（如 category HAS '5G'）快速过滤  
`
```

## 6、 分片与副本策略

在 ClickHouse 中，为了提升查询性能及增加数据容错性，分别在水平方向和垂直方向上划分为分片（shard）及副本（replica）。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP2NQkTh3ibu7ZKibTiceKHchEHrW8Kr9Fhu9v6PUictCV1IoTkZmzhsz5sG1pUy75MJRjF6olZG9d8IA/640?from=appmsg&watermark=1#imgIndex=11)

在分布式模式下，ClickHouse 会将数据在水平方向上分为多个分片（shard），并且分布到不同节点上，不同分片间的数据不同。

分片（shard） 的目的主要是为了提升查询性能，方便多线程及分布式查询。

在分布式查询时，按照分片数量拆分成若干个对本地表的子查询，然后依次查询每个分片的数据，再合并汇总返回。

### 6.1 分片：水平拆分数据

将数据按“分片键”拆分为多个部分（分片），分布到不同节点，查询时并行处理各分片数据，提升效率。分片键选择需满足“数据均匀分布”（如 user_id、order_id）。

**分片策略：**

*   **哈希分片**：按分片键哈希值分配（如 `user_id % 分片数`，最常用）；
    
*   **随机分片**：按随机数分配（数据均匀，但同一用户数据可能跨分片）；
    
*   **范围分片**：按分片键范围分配（如时间范围，适合按时间查询的场景）。
    

### 6.2 副本：垂直冗余数据

同一分片的数据在多个节点存储副本，借助 ZooKeeper 实现副本同步（写入一个副本后，自动同步到其他副本），避免单点故障。

副本选择策略（负载均衡）

查询时，ClickHouse 按 `load_balancing` 参数选择副本，常用策略：- **Random（默认）**：优先选择错误数最少的副本，再随机；- **Round Robin**：循环选择副本，避免单点压力；- **Nearest hostname**：选择主机名最相似的副本（减少网络延迟）。

### 6.3 分片与副本整体流程

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP2NQkTh3ibu7ZKibTiceKHchEk5OKJlXdbcXFQBJiaH9QIkRwIicMERgpQVcgWAftcKhs3O61YwzA3EEg/640?from=appmsg&watermark=1#imgIndex=12)

## 7、ClickHouse 索引设计

索引是加速查询的关键，ClickHouse 主要支持**稀疏索引（主键索引）** 和**跳数索引**，避免传统稠密索引的高存储开销。

### 7.1 稀疏索引（主键索引）

ClickHouse 将数据按“颗粒（默认 8192 行）”划分，仅为每个颗粒的首行存储主键值（存于 `primary.idx`），并通过标记文件（`.mrk`）记录颗粒的磁盘物理地址。查询时先通过索引定位颗粒，再读取颗粒数据，大幅减少扫描范围。

  

**查询流程：**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP2NQkTh3ibu7ZKibTiceKHchE6xSYReHK6xRnUsxL2ao11Qa2ITO2l7dRc0LtDcEJbpYv2tHtjCDWUQ/640?from=appmsg&watermark=1#imgIndex=13)

### 7.2 跳数索引（辅助索引）

当查询条件不命中主键时，跳数索引可进一步跳过无匹配数据的块，常用类型：- **minmax 索引**：存储每个块的字段最值（适合范围查询，如时间范围）；- **布隆过滤器索引**：快速判断块是否包含目标值（适合等值查询，如 `category='5G'`）；- **set 索引**：存储块内所有 distinct 值（适合枚举值查询）。

**跳数索引创建示例：**

```
`  
-- 示例1：为时间字段创建minmax索引（优化范围查询）  
ALTER TABLE user_behavior   
ADD INDEX idx_create_time create_time TYPE minmax GRANULARITY 8192;  
-- GRANULARITY 8192：每8192行生成一个索引条目  
-- 示例2：为数组字段创建布隆过滤器索引（优化等值查询）  
ALTER TABLE product_info   
ADD INDEX idx_category category TYPE bloom_filter(0.01) GRANULARITY 4096;  
-- bloom_filter(0.01)：误判率1%，平衡空间与精度  
-- 关键解释：  
-- 1. 跳数索引需手动创建，默认不开启  
-- 2. 选择合适的GRANULARITY：值越小，索引越细，但存储开销越大  
`
```

### 7.2 比喻说明

想象一下，ClickHouse 的表就是一本**超厚的百科全书**（比如《中国大百科全书》）。

1.**主键索引（Primary Index）是啥？**

*   它就是书最前面的 **总目录**。这个目录很稀疏，不会告诉你“苹果”在第几页，只会告诉你“A字母开头的条目”从第1页开始，“B字母开头的条目”从第50页开始……
    
*   在 ClickHouse 里，这就是**稀疏索引**。它通过主键帮你快速定位到数据块（Granule），`mark`文件就像精确的页码，帮你翻到那一章。
    

2.**那“跳数索引”又是干嘛的？**

*   现在，你想在这本百科全书里找所有 **“产地是新疆的水果”**。
    
*   用总目录（主键索引）你只能找到“水果”这个大类在哪（比如在“S”字母 section），但你还是得把“水果”这一整章几十页全部读完，才能找出哪些水果的产地是新疆。这很慢。
    
*   **跳数索引**，就像是有人在书页边上贴了各种各样的**便利贴**，或者给书做了很多**专项小目录**。
    
*   比如，有人做了一份 **“产地索引”** 的小册子，上面写着：•`产地：新疆` -> **出现在第15页、第28页、第33页**•`产地：海南` -> **出现在第22页、第40页**
    
*   有了这个“产地索引”（也就是跳数索引），你就不用去读“水果”章节的所有几十页了。你直接翻到第15、28、33页看一眼，就能找到所有新疆水果。**你跳过了大量根本不相关的书页！**
    

## 8、ClickHouse 计算引擎

ClickHouse 计算引擎的核心是“将 SQL 转化为可执行计划, 然后 并行执行”

从功能和整体架构上讲，ClickHouse 的计算引擎与  其他数据库（如mysql  ）的计算引擎并没有很大的不同。

功能都是将可描述的结构化查询语言（SQL）转化翻译为可以执行的物理计划以及执行计算并获得计算结果，而整体架构都有包含 SQL 解析、翻译解释、计算执行等部分。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP2NQkTh3ibu7ZKibTiceKHchEmJJYflzW5SkWUQzxXkkvksqVZzjibB74AvR8XVskvLbWtdxeTBrCGzA/640?from=appmsg&watermark=1#imgIndex=14)

计算引擎核心流程

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP2NQkTh3ibu7ZKibTiceKHchEj5KdhibwHAzmWQI1UYfsPyY5zm1zy5XmJQ90PVWHa2IB9OhSk7rvWzw/640?from=appmsg&watermark=1#imgIndex=15)

从实现方面，ClickHouse 同样采用了多线程及分布式查询，成为其高性能和高扩展性的关键。

通过线程级并行的方式提升性能，利用多核 CPU 的计算能力；

通过采用分布式架构，支持分片和副本，通过将数据分布到多个节点实现存储和计算的扩展。

# 二、ClickHouse聚合算子实现原理

在ClickHouse中，聚合是最核心的算子之一，也是优化最多的部分。

聚合 底层实现融合了算法设计、数据结构优化和硬件特性利用，最终实现了对海量数据的高效聚合。

在 ClickHouse 中，**聚合操作所使用的 Hashmap 中的 Key，本质上， 就是由 `GROUP BY` 子句计算出的值（分组键）**。

ClickHouse的聚合实现是“算法设计+硬件特性+场景适配”的结合体，每个优化都针对具体痛点。

理解这些细节，不仅能更好地使用ClickHouse，也能为其他大数据引擎的优化提供思路。

## 2.1、聚合算法核心架构：从两阶段到并行化

ClickHouse的聚合逻辑核心围绕三个要素展开：HashMap（存储中间结果）、并行计算（提升效率）、两阶段聚合（减少数据传输）。

理解这三个要素，是掌握聚合实现的基础。

### 2.1.1 聚合流程 的 两个 基础 阶段

常规聚合过程分为**pre-aggregate（初步聚合）** 和**final-aggregate（最终聚合）** 两个阶段：

*   **pre-aggregate**：多个线程（T1、T2、T3）各自处理部分数据，用HashMap存储本地中间结果（如每个线程计算自己负责的数据的sum/count）；
    
*   **final-aggregate**：单线程合并所有线程的中间结果，得到最终聚合结果。
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP2NQkTh3ibu7ZKibTiceKHchE3EgE32q8VTBLiapOMr1dXs0j4v52oacYZVc6EYhDaFK7AZZO62dHwuA/640?from=appmsg&watermark=1#imgIndex=16)

**存在的问题**：

如果pre-aggregate后数据量几乎没减少（比如聚合key基数极高），final-aggregate会因单线程处理大量数据而成为瓶颈，严重影响效率。

### 2.1.2 并行优化：Two-level HashMap的引入

为解决final-aggregate单线程瓶颈，ClickHouse引入**Two-level HashMap（二级哈希表）**，实现final-aggregate的并行化。

#### 2.1.2.1 Two-level HashMap结构

与常规HashMap不同，Two-level HashMap分为两层：- 第一层：固定数量的Bucket（默认256个），每个Bucket是一个独立的子HashMap；- 第二层：子HashMap内部用开放地址法处理冲突。

数据先通过哈希映射到第一层Bucket，再进入子HashMap处理，天然支持按Bucket拆分并行处理。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP2NQkTh3ibu7ZKibTiceKHchERa5oGwcOoSB5siboKRTR4QJqJRJ3FacQDmppX0sc9JFgu8ljzO7HWeA/640?from=appmsg&watermark=1#imgIndex=17)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP2NQkTh3ibu7ZKibTiceKHchEpPg3ia4UD9RGiah1BJFHqFicIryMmMRerj9PDicBjKexJHJmkEcbyvCGlg/640?from=appmsg&watermark=1#imgIndex=18)

#### 2.1.2.2 并行化的final-aggregate流程

引入Two-level HashMap后，final-aggregate可按第一层Bucket拆分任务，

每一个线程可以独立的 处理一个Bucket的子HashMap，避免并发冲突，实现并行合并：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP2NQkTh3ibu7ZKibTiceKHchE1sicSy8kLSs1neoOwAiaqmA7DBLNicxCGXHibYNvjNf01rqU60H7ibx9zWA/640?from=appmsg&watermark=1#imgIndex=19)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP2NQkTh3ibu7ZKibTiceKHchECDib0f42QZBVexrrNHUicyiazJ8t1OzamS6kbHiaosAL4qkWKtmrGuQVgQ/640?from=appmsg&watermark=1#imgIndex=20)

### 2.1.3 聚合算法的自适应选择

ClickHouse提供了一种运行时自适应的方式，在运行时选择选择聚合算法以及是否使用Two-level HashMap。

具体的策略是当pre-aggregate过程中，HashMap中的数据超过一定阈值，将其转换成Two-level HashMap。

参考参数：

*   `group_by_two_level_threshold`：HashMap中条目数超过该阈值时，转换为Two-level HashMap；
    
*   `group_by_two_level_threshold_bytes`：HashMap占用内存超过该阈值时，转换为Two-level HashMap。
    

每个pre-aggregate线程独立选择是否是用Two-level HashMap的，那么在进行第二次聚合的时候输入可能既有常规HashMap也有Two-level Hashmap，对于这种情况会在final-aggregate前先将常规HashMap转换成Two-level的。整体的算法如下图所示：

转换逻辑如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP2NQkTh3ibu7ZKibTiceKHchExw0dicxQP4jYzTf7diaYQGnicsmFeTTErpqibAMv91tzy6TyBZxouwso0Q/640?from=appmsg&watermark=1#imgIndex=21)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP2NQkTh3ibu7ZKibTiceKHchEmQIkEicJQoFsrH2xjCibO0tadCZee3wVJKbPq01ictvn0WwSZmfZAe6KA/640?from=appmsg&watermark=1#imgIndex=22)

**潜在优化点**：目前HashMap到Two-level的转换是单线程执行，若改为并行转换可进一步提升效率。

## 2.2、HashMap的构建与聚合执行

ClickHouse的计算引擎基于列式存储，数据以Block（由多列组成的二维表）为单位流转。

聚合过程本质是将Block数据写入HashMap并执行聚合计算，分为“构建HashMap”和“执行聚合”两步。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP2NQkTh3ibu7ZKibTiceKHchEzLcOa4lHFicwDc3CmvNYkDPSwfnU9DfUHcsWXfCqYvtsUB6doYn6RVQ/640?from=appmsg&watermark=1#imgIndex=23)

### 2.2.1 构建HashMap：初始化聚合状态

构建阶段 de 目标 ：为所有聚合key分配内存，并初始化聚合函数的状态对象（如sum的初始值0、uniqExact的空HashSet）。

**关键流程：**

**(1) 遍历Block中的每行数据，判断聚合key是否已在HashMap中；**

**(2) 若不存在，通过 Arena 分配聚合函数状态所需内存, `Arena` 就是内存池 ；**

**(3) 初始化状态对象（如sum函数初始化为0）；**

**(4) 记录每行对应的状态对象内存地址，便于后续快速访问。**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP2NQkTh3ibu7ZKibTiceKHchEW0o45QnA8I7tL2T5UynibBoze2sniccFQfHDg5q1uc2drFey7MDtlnKQ/640?from=appmsg&watermark=1#imgIndex=24)

关键代码解析（内存分配与状态初始化）：

```
`  
/// 1. 为所有聚合函数状态分配内存  
/// total_size_of_aggregate_states：所有聚合函数状态的总内存大小  
/// aggregates_pool：Arena内存池（统一管理内存，便于监控和释放）  
/// place：分配到的连续内存块起始地址  
AggregateDataPtr place = result.aggregates_pool->alignedAlloc(  
 total_size_of_aggregate_states,  // 总大小  
 align_aggregate_states            // 内存对齐要求  
);  
/// 2. 初始化每个聚合函数的状态对象  
/// offsets_of_aggregate_states：每个聚合函数在总内存中的偏移量  
for (size_t j = 0; j < aggregate_functions.size(); ++j)  
{  
 // 为第j个聚合函数创建状态对象（如sum创建AggregateFunctionSumData<Int64>）  
 aggregate_functions[j]->create(place + offsets_of_aggregate_states[j]);  
}  
`
```

**为什么用Arena而不是new？**

聚合可能消耗大量内存，Arena可集中管理内存，便于监控总占用量、限制单查询内存使用，或在内存不足时将状态 spill 到磁盘。

> Arena 是什么？
> 
> Netty 源码中也用到了 Arena ，具体请参考 尼恩的 Netty对象池、内存池视频。
> 
> 简单说，**Arena 是一个预分配的大块连续内存（内存池）**。当聚合需要为新的分组键分配内存（存储键值或聚合状态）时，不再频繁调用系统的 `new` 或 `malloc`，而是从 Arena 中“划出”一小段连续空间使用。

### 2.2.2 执行聚合：更新状态对象

构建完HashMap后，需根据每行数据更新对应的聚合函数状态（如sum累加、count递增）。

由于状态对象分散在不同内存地址，无法通过SIMD指令批量处理，但可针对特殊场景优化。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP2NQkTh3ibu7ZKibTiceKHchEwpouON1icicNdCw5RibmA4hN2AhrJg20apQEoyuIobfFdtfiblT51TSgjA/640?from=appmsg&watermark=1#imgIndex=25)

**常规聚合执行代码：**

```
`  
/// 为一个聚合函数批量处理多行数据  
void IAggregateFunctionHelper::addBatch(  
 size_t row_begin,          // 起始行  
 size_t row_end,            // 结束行  
 AggregateDataPtr * places, // 每行对应的状态对象地址  
 size_t place_offset,       // 状态对象在内存块中的偏移  
 const IColumn ** columns,  // 输入列数据  
 Arena * arena,             // 内存池（可能用于动态分配，如HashSet扩容）  
 ssize_t if_argument_pos    // 条件列位置（可选）  
) const override  
{  
 // 遍历行，更新对应状态  
 for (size_t i = row_begin; i < row_end; ++i)  
 if (places[i])  // 若状态对象存在（已初始化）  
 // 调用具体聚合函数的add方法（如sum累加）  
 static_cast<const Derived *>(this)->add(  
 places[i] + place_offset,  // 状态对象地址  
 columns,                    // 输入数据列  
 i,                          // 当前行号  
 arena  
 );  
}  
`
```

**特殊优化：常量key的SIMD加速**

若所有聚合key相同（结果只有一条），可通过SIMD指令批量处理：

```
`  
/// Sum函数针对常量key的批量处理（用SIMD加速）  
template <typename Value>  
void NO_INLINE AggregateFunctionSumData::addMany(  
 const Value * __restrict ptr,  // 输入数据指针（__restrict避免指针别名，优化编译）  
 size_t start,   
 size_t end  
)  
{  
#if USE_MULTITARGET_CODE  
 // 根据CPU支持的指令集选择最优实现  
 if (isArchSupported(TargetArch::AVX512BW))  
 {  
 addManyImplAVX512BW(ptr, start, end);  // 用AVX512BW指令  
 return;  
 }  
 else if (isArchSupported(TargetArch::AVX2))  
 {  
 addManyImplAVX2(ptr, start, end);      // 用AVX2指令  
 return;  
 }  
 // ... 其他指令集（SSE42等）  
#endif  
 // 无SIMD支持时的普通实现  
 addManyImpl(ptr, start, end);  
}  
`
```

**JIT优化**：新版本默认开启`compile_aggregate_expressions`参数，通过即时编译（JIT）生成聚合代码，减少函数调用开销，进一步提升效率。

## 三、HashMap类型选择策略

HashMap是聚合的核心数据结构，ClickHouse针对不同场景设计了30+种HashMap及哈希函数组合，核心思路是“特例化优化”——根据聚合key的类型和数量选择最优实现。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP2NQkTh3ibu7ZKibTiceKHchEzvV3o1ZKMZSPSTsPk5ZFXNo2BibKv4lP3OqYibcXDxYiavk95VqhbZibCQ/640?from=appmsg&watermark=1#imgIndex=26)

**智能数据结构选择（临界点优化）**

<table style="line-height: 1.75em;padding: 6px 13px;border-color: rgb(198, 203, 209) rgb(223, 226, 229) rgb(223, 226, 229);background-color: rgb(246, 248, 250);border-top-width: 1px;border-top-style: solid;"><thead><tr><td style="line-height: 1.75em;padding: 6px 13px;border-color: rgb(198, 203, 209) rgb(223, 226, 229) rgb(223, 226, 229);background-color: rgb(246, 248, 250);border-top-width: 1px;border-top-style: solid;"><span style="font-size: 15px;"><strong><strong><span leaf="">分组键特征</span></strong></strong></span></td><td style="line-height: 1.75em;padding: 6px 13px;border-color: rgb(198, 203, 209) rgb(223, 226, 229) rgb(223, 226, 229);background-color: rgb(246, 248, 250);border-top-width: 1px;border-top-style: solid;"><span style="font-size: 15px;"><strong><strong><span leaf="">选用数据结构</span></strong></strong></span></td><td style="line-height: 1.75em;padding: 6px 13px;border-color: rgb(198, 203, 209) rgb(223, 226, 229) rgb(223, 226, 229);background-color: rgb(246, 248, 250);border-top-width: 1px;border-top-style: solid;"><span style="font-size: 15px;"><strong><strong><span leaf="">性能临界点</span></strong></strong></span></td></tr></thead><tbody><tr style="background-color: rgb(246, 248, 250);border-top-width: 1px;border-top-style: solid;border-top-color: rgb(198, 203, 209);"><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><code><span leaf="">UInt8</span></code><span leaf="">&nbsp;小范围（&lt;256）</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><code><span leaf="">FixedHashMap</span></code><span leaf="">（数组）</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">无哈希计算 O(1)</span></span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top-width: 1px;border-top-style: solid;border-top-color: rgb(198, 203, 209);"><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">普通整型/字符串</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><code><span leaf="">HashMapWithSavedHash</span></code></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">基数 &lt; 1000万</span></span></section></td></tr><tr style="background-color: rgb(246, 248, 250);border-top-width: 1px;border-top-style: solid;border-top-color: rgb(198, 203, 209);"><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">海量基数（&gt;1000万）</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><code><span leaf="">TwoLevelHashMap</span></code></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">桶分治 + 并行</span></span></section></td></tr><tr style="background-color: rgb(255, 255, 255);border-top-width: 1px;border-top-style: solid;border-top-color: rgb(198, 203, 209);"><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">超长字符串</span></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><code><span leaf="">StringHashMap</span></code></span></section></td><td style="padding: 6px 13px;border-color: rgb(223, 226, 229);"><section style="line-height: 1.75em;"><span style="font-size: 15px;"><span leaf="">短字符串内联存储</span></span></section></td></tr></tbody></table>

## 3.1 基础结构：开放地址法HashMap

ClickHouse采用**开放地址法**实现HashMap（而非Java标准库的拉链法），原因如下：- **缓存友好**：数据存储连续，CPU缓存命中率更高；- **无内存碎片**：插入时无需动态分配链表节点，减少内存碎片；- **实现简单**：避免链表遍历，合并HashMap时效率更高。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP2NQkTh3ibu7ZKibTiceKHchEKJ0iciaPibGeKqLDQB0gXTdeJPkFm3kagLROZDVtTXlLjia63bcIp6oPibg/640?from=appmsg&watermark=1#imgIndex=27)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP2NQkTh3ibu7ZKibTiceKHchEpICHojUaic96Ukj4LPWhoGWVGORwUBYbjP33NSkNuFKYQZptDJq4SHA/640?from=appmsg&watermark=1#imgIndex=28)

## 3.2 按场景优化的HashMap类型

### 3.2.1 FixedHashMap：适用于低基数整数key

当聚合key是UInt8（0-255）或UInt16（0-65535）时，直接用key值作为索引，无需哈希计算：- 内存结构：固定大小的数组（如UInt8对应256个槽位）；- 优势：无哈希冲突、无需resize、查询O(1)。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP2NQkTh3ibu7ZKibTiceKHchEju7nKzd4GtIsRr18dNPNibHhkzzicoCnP8d22EoHxKK8kGws8tANicwibA/640?from=appmsg&watermark=1#imgIndex=29)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP2NQkTh3ibu7ZKibTiceKHchEGKxiaaGZB9rwmk4jPUWcJoOV5svpvxdr09Y3E06YbarxWeL7g2lXfNw/640?from=appmsg&watermark=1#imgIndex=30)

### 3.2.2 StringHashMap：适用于字符串key

字符串key长度可变，StringHashMap内置多个子HashMap，按字符串长度路由：- 短字符串（≤8字节）：用UInt64存储，哈希函数基于SSE42的`_mm_crc32_u64`（硬件加速）；- 中长字符串（≤24字节）：拆分后用多个UInt64处理；- 长字符串：用常规哈希函数。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP2NQkTh3ibu7ZKibTiceKHchEmAw2jaKGhoM41vex9H6yBxA2QgkKalsSose0XgsY8tM3eRGC7fSh8A/640?from=appmsg&watermark=1#imgIndex=31)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP2NQkTh3ibu7ZKibTiceKHchEU0b9UYXJuVEyEgvMo66PzkgKed94NXujjvL4Dc8uBfZnNNo5bkC4Uw/640?from=appmsg&watermark=1#imgIndex=32)

**哈希函数示例（SSE42加速）**：

```
`  
struct StringHashTableHash  
{  
#if defined(**SSE4_2**)  
 // 处理8字节字符串key（StringKey8本质是UInt64）  
 size_t ALWAYS_INLINE operator()(StringKey8 key) const  
 {  
 size_t res = -1ULL;  // 初始值  
 res = _mm_crc32_u64(res, key);  // 用硬件CRC32指令计算哈希  
 return res;  
 }  
 // 处理16字节字符串key（拆分为两个UInt64）  
 size_t ALWAYS_INLINE operator()(StringKey16 key) const  
 {  
 size_t res = -1ULL;  
 res = _mm_crc32_u64(res, key.items[0]);  // 第一部分  
 res = _mm_crc32_u64(res, key.items[1]);  // 第二部分  
 return res;  
 }  
 // ... 其他长度处理  
#endif  
};  
`
```

### 3.2.3 TwoLevelHashMap：支持并行聚合

如前文所述，TwoLevelHashMap含256个固定子HashMap，通过哈希值前8位选择子HashMap，剩余位用于子HashMap内部定位。

其变种`TwoLevelHashMapWithSavedHash`会缓存哈希值，合并时无需重复计算，进一步提速。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP2NQkTh3ibu7ZKibTiceKHchENdzwTs438WnCXofq93l6jf5nvCxJOfNyv5iaJ2DP0GqQV6odb1no46w/640?from=appmsg&watermark=1#imgIndex=33)

## 四、ClickHouse Group By 执行流程

### 4.1 ClickHouse Group By 执行流程简洁梳理

#### 1. 本地表执行（单节点）

*   **步骤 1：过滤读取**：
    

按WHERE用 MergeTree 分区 / 主键索引定位数据，仅读分组 + 聚合字段（列式存储减 IO）；

*   **步骤 2：分组计算**：
    

基数小（百万级内）用哈希表实时更新聚合值，基数大（千万级 +）先排序再累加（防内存溢出）；

*   **步骤 3：结果输出**：
    
*   直接返回分组结果，无需跨节点处理。
    

#### 2. 分布式表执行（多节点）

*   **阶段 1：本地预聚合**：
    

Distributed 拆查询到各节点，每个节点按本地逻辑计算，压缩数据（如数十亿明细→百万级分组，减 90% 传输量）；

*   **阶段 2：全局聚合**：
    

Distributed 收集各节点结果，可分解函数（SUM/COUNT）直接合并，不可分解函数（DISTINCT COUNT）需重算。

#### 3. 核心优化点

*   引擎层面：用分区键过滤缩小数据范围，主键含分组键省排序；
    
*   参数层面：超阈值开并行分组（group_by_two_level_threshold），按基数选哈希 / 排序模式（group_by_algorithm）；
    
*   SQL 层面：DISTINCT COUNT换approx_count_distinct，少分组字段，先过滤再聚合。
    

#### 4. 常见问题解决

*   数据倾斜：拆分倾斜分组、分组键作分片键、客户端合并结果；
    
*   内存溢出：切排序分组、临时加内存限制、按分区分批查；
    
*   结果不一致：用read_committed隔离、Decimal类型、固定协调节点。
    

### 4.2 、Group By 执行 Demo 演示

#### 1. 场景与表结构

统计 2024 年 1 月各用户的订单总金额（表orders为 Distributed 表，底层是 MergeTree 本地表，分区键create_date，主键(user_id, create_date)，分片键user_id）；

**表结构**：

user_id Int64, order_id Int64, order_amount Decimal(18,2), create_date Date。

#### 2. 执行 SQL

```
`  
SELECT   
 user_id,   
 SUM(order_amount) AS total_amount  -- 可分解聚合函数  
FROM orders   
WHERE create_date BETWEEN '2024-01-01' AND '2024-01-31'  -- 分区过滤  
GROUP BY user_id   
SETTINGS   
 group_by_algorithm = 'hash',  -- 分组基数小（假设用户数50万）用哈希  
 group_by_two_level_threshold = 100000;  -- 超10万分批并行  
`
```

#### 3. 执行流程拆解

**Distributed 表分发请求**：

协调节点将 SQL 拆为子查询，下发到集群 3 个节点（节点 A、B、C），子查询自动匹配各节点本地表orders_local。

**各节点本地预聚合（以节点 A 为例）**：

*   过滤：仅读create_date在 2024 年 1 月的 parts 文件，避免全表扫描；
    
*   读取：仅加载user_id（分组字段）、order_amount（聚合字段）；
    
*   分组：用哈希表统计，如将  user_id=101对应 3 笔订单，累加得total_amount=3500.50，生成本地结果（共 15 万条分组数据）。
    

**全局聚合**：

协调节点接收 3 个节点结果（A：15 万条，B：18 万条，C：17 万条），对相同user_id的total_amount求和。

比如： user_id=101在 B 节点为 2000.00，C 节点为 1500.00，最终合并为 7000.50 。

**结果返回**：

输出全局分组结果。

如user_id:101   total_amount:7000.50、user_id:102 total_amount:4200.20等。

#### 4. 优化效果体现

*   分区过滤：各节点仅扫描 1 月分区（30 个 parts），比全表（365 个 parts）IO 降 92%；
    
*   并行分组：因group_by_two_level_threshold=100000，节点 A 自动用 2 线程并行处理，计算耗时从 8 秒缩至 3 秒；
    
*   数据压缩：本地预聚合后，3 节点共传输 50 万条分组数据，而非原始 1 亿条明细，传输耗时降 99.5%。
    

## 说在最后：有问题找老架构取经‍

只要按照上面的 尼恩团队梳理的 方案去作答， 你的答案不是 100分，而是 120分。  面试官一定是 心满意足， 五体投地。

按照尼恩的梳理，进行 深度回答，可以充分展示一下大家雄厚的 “技术肌肉”，**让面试官爱到 “不能自已、口水直流”**，然后实现”offer直提”。

在面试之前，建议大家系统化的刷一波 5000页《[尼恩Java面试宝典PDF](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247497474&idx=1&sn=54a7b194a72162e9f13695443eabe186&scene=21#wechat_redirect)》，里边有大量的大厂真题、面试难题、架构难题。

很多小伙伴刷完后， 吊打面试官， 大厂横着走。

在刷题过程中，如果有啥问题，大家可以来 找 40岁老架构师尼恩交流。

另外，如果没有面试机会， 可以找尼恩来改简历、做帮扶。前段时间，**[刚指导一个小伙 暴涨200%（涨2倍），29岁/7年/双非一本 ， 从13K一次涨到 37K ，逆天改命](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486433&idx=1&sn=0f630e8fc7620b3bf0e054827b48d91e&scene=21#wechat_redirect)。**

狠狠卷，实现 “offer自由” 很容易的， 前段时间一个武汉的跟着尼恩卷了2年的小伙伴， 在极度严寒/痛苦被裁的环境下， offer拿到手软， 实现真正的 “offer自由” 。

## 惊天大逆袭： 通过  Java+AI  实现弯道超车，  完成转架构 

 [会 AI的程序员，工资暴涨50%！](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486827&idx=1&sn=db3f268c6752ddb074a98c7b13a84011&scene=21#wechat_redirect)

[极速上岸： 被裁 后， 8天 拿下 京东，狠涨 一倍 年薪48W， 小伙伴 就是 做对了一件事](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486913&idx=1&sn=edd6774e9d17c39327e8dcb95fdd68d9&scene=21#wechat_redirect)

[外包+二本 可以进 美团： 26岁小2本 一步登天， 进了顶奢大厂（ 美团） ， 太爽了](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486904&idx=1&sn=e7c878f99ed101ebfbd1d80ec0e07401&scene=21#wechat_redirect)

[暴涨 150%，4年 CRUD 一步登天， 进  ‘宇宙厂’， 26 岁 小伙 6个月 大逆袭](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486898&idx=1&sn=c1f7b50eb13ad990a19f099d9e608231&scene=21#wechat_redirect)

[Java+Al 大逆袭1： 34岁无路可走，一个月翻盘，拿 3个架构offer，靠  Java+Al   逆天改命！！！](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486848&idx=1&sn=ff43d058271b0801f84aba3f3d957855&scene=21#wechat_redirect)

java+AI 大逆袭2：[：3年 程序媛 被裁， 25W-》40W 上岸， 逆涨60%。 Java+AI 太神了， 架构小白 2个月逆天改命](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486858&idx=1&sn=9cd122c18d166433b43d0952d3a7a7a8&scene=21#wechat_redirect)

  

[Java+AI逆袭 ： 36岁/失业7个月/彻底绝望 。狠卷 3个月 Java+AI ，终于逆风翻盘，顺利 上岸](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486870&idx=1&sn=990a540a155cdb8254ebce6c75816882&scene=21#wechat_redirect)

  

[Java+AI逆袭 ： 闲了一年，41岁/失业12个月/彻底绝望 。狠卷 2个月 Java+AI ，终于逆风翻盘](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486880&idx=1&sn=a37033157c766ac5ae7c9195fe776693&scene=21#wechat_redirect)

## 冲大厂 案例：  全网顶尖、高薪案例， 进大厂拿高薪， 实现薪酬腾飞、人生逆袭 

  

[涨一倍：从30万 涨 60万，3年经验小伙 冲大厂成功，逆天了 ！！！](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486818&idx=1&sn=ec5ff9c809133d903bdecddcd25c440e&scene=21#wechat_redirect)

  

[阿里+美团offer：25岁 屡战屡败 绝望至极。找尼恩转架构升级，1个月拿到阿里+美团offer，逆天改命年薪 50W](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486794&idx=1&sn=fc1a1f843b0a91b61f7124ac0c2b51e2&scene=21#wechat_redirect)

  

[阿里offer：6年一本 不想 混小厂了。狠卷1年  拿到 得物 + 阿里 offer ， 彻底上岸 ，逆天改命](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486782&idx=1&sn=52ed05742c8afe7abf3b50405e78c853&scene=21#wechat_redirect)

  

  

## 大龄逆袭的案例： 大龄被裁，快速上岸的，远离没有 offer 的焦虑、恐慌 

  

  

[47岁超级大龄，被裁员后 找尼恩辅导收  2个offer，一个40多W。 35岁之后，只要 技术好，还是有饭吃，关键是找对方向，找对路子](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486323&idx=1&sn=a088584f97b657add9a058c149b72067&scene=21#wechat_redirect)

  

[大龄不难：39岁/15年老码农，15天时间40W上岸，管一个team，不用去 铁人三项了！](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486310&idx=1&sn=505e479da54cf1365afd4d39bddc18bd&scene=21#wechat_redirect)

## 草根逆袭， 100W 年薪 天花板 案例。 他们 如何 实现薪酬腾飞、人生逆袭？ 

  

[专科生 100年薪 ：35岁专科 草根逆袭，2线城市年薪100W 逆天改命， 从 超低起点 塔基（8W）--》塔腰-》塔尖（100W）](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486860&idx=1&sn=940405ff775e014bac52ba0636ea69e7&scene=21#wechat_redirect)

  

# **[年薪100W的底层逻辑：](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486315&idx=1&sn=52cc0a640c1499115f5a45b82097d857&chksm=97b57ae3a0c2f3f502293e84cde4bc892c45d06dee3579471a522ab497fb53e32640fb00c339&scene=21#wechat_redirect)** [大厂被裁，他们两个，如何实现年薪百万？](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486315&idx=1&sn=52cc0a640c1499115f5a45b82097d857&chksm=97b57ae3a0c2f3f502293e84cde4bc892c45d06dee3579471a522ab497fb53e32640fb00c339&scene=21#wechat_redirect) 

**[年薪100W](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485942&idx=1&sn=faa999fc9508435db7a88af2716492d9&chksm=97b5787ea0c2f168ac80d344670df16be12614d37161a893ebed88d4c915384fcbc024996403&scene=21#wechat_redirect)****[：](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485942&idx=1&sn=faa999fc9508435db7a88af2716492d9&chksm=97b5787ea0c2f168ac80d344670df16be12614d37161a893ebed88d4c915384fcbc024996403&scene=21#wechat_redirect)**[40](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485942&idx=1&sn=faa999fc9508435db7a88af2716492d9&chksm=97b5787ea0c2f168ac80d344670df16be12614d37161a893ebed88d4c915384fcbc024996403&scene=21#wechat_redirect)[岁小伙，被裁6个月，猛卷3月，100W逆袭 ，秘诀：升级首席架构/总架构](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485942&idx=1&sn=faa999fc9508435db7a88af2716492d9&chksm=97b5787ea0c2f168ac80d344670df16be12614d37161a893ebed88d4c915384fcbc024996403&scene=21#wechat_redirect)

[最新的100W案例：环境太糟，如何升 P8级，年入100W？](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486369&idx=1&sn=0b32e8907dbcf26eaf33ea3f37ddd099&chksm=97b57a29a0c2f33f468d29dafe739204543873efb459f324871c3c68a3ecc178b1a9ab08a2fa&scene=21#wechat_redirect)

##   

  

**职业救助站**

实现职业转型，极速上岸

  

![](https://mmbiz.qpic.cn/mmbiz_jpg/9qY2Gt3PBs3icE6WcM4nBOdRRcczOW4wkNDgCMZhBfqeet9suwGHfgQduP33YYqqZLOppecMI72Pv5ibNkPeIk6g/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1#imgIndex=34)

关注**职业救助站**公众号，获取每天职业干货  
助您实现**职业转型、职业升级、极速上岸**  
---------------------------------

**技术自由圈**

实现架构转型，再无中年危机

  

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/xlgvgPaib7WMtyFcJx45oIeWL1d6AENcdoAvuTPuq46E9n8pPe4SHXoSicuEVvhibHRSIzmEBPlC7POcDaqD6ID7w/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1#imgIndex=35)

关注**技术自由圈**公众号，获取每天技术千货  
一起成为牛逼的**未来超级架构师**

**几十篇架构笔记、5000页面试宝典、20个技术圣经  
请加尼恩个人微信 免费拿走**

**暗号，请在 公众号后台 发送消息：领电子书**

如有收获，请点击底部的"在看"和"赞"，谢谢