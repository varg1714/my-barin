---
source: https://mp.weixin.qq.com/s/cTo35wu9PBkRRrrm5QU-sQ
create: 2025-09-11 21:14
read: true
knowledge: true
knowledge-date: 2025-11-25
tags:
  - 数据库
  - Mysql
summary: "[[Mysql的存储结构]]"
---
FSAC 未来超级架构师

架构师总动员  
实现架构转型，再无中年危机

## 尼恩说在前面

在 40 岁老架构师 尼恩的**读者交流群** (50+) 中，最近有小伙伴拿到了一线互联网企业如得物、阿里、滴滴、极兔、有赞、希音、百度、网易、美团、蚂蚁、得物的面试资格，遇到很多很重要的相关面试题：

InnoDB 内存结构 和 磁盘 结构，  你理解吗？

什么是  Doublewrite Buffer ？InnoDB 是如何实现  Doublewrite Buffer  的？

比较 undo log、redo log 和 bin log 的作用和区别？

**最近有小伙伴在面 腾讯，问到了 mysql  InnoDB 存储引擎   相关的面试题。** 小伙伴  没有系统的去梳理和总结，所以支支吾吾的说了几句，面试官不满意，面试挂了。

所以，尼恩给大家做一下系统化、体系化的梳理，使得大家内力猛增，可以充分展示一下大家雄厚的 “技术肌肉”，**让面试官爱到 “不能自已、口水直流”**，然后实现”offer 直提”。

当然，这道面试题，以及参考答案，也会收入咱们的 《尼恩 Java 面试宝典 PDF》V175 版本，供后面的小伙伴参考，提升大家的 3 高 架构、设计、开发水平。

《尼恩 架构笔记》《尼恩高并发三部曲》《尼恩 Java 面试宝典》的 PDF，请到文末公号【技术自由圈】获取

本文作者：

*   第一作者  老架构师 肖恩（肖恩  是尼恩团队 高级架构师，负责写此文的第一稿，初稿 ）
    
*   第二作者   老架构师 尼恩   （**45 岁老架构师， 负责  提升此文的 技术高度，让大家有一种  俯视 技术、俯瞰技术、 技术自由  的感觉**）
    

# 一、InnoDB 存储引擎

## 1、MySQL 体系和 InnoDB 存储引擎

MySQL 的体系结构是分层设计的，包括 Server 层和 Engin 层。

**1. 服务层 (Server 层)**：处理连接、查询解析、优化、内置函数

**2. 存储引擎层 (Engin 层)**：负责数据存储 / 检索（可插拔）

Engin 层 是可以拔插设计， 可以 选择不同的 存储引擎。

InnoDB 属于  Engin 层 ，是默认的   存储引擎，  负责  "最终数据存储与管理" 的核心组件。

MySQL 整体架构：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHa457ibOibF1fCBDuK9nsfLn6BAmMwwzJfXRc3kQ6pWaaImro0Zx5mCsA/640?from=appmsg&watermark=1#imgIndex=0)

**各层作用**：

*   连接层：处理客户端接入（如 TCP 连接），验证密码，管理连接池。
    
*   服务层：负责 SQL 的解析、优化（比如选最优索引）、缓存，以及执行存储过程等。
    
*   存储引擎层：这是 MySQL 的 "数据管家"，通过统一接口与服务层交互。InnoDB 是其中功能最完善的（支持事务、行锁等），直接对接磁盘文件。
    
*   文件系统层：最终存储数据的物理文件（如`.ibd`数据文件、日志文件等）。
    

**关键点**：可拔插架构中，有一套规范的 I/O 操作接口，InnoDB 通过标准接口嵌入 MySQL，处理所有数据 I/O 操作

## 2、Inno DB 总体架构

InnoDB 存储引擎目前也是应用最广泛的存储引擎。 从 MySQL 5.5 版本开始作为表的默认存储引擎。

InnoDB 存储引擎 最早由 Innobase Oy 公司开发（属第三方存储引擎）。

InnoDB 存储引擎  是第一个完整支持 ACID 事务的 MySQL 存储引擎，特点是行锁设计、支持 MVCC、支持外键、提供一致性非锁定读，非常适合 OLTP 场景的应用使用。

InnoDB 存储引擎架构包含内存结构和磁盘结构两大部分

MySQL 8.0 版本，总体架构图如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHPNZUFSfyMMN6RMpIplhxCqoIdGKH9ibibl54GFZM898wBCfm91DUhjzw/640?from=appmsg&watermark=1#imgIndex=1)

MySQL 5.5 版本，总体架构图如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHhzNukpctibodFdIL37xN6ao5IMoicHRRGFfaYLQNw1gyeULeC72gwo1w/640?from=appmsg&watermark=1#imgIndex=2)

## 3、Inno DB 数据读写流程

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHhiaMXLHhAHUiaHud0qktsljeQCWgkDeHuJ9rFbGJKzHwCDkAPaTMad3A/640?from=appmsg&watermark=1#imgIndex=3)

**关键步骤：**

**1）读路径**：优先检查缓冲池，未命中时从. ibd(也就是各种表空间) 加载

**2）写路径**：

*   先写 redo log（顺序 I/O）
    
*   异步写入数据文件（随机 I/O）
    

```
SHOW ENGINE INNODB STATUS\G -- 查看刷脏进度
```

**3）崩溃恢复**：通过 redo log 重做未落盘操作

# 二、InnoDB 内存架构

InnoDB 的内存就像 "高速缓存区"，减少磁盘 IO，提升速度。

主要组件如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHpYWRhp6RGMe3ic2EZ61ShdEugauLSym8F7Sxhnc7JVIsJZYCLyUrIDQ/640?from=appmsg&watermark=1#imgIndex=4)

**核心组件**：

**(1) Buffer Pool `数据热区枢纽`：**

预分配连续内存缓存数据页，通过 LRU 算法管理热数据，将随机 I/O 转为内存操作。

**(2) Log Buffer `写操作高速通道`：**

暂存事务中的 redo 日志，`innodb_flush_log_at_trx_commit`控制刷盘策略，平衡性能与安全。

**(3) Change Buffer `非聚簇索引加速器`：**

缓存非唯一索引的 DML 操作（INSERT/UPDATE/DELETE），后台异步合并到磁盘索引结构。

**(4) 自适应哈希索引 `智能路径优化器`：**

自动检测高频等值查询路径，在内存中构建哈希索引，突破 B + 树检索深度限制。

**(5) undo 日志缓冲：**

`InnoDB 内存中临时存放 undo 日志的区域`，用于事务回滚和多版本控制，最终会刷新到磁盘的 Undo 表空间。

## 2.1、Buffer Pool

### 2.1.1 什么是 Buffer Pool？

简单说，Buffer Pool 是 InnoDB 存储引擎里一块**内存区域**，专门用来缓存表数据和索引数据。

就像我们平时把常用的文件放在桌面方便拿取，MySQL 也会把频繁访问的数据存到 Buffer Pool 里，避免每次都去读写磁盘（磁盘速度比内存慢太多），以此提高查询效率。

它是 InnoDB 性能的 “核心加速器”，大部分时候，我们查数据、改数据，都是和 Buffer Pool 打交道，而不是直接操作磁盘。

Buffer Pool 缓存磁盘数据页（16KB / 页）。

通过减少磁盘 I/O 提升性能，使用 **LRU 算法 + 冷热分离**管理数据页。

### 2.1.2 数据读取流程

```
```  SELECT * FROM table WHERE id=1; -- 直接返回内存数据
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHiaxJB5sW5KGQIMaGOzgtXcLBJsQTCsr4ASoicwPN3iakVEGgakLoFDWmw/640?from=appmsg&watermark=1#imgIndex=5)

**关键步骤说明：**

*   **缓存命中**（哈希表检索）：直接返回内存数据
    
*   **缓存未命中**：
    

 **从 `free_list`获取空闲页**

1.  若空 → 触发 **LRU 淘汰冷区尾部页**
    
2.   若为脏页 → **异步刷盘**
    
3.  从磁盘加载数据到空闲页
    

### 2.1.3 冷热数据迁移机制

冷热数据迁移是 MySQL 的 **内存优化策略**，通过将 Buffer Pool 中的内存页分为热区（高频访问）和冷区（低频访问），从而实现：

*   **保护热点数据**：高频访问页不被异常挤出
    
*   **隔离临时访问**：全表扫描等操作不污染热区
    
*   **智能淘汰**：优先释放低频使用的内存
    

**1）LRU 冷热分区结构**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHD5N02T5jpnWxia1ic1nh9K05waQwy59qsILgMUwNnQsdKicKTzc9fbPVg/640?from=appmsg&watermark=1#imgIndex=6)

<table><thead><tr><td><span><strong><span leaf="">位置</span></strong></span></td><td data-colwidth="194"><span><strong><span leaf="">特征</span></strong></span></td><td><span><strong><span leaf="">流动规则</span></strong></span></td></tr></thead><tbody><tr><td><section><span><strong><span leaf="">热区头部</span></strong></span></section></td><td data-colwidth="194"><section><span><span leaf="">最近高频访问页</span></span></section></td><td><section><span><span leaf="">持续访问则保留</span></span></section></td></tr><tr><td><section><span><strong><span leaf="">冷区头部</span></strong></span></section></td><td data-colwidth="194"><section><span><span leaf="">新加载页 / 降级页</span></span></section></td><td><section><span><span leaf="">二次访问可升热区</span></span></section></td></tr><tr><td><section><span><strong><span leaf="">冷区尾部</span></strong></span></section></td><td data-colwidth="194"><section><span><span leaf="">待淘汰页</span></span></section></td><td><section><span><span leaf="">内存不足时立即释放</span></span></section></td></tr></tbody></table>

关键控制参数

```
-- 冷区占比 (默认37%)SET GLOBAL innodb_old_blocks_pct = 37;  -- 冷区页停留最短时间 (默认1000ms)SET GLOBAL innodb_old_blocks_time = 1000;
```

**2）基本流程**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHV2QRycMz61vxUjU8nQCbIF1pcorL2IyZQOT9qiaME5F3KgXNaqQnySg/640?from=appmsg&watermark=1#imgIndex=7)

**3）冷热区规则**：

*   首次加载：插入 **冷区头部**
    
*   二次访问：移至 **热区头部**
    
*   **冷→热迁移条件**：
    
*   第二次访问该数据页
    
*   距首次加载时间 > `innodb_old_blocks_time`
    
*   访问间隔需超过过滤阈值
    
*   **热→冷降级条件**：
    
*   连续未访问时间 > 热区保护期
    
*   热区空间不足时尾部页降级
    
*   访问频率跌出热区保持阈值
    

**4）淘汰规则**：

```
if page in lru_cold and not recently_used: # 冷区尾部    evict_page(page)elif page in lru_hot and not accessed_in_time: # 热区未访问    move_to_cold_head(page) # 降级至冷区
```

**5）案例分析**

**场景：** 10GB 全表扫描

```
SELECT * FROM 10GB_table;  -- 数千万页级扫描
```

**保护机制：**

**(1) 所有新页插入冷区头部**

**(2) 1 秒内连续访问不触发升温**

**(3) 扫描结束自动从冷区尾部淘汰**

**(4) 热区 100% 不受影响**

**6）与传统 LRU 对比优势**

<table><thead><tr><td><span><strong><span leaf="">机制</span></strong></span></td><td><span><strong><span leaf="">传统 LRU</span></strong></span></td><td><span><strong><span leaf="">MySQL 冷热迁移</span></strong></span></td></tr></thead><tbody><tr><td><section><span><span leaf="">新页插入位置</span></span></section></td><td><section><span><span leaf="">直接放头部</span></span></section></td><td><section><span><span leaf="">冷区头部</span></span></section></td></tr><tr><td><section><span><span leaf="">全表扫描影响</span></span></section></td><td><section><span><span leaf="">立即污染热点区</span></span></section></td><td><section><span><span leaf="">完全隔离在冷区</span></span></section></td></tr><tr><td><section><span><span leaf="">淘汰策略</span></span></section></td><td><section><span><span leaf="">纯按访问时间</span></span></section></td><td><section><span><span leaf="">冷区优先 + 频率加权</span></span></section></td></tr><tr><td><section><span><span leaf="">二次机会</span></span></section></td><td><section><span><span leaf="">无</span></span></section></td><td><section><span><span leaf="">热区降级页回冷区头部</span></span></section></td></tr></tbody></table>

**本质价值**：通过物理隔离和延时升温机制，解决了传统 LRU 算法的 “缓存污染” 问题，使有限的 Buffer Pool 空间始终服务于真正的热点数据。

### 2.1.4 脏页刷盘机制

什么是  “脏页”？

当我们修改数据时，先改 Buffer Pool 里的缓存（内存），这时候缓存和磁盘数据就不一致了，这部分缓存叫 “脏页”。

#### 脏页怎么来的？

事务修改数据时，InnoDB 会先从磁盘加载目标数据页到 Buffer Pool（如果不在内存中），修改内存页后，会：

**(1) 标记该页为 “脏页”（dirty page）；**

**(2) 记录修改操作到 Redo Log（保证崩溃后能恢复）；**

**(3) 不立即写回磁盘（磁盘 IO 太慢，影响性能）。**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHcucMC0n2YEucSeWLLDGxuRceicF6EvUj8S8ibaY5aFW9dN4pBtjE4Rcw/640?from=appmsg&watermark=1#imgIndex=8)

脏页刷盘用到 mysql 的 checkpoint 机制。

#### Checkpoint 机制

什么是 checkpoint 机制呢？

InnoDB 改数据先在内存瞎折腾（Buffer Pool），不立马写到磁盘——怕慢。

但这么搞有俩问题：

*   万一崩了，内存里的改动丢了咋办？
    
*   Redo Log 总不能无限存吧？
    

Checkpoint 就是来解决这俩问题的。

说白了，它就是个「标记点」，告诉系统：“在我这时间点之前，所有内存里改了还没写到磁盘的脏页，都已经安全落地了”。

checkpoint 标记点以后的修改，崩溃恢复交给 redo log 处理

**Checkpoint 的核心作用：**

*   **缩短恢复时间**：崩溃恢复时只需要从最近的 checkpoint 开始重做日志
    
*   **脏页刷盘**：定期清理缓冲池中的脏页
    
*   **日志空间回收**：标记哪些 redo log 可以被覆盖重用
    

**Checkpoint 有哪几种？啥时候会触发（也就是啥时候进行脏页刷盘）？**

**1）Sharp Checkpoint（彻底型）**

就一种情况会触发：**数据库正常关闭（shutdown）**。

这时候会把所有脏页全刷到磁盘，Checkpoint 直接怼到 Redo Log 的末尾。下次启动不用恢复，因为啥都落盘了。简单粗暴，但耗时——生产库大的话，关一次可能等半天。

**2）Fuzzy Checkpoint（模糊型）**

数据库正常运行时用的，不刷所有脏页，只挑一部分刷，避免阻塞业务。细分为 4 种：

*   **Master Thread Checkpoint**：
    

主线程自己偷偷干的，每秒或每 10 秒（默认 1S）刷一点脏页（数量很少），不影响性能。比如每秒刷 10 个页，慢悠悠的。

*   **FLUSH_LRU_LIST Checkpoint**：
    

Buffer Pool 有个 LRU 链表（最近最少用），淘汰旧页时，发现是脏页，就得先刷盘再扔。不然扔了就丢数据了。

*   **Async/Sync Flush Checkpoint**：
    

这是急活儿！Redo Log 快写满时（write pos 快追上 Checkpoint 了），必须赶紧刷脏页推进 Checkpoint，给新日志腾地方。

要是还剩点空间，异步刷（不卡事务）；

要是快满了，同步刷（卡着新事务，直到腾出新空间）。

*   **Dirty Page too much Checkpoint**：
    

当脏页占 Buffer Pool 的比例超过 `innodb_max_dirty_pages_pct`（默认 75%），就触发刷盘，把比例压下去。避免脏页太多，万一崩了恢复慢。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHcucMC0n2YEucSeWLLDGxuRceicF6EvUj8S8ibaY5aFW9dN4pBtjE4Rcw/640?from=appmsg&watermark=1#imgIndex=9)

#### 核心流程图

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHg1THqY4Xdiaib3gsX7YTdxHdGfT9ibE68uBG81Z4vdzSiaVqkAJsMVYibEw/640?from=appmsg&watermark=1#imgIndex=10)

**脏页生命周期：**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHMb8uq7Cw9kjJrM6Zmw1CK3zJK6w2k9ckffoBicSX22ztoWT6vyb2FUQ/640?from=appmsg&watermark=1#imgIndex=11)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHRQCbBXkHNNOK9snUibcfB3O2RkQ9rGu64QuTFB2txpk0UskZCahhiaLA/640?from=appmsg&watermark=1#imgIndex=12)

### 2.1.5 性能调优实战

**关键配置参数**

<table><thead><tr><td><span><strong><span leaf="">参数</span></strong></span></td><td><span><strong><span leaf="">说明</span></strong></span></td><td><span><strong><span leaf="">推荐值</span></strong></span></td></tr></thead><tbody><tr><td><section><span><code><span leaf="">innodb_buffer_pool_size</span></code></span></section></td><td><section><span><span leaf="">总内存大小</span></span></section></td><td><section><span><span leaf="">物理内存的 50%-80%</span></span></section></td></tr><tr><td><section><span><code><span leaf="">innodb_old_blocks_pct</span></code></span></section></td><td><section><span><span leaf="">冷区内存占比</span></span></section></td><td><section><span><span leaf="">默认 37%</span></span></section></td></tr><tr><td><section><span><code><span leaf="">innodb_lru_scan_depth</span></code></span></section></td><td><section><span><span leaf="">每次扫描深度</span></span></section></td><td><section><span><span leaf="">默认 1024</span></span></section></td></tr></tbody></table>

**预热技巧**（重启后加载）：

```
SELECT pg.space_id, pg.page_noFROM information_schema.innodb_buffer_page AS pg;
```

**监控命令**：

```
SHOW ENGINE INNODB STATUS; -- Buffer Pool命中率 = (1 - disk_reads / logical_reads) * 100%
```

**核心价值**：将随机磁盘 I/O 转换为内存访问，加速高频数据操作。理解冷热分离与异步刷盘机制是调优关键。

## 2.2、Change Buffer

### 2.2.1 什么是 Change Buffer

简单说，Change Buffer 是 InnoDB 里一块**专门缓存非唯一二级索引修改**的内存区域。

作用：当你改数据时，如果要改的索引页不在内存里（Buffer Pool），不用立刻去磁盘找这个页，先把修改记在 Change Buffer 里，等以后有机会再一起处理。

打个比方： 这就像 网购时，快递员不会每到一个包裹就立刻送上门，而是攒一批顺路送——减少跑腿次数，效率自然高。

**为什么 Change Buffer 能提升性能？**

核心是**减少磁盘 IO**：

*   对非唯一二级索引的高频修改（比如批量插入），不用每次都读磁盘页，先在内存里 “记账”；
    
*   合并时一次性处理多个修改，把零散的磁盘操作变成集中操作。
    

反过来想：如果没有 Change Buffer，每插一条数据就要读一次磁盘索引页（如果不在内存），1000 条就是 1000 次磁盘 IO，慢得让人着急。

**Change Buffer 本质定位：**Change Buffer（写缓冲）是 InnoDB 加速 **非唯一二级索引变更** 的杀手锏。

**Change Buffer 核心价值：**将索引更新由随机写转为顺序写，解决二级索引写入瓶颈。

不是所有索引修改都能用 Change Buffer，有两个前提：

**(1) 必须是二级索引（非主键索引）；**

**(2) 这个索引不是唯一的（因为唯一索引要检查唯一性，必须访问磁盘页确认，绕不开）。**

像普通的 `INSERT`、`UPDATE`、`DELETE` 操作，只要符合上面两个条件，就可能用到 Change Buffer。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHVL9OG52Y0yMEHmBRE8iaia3LXJZpa4fyibRy8Vgzy4DojywENLb3PAnPw/640?from=appmsg&watermark=1#imgIndex=13)

**Change Buffer 运作全流程：**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHRuuicj9tqSLJkYvwxfVURXp9IIQgglnXFjVFVHPG0rRweeRBibIuNOXg/640?from=appmsg&watermark=1#imgIndex=14)

### 2.2.2 合并触发场景

存在 Change Buffer 里的修改，终究要写到磁盘的索引页上，这个过程叫 “合并”（merge）

```
-- 手动强制合并命令ALTER TABLE tbl_name FORCE CHANGE BUFFER MERGE;
```

### 2.2.3 性能调优实战

**关键控制参数**

```
-- 最大内存占比 (默认25%)SET GLOBAL innodb_change_buffer_max_size=30;-- 合并操作类型配置SET GLOBAL innodb_change_buffering='all'; -- 支持insert/delete/purge
```

**状态监控命令**

```
SHOW ENGINE INNODB STATUS\G---BUFFER POOL AND MEMORYIbuf: size 7549, free list len 3980, seg size 11530,merged operations: insert 5934234, delete mark 387703, delete 7392
```

**与传统方案对比**

<table><thead><tr><td><span><strong><span leaf="">特性</span></strong></span></td><td><span><strong><span leaf="">无 Change Buffer</span></strong></span></td><td><span><strong><span leaf="">Change Buffer 启用</span></strong></span></td></tr></thead><tbody><tr><td><section><span><span leaf="">二级索引更新</span></span></section></td><td><section><span><span leaf="">每次触发磁盘随机写</span></span></section></td><td><section><span><span leaf="">批量顺序写</span></span></section></td></tr><tr><td><section><span><span leaf="">索引页不在内存时</span></span></section></td><td><section><span><span leaf="">先读磁盘 → 更新 → 写回</span></span></section></td><td><section><span><span leaf="">内存记录 → 延迟合并</span></span></section></td></tr><tr><td><section><span><span leaf="">写性能</span></span></section></td><td><section><span><span leaf="">100TPS</span></span></section></td><td><section><span><span leaf="">10000+ TPS</span></span></section></td></tr><tr><td><section><span><span leaf="">适用场景</span></span></section></td><td><section><span><span leaf="">唯一索引更新</span></span></section></td><td><section><span><span leaf="">非唯一索引批量写入</span></span></section></td></tr></tbody></table>

**设计哲学**：

用内存换磁盘随机 I/O，牺牲数据落地实时性换取吞吐量跃升。

在账单记录、时序数据场景可带来 10 倍 + 写入性能提升，但对交易核心表需谨慎评估数据一致性要求。

### 2.2.4 Change Buffer 总结

Change Buffer 就是 InnoDB 为非唯一二级索引修改设计的 “内存暂存区”：

**(1) 先把修改记在内存，避免频繁访问磁盘；**

**(2) 等合适的时机（比如查询该索引时）再批量合并到磁盘；**

**(3) 核心价值：把零散的磁盘操作变成集中操作，大幅提升写性能。**

理解它，就能更好地优化那些带非唯一二级索引的表的写入性能了。

## 2.3、Log Buffer

### 2.3.1 什么是 Log Buffer

Log Buffer（日志缓冲区）是 InnoDB 的 **事务日志高速通道**，在内存中缓冲 redo log 数据，通过**批量合并写盘机制**将随机 I/O 转化为顺序 I/O，实现事务提交的瞬时响应。

简单说，Log Buffer 就是 InnoDB 存 redo log 的一块**内存缓冲区**。redo log 是保证数据安全的关键（比如断电时恢复数据），但直接写磁盘太慢，所以先放内存里攒一攒，凑够一批再写磁盘，这就是 Log Buffer 的作用。

打个比方：就像你写日记，不会写一个字就立刻存档，而是写满一页再存——减少存档次数，效率更高。

**基本流程图：**

当你执行增删改操作时，redo log 的产生和存储流程是这样的：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHhcxyPuwpR2WTkPbZdITEexp0lJqZyRccaxib1ia6kzKSOGmJSt3mn5RA/640?from=appmsg&watermark=1#imgIndex=15)

比如你连续执行 10 条 `UPDATE`，InnoDB 不会每条都写磁盘，而是先把这 10 条的 redo log 都放 Log Buffer 里，等满足条件了再一次性刷到磁盘，大大减少磁盘 IO 次数。

**什么时候会把 Log Buffer 刷到磁盘？**

<table><thead><tr><td><span><strong><span leaf="">触发条件</span></strong></span></td><td><span><strong><span leaf="">刷盘模式</span></strong></span></td><td><span><strong><span leaf="">数据安全性</span></strong></span></td><td><span><strong><span leaf="">性能影响</span></strong></span></td></tr></thead><tbody><tr><td><section><span><code><span leaf="">innodb_flush_log_at_trx_commit=1</span></code></span></section></td><td><section><span><span leaf="">事务提交同步刷</span></span></section></td><td><section><span><span leaf="">最高（ACID）</span></span></section></td><td><section><span><span leaf="">高延迟</span></span></section></td></tr><tr><td><section><span><code><span leaf="">innodb_flush_log_at_trx_commit=2</span></code></span></section></td><td><section><span><span leaf="">每秒后台异步刷</span></span></section></td><td><section><span><span leaf="">中等（OS 崩溃丢数据）</span></span></section></td><td><section><span><span leaf="">低延迟</span></span></section></td></tr><tr><td><section><span><span leaf="">Buffer 使用率 &gt; 75%</span></span></section></td><td><section><span><span leaf="">强制刷盘</span></span></section></td><td><section><span><span leaf="">防溢出</span></span></section></td><td><section><span><span leaf="">可控</span></span></section></td></tr><tr><td><section><span><span leaf="">Checkpoint 推进</span></span></section></td><td><section><span><span leaf="">连带刷盘</span></span></section></td><td><section><span><span leaf="">保证恢复点</span></span></section></td><td><section><span><span leaf="">周期影响</span></span></section></td></tr></tbody></table>

Checkpoint 的执行是 InnoDB 存储引擎的核心机制，其触发基于四大条件：

1.  ‌**时间周期**‌：秒级 / 分钟级定时触发
    
2.  ‌**空间阈值**‌：日志空间 >75% 或脏页比例 > 阈值
    
3.  事件驱动‌：关闭、备份等特殊操作
    
4.  负载压力‌：高并发写入时的自适应触发
    

Checkpoint 执行时完成的关键工作：

1.  确定最小安全 LSN 位置
    
2.   批量刷新 Redo 日志到磁盘
    
3.  按顺序写入脏数据页
    
4.  原子更新检查点元数据
    
5.  回收日志和数据页资源
    

### 2.3.2 性能调优实战

**关键控制参数**

```
-- 缓冲区大小 (默认16MB，建议1-4GB)SET GLOBAL innodb_log_buffer_size = 268435456; -- 256MB-- 刷盘策略 (1=全持久化, 2=高性能模式)SET GLOBAL innodb_flush_log_at_trx_commit = 2; -- 刷盘间隔 (默认1秒)SET GLOBAL innodb_flush_log_at_timeout = 2;
```

**状态监控命令：**

```
SHOW ENGINE INNODB STATUS\GLOGLog sequence number 182701152  // 当前LSNLog flushed up to   182701152  // 刷盘LSNPages flushed up to 182701152  // 页刷盘LSNLast checkpoint at  182701092  // 检查点LSN
```

**系统崩溃恢复逻辑：**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQH9XibTw3Z6DiaxKgGETeFzAvcRBbL0m0aBkcTHiaibBLOWibcXKokGJhoyVQ/640?from=appmsg&watermark=1#imgIndex=16)

### 2.3.3 Log Buffer 总结

核心是**减少磁盘 IO 次数**：

*   内存写速度比磁盘快 1000 倍以上，先放内存能让事务执行更快；
    
*   批量刷盘把多次小 IO 变成一次大 IO，效率更高。
    

但记住：Log Buffer 只是 “暂存”，最终还是要刷到磁盘才能保证数据安全，这就是为什么有各种刷盘策略的平衡。

**(1) 先存内存，减少磁盘 IO，让事务跑得更快；**

**(2) 靠定时、满了、事务提交这三个时机刷到磁盘，兼顾性能和安全；**

**(3) 大小和刷盘策略可以调，根据业务的 “速度需求” 和“安全需求”平衡。**

理解它，就能更好地配置 MySQL，在数据安全和写入性能之间找到合适的平衡点。

## 2.4、Adaptive Hash Index(自适应哈希索引)

### 2.4.1 什么是自适应哈希索引（AHI）？

自适应哈希索引（AHI）是 InnoDB 的**动态索引加速器**，自动将高频访问的 B + 树 路径转换为哈希索引。

自适应哈希索引（AHI） 核心目标：**将索引检索复杂度从 O(log n) 降至 O(1)**，针对热点数据查询实现毫秒级响应。

简单说，AHI 是 InnoDB 自己偷偷搞的 “加速工具”——它会盯着那些被频繁查询的索引，悄悄在内存里建哈希索引，帮你把某些查询速度提得更快。

哈希索引的特点是 “等值查询贼快”（比如 `WHERE id = 123`），但维护起来麻烦，还不适合范围查询（比如 `WHERE id > 100`）。

InnoDB 就想出个招：不麻烦人手动建，自己观察哪些查询频繁，符合哈希索引的脾气，就自动建，这就是 “自适应” 的意思。

**1）运作流程：**

不是随便什么查询都会触发 AHI，得满足几个条件：

*   必须是**等值查询**（比如 `\=、IN、<=>`）；
*   同一索引页（B + 树里的一个叶子节点）被频繁访问，且查询模式固定（比如总是查 `WHERE col = ?`）；
    
*   访问次数达到阈值（InnoDB 内部判断，不用我们管）。
    

举个例子：一张表有索引 `idx_name`，如果经常执行 `SELECT * FROM t WHERE name = '张三'` 这类查询，InnoDB 发现这个索引页被反复查，就会给这个页建个 AHI。

比如你第一次查 `name = '张三'`，没 AHI，走 B + 树查；查多了，InnoDB 建了 AHI，下次再查，直接通过哈希值定位到数据，不用再遍历 B + 树的层级，速度能快好几倍。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHibzUU2sDAZ6M5ToibGib3pXRpFjzZQzbcaDEAQvwEtqOz68a4dujaiaP8w/640?from=appmsg&watermark=1#imgIndex=17)

**2）AHI 会自动 “清理” 吗？**

会的。InnoDB 不只是建，还会盯着 AHI 的使用情况：

*   如果某个 AHI 建完后很少被用到，会自动删掉，腾内存；
    
*   当索引结构变化（比如删数据、改索引），对应的 AHI 也会跟着更新或删除，不用手动维护。
    

这也是 “自适应” 的体现——只留有用的，没用的自动清。

**3）AHI 适合什么场景？**

最适合 “大量等值查询” 的场景，比如：

*   电商商品详情页（频繁查 `WHERE goods_id = ?`）；
    
*   用户中心（频繁查 `WHERE user_id = ?`）。
    

这些场景下，AHI 能把查询从 “遍历 B + 树” 变成“哈希直接定位”，性能提升明显。

但如果是**范围查询多**（比如 `WHERE price > 100`），AHI 几乎没用，因为哈希索引不支持范围查找，这时甚至可能浪费内存。

**4）AHI 总结：**

AHI 就是 InnoDB 自带的 “智能加速插件”：

**(1) 自动观察频繁的等值查询，悄悄建哈希索引；**

**(2) 加速查询时直接定位数据，跳过 B + 树遍历；**

**(3) 没用的 AHI 自动删，不麻烦人维护；**

**(4) 适合等值查询多的场景，范围查询多可以关掉。**

理解它，就能更好地判断要不要留着这个 “加速工具”，让数据库跑更快。

### 2.4.2 性能调优实战

**适用场景：**

<table><thead><tr><td><span><strong><span leaf="">场景类型</span></strong></span></td><td><span><strong><span leaf="">加速效果</span></strong></span></td><td><span><strong><span leaf="">典型案例</span></strong></span></td></tr></thead><tbody><tr><td><section><span><span leaf="">主键点查询</span></span></section></td><td><section><span><span leaf="">8-10 倍</span></span></section></td><td><section><span><span leaf="">用户 ID 查询</span></span></section></td></tr><tr><td><section><span><span leaf="">短连接查询</span></span></section></td><td><section><span><span leaf="">5-7 倍</span></span></section></td><td><section><span><span leaf="">微服务 API 请求</span></span></section></td></tr><tr><td><section><span><span leaf="">排序索引访问</span></span></section></td><td><section><span><span leaf="">3-5 倍</span></span></section></td><td><section><span><span leaf="">分页顺序查询</span></span></section></td></tr><tr><td><section><span><span leaf="">大范围扫描</span></span></section></td><td><section><span><span leaf="">无提升</span></span></section></td><td><section><span><span leaf="">全表扫描</span></span></section></td></tr></tbody></table>

**控制参数：**

```
-- 全局开关 (默认ON)SET GLOBAL innodb_adaptive_hash_index = OFF;  -- 分区数设置 (默认8，解决锁竞争)SET GLOBAL innodb_adaptive_hash_index_parts = 16;-- 实时状态监控SHOW GLOBAL STATUS LIKE 'Innodb_ahi%';
```

**输出关键指标：**

```
+-----------------------------------+-------+| Variable_name                     | Value |+-----------------------------------+-------+| Innodb_ahi_searches               | 38245 |  # AHI查询次数| Innodb_ahi_inserts                | 1298  |  # AHI新增条目| Innodb_ahi_contention             | 83    |  # 哈希冲突次数+-----------------------------------+-------+
```

**与 B + 树索引对比：**

<table><thead><tr><td><span><strong><span leaf="">维度</span></strong></span></td><td><span><strong><span leaf="">B+Tree</span></strong></span></td><td><span><strong><span leaf="">AHI</span></strong></span></td></tr></thead><tbody><tr><td><section><span><span leaf="">索引类型</span></span></section></td><td><section><span><span leaf="">持久化结构</span></span></section></td><td><section><span><span leaf="">内存临时结构</span></span></section></td></tr><tr><td><section><span><span leaf="">构建方式</span></span></section></td><td><section><span><span leaf="">显示创建</span></span></section></td><td><section><span><span leaf="">自动按需生成</span></span></section></td></tr><tr><td><section><span><span leaf="">检索复杂度</span></span></section></td><td><section><span><span leaf="">O(log n)</span></span></section></td><td><section><span><span leaf="">O(1)</span></span></section></td></tr><tr><td><section><span><span leaf="">适用操作</span></span></section></td><td><section><span><span leaf="">范围 / 精确 / 排序</span></span></section></td><td><section><span><span leaf="">仅精确查询</span></span></section></td></tr><tr><td><section><span><span leaf="">内存占用</span></span></section></td><td><section><span><span leaf="">固定</span></span></section></td><td><section><span><span leaf="">动态增长 (最大 BPOOL 1/32)</span></span></section></td></tr><tr><td><section><span><span leaf="">更新代价</span></span></section></td><td><section><span><span leaf="">中</span></span></section></td><td><section><span><span leaf="">高 (需重建)</span></span></section></td></tr><tr><td><section><span><span leaf="">最佳场景</span></span></section></td><td><section><span><span leaf="">通用业务</span></span></section></td><td><section><span><span leaf="">超高频点查询</span></span></section></td></tr></tbody></table>

**核心价值**：对热点主键查询实现零层检索（直接内存定位），在交易系统核心表（如订单号查询）可提升 10 倍吞吐量。但高并发写入场景可能因重建开销导致 20% 性能下降，需通过`innodb_adaptive_hash_index_parts`缓解锁冲突。

# 三、InnoDB 磁盘架构

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHdiaOPR6qXs2pf0yd6mdFCz23xRluW9iaCoh5ThrrqmchM05y9B9CK6TQ/640?from=appmsg&watermark=1#imgIndex=18)

**核心组件：**

**(1) Redo Log（重做日志）**

`崩溃恢复的保险丝`：顺序记录所有物理数据变更，实例崩溃时通过重放保证 ACID 持久性，物理文件为`ib_logfile0/1`的循环写入。

**(2) Undo Log（回滚日志）**

`事务回滚的时光机`：存储数据修改前的原始镜像，支撑事务回滚和 MVCC 多版本读，MySQL 8.0 后独立存储在`undo_001`等专用表空间。

**(3) 系统表空间（System Tablespace）**

`引擎核心仓库`：默认存储数据字典、双写缓冲、Change Buffer 等元数据，主文件`ibdata1`持续增长且不可收缩。

**(4) 独立表空间（File-Per-Table Tablespace）**

`表专属数据容器`：每个 InnoDB 表独立的`.ibd`文件存储表数据 + 索引，通过`innodb_file_per_table=ON`启用，支持空间回收。

**(5) 通用表空间（General Tablespace）**

`多表共享存储池`：用户创建的跨表存储空间（`CREATE TABLESPACE`），可将多个表集中存储于自定义`.ibd`文件中。

**(6) 撤销表空间（Undo Tablespaces）**

`回滚日志专用住宅`：MySQL 8.0 + 默认将 undo log 从系统表空间剥离，存储在独立的`undo_001/002`文件，避免 ibdata1 膨胀。

**(7) 临时表空间（Temporary Tablespaces）**

`瞬时数据沙盒`：存储临时表及排序操作的磁盘中间数据，主文件`ibtmp1`随服务启动动态创建，重启自动清理。

## 3.1、Redo Log

Redo Log（重做日志）是 InnoDB 的**崩溃恢复核心组件**，采用物理逻辑日志结构，在事务提交前确保操作可恢复。

核心价值：**将随机数据写入转化为顺序日志写入**，实现 ACID 中的持久性（Durability）。

**Redo Log 是 InnoDB 存储引擎特有的物理日志**，记录数据页的物理修改（如表空间号、页号、偏移量、修改值），而非逻辑 SQL 语句（bin log 是逻辑日志，存储的是逻辑 sql）

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHzKhibe9QvB7Fm7L5ZrnN6icyhZibwyQVTyXticSdSA7NlFX7SA9XwweSYQ/640?from=appmsg#imgIndex=19)

### 3.1.1 WAL 机制

Redo Log 基于 Write-Ahead Logging（WAL）机制，即 “先写日志，后写磁盘”。事务提交时，先将修改记录写入 Redo Log Buffer 并刷盘，再异步将内存中的脏页写入磁盘。

这一机制通过顺序写（日志）替代随机写（数据页），显著降低 IO 开销。

即使事务提交后脏页未落盘，Redo Log 的存在仍能保证数据可恢复，从而提升性能并保障持久性。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHXqsXhsFpyQTeZiceGycUC1OBtLsc8WalodDQwmqhK0QzH8ewRWuRNjg/640?from=appmsg&watermark=1#imgIndex=20)

redo log 和 undo log 配合起来的作用就是：

*   事务提交前崩溃，通过 undo log 回滚事务
    
*   事务提交后崩溃，通过 redo log 恢复事务
    

### 3.1.2 循环写入

Redo Log File 是**循环写入**的，由多个日志文件组成文件组（如 4 个 1GB 文件），类似 “环形跑道”：

*   **write pos**：当前日志写入位置（不断向后移动）。
    
*   **checkpoint**：当前要覆盖的位置（需先将此位置前的脏页写入磁盘，才能推进）。
    

当 `write pos` 追上 `checkpoint` 时，数据库会先触发 checkpoint 机制（刷脏页到磁盘），再推进 `checkpoint`，避免日志被覆盖前数据丢失。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHeEgPtmxUawcfYZjGibSKTlWB8AXk676wgibaQOw4VliaWkbnWybI9GiaDw/640?from=appmsg&watermark=1#imgIndex=21)

*   write pos：当前记录写到的位置，或者说 当前 redo log 文件写到了哪个位置
    
*   checkpoint：当前要擦除的位置，或者说 目前 redo log 文件哪些记录可以被覆盖
    

这两个指针把整个环形划成了几部分

*   write pos - checkpoint：待写入的部分
    
*   checkpoint - write pos：还未刷入磁盘的记录
    

### 3.1.3 刷盘策略

先说明下 redo log 的三层存储架构：

**(1) 粉色，是 InnoDB 的一项很重要的内存结构 (In-Memory Structure)，日志缓冲区 (Log Buffer)，这一层，是 MySQL 应用程序用户态；**

**(2) 黄色，是操作系统的缓冲区 (OS cache)，这一层，是 OS 内核态；**

**(3) 蓝色，是落盘的日志文件；**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHRMtIe0RGgEw4hRX2dRlQ3hSvFFETHhp65cVNr727ZIoTWEfibDkuuOA/640?from=appmsg&watermark=1#imgIndex=22)

**Redo Log 是怎么刷盘的？**

**第一步：**事务提交的时候，会写入 Log Buffer，这里调用的是 MySQL 自己的函数 WriteRedoLog；

**第二步：**只有当 MySQL 发起系统调用写文件 write 时，Log Buffer 里的数据，才会写到 OS cache。注意，MySQL 系统调用完 write 之后，就认为文件已经写完，如果不 flush，什么时候落盘，是操作系统决定的；

**第三歩：**由操作系统（当然，MySQL 也可以主动 flush）将 OS cache 里的数据，最终 fsync 到磁盘上；

能够控制事务提交时，刷 redo log 的策略。目前有三种策略：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHG3Cz4GYzYwd372ZBvMDpSRmKpp31YuBZgqPGoK9YlK8Iz34cwP8tJQ/640?from=appmsg&watermark=1#imgIndex=23)

**策略一：最佳性能 (innodb_flush_log_at_trx_commit=0)**

*   每隔一秒，才将 Log Buffer 中的数据批量 write 入 OS cache，同时 MySQL 主动 fsync。
    
*   这种策略，如果数据库崩溃，有一秒的数据丢失。
    

**策略二：强一致 (innodb_flush_log_at_trx_commit=1)**

*   每次事务提交，都将 Log Buffer 中的数据 write 入 OS cache，同时 MySQL 主动 fsync。
    
*   这种策略，是 InnoDB 的默认配置，为的是保证事务 ACID 特性。
    

**策略三：折衷 (innodb_flush_log_at_trx_commit=2)**

*   每次事务提交，都将 Log Buffer 中的数据 write 入 OS cache；
    
*   操作系统决定刷盘时机（默认每秒一次 `fsync`）
    
*   MySQL 后台线程每秒也会主动触发一次 `fsync`‌，确保日志落盘
    
*   这种策略，如果操作系统崩溃，可能有一秒的数据丢失
    

策略三，如果操作系统崩溃，最多有一秒的数据丢失。因为 OS 也会 fsync，MySQL 主动 fsync 的周期是一秒，所以最多丢一秒数据。

策略三，磁盘 IO 次数不确定，因为操作系统的 fsync 频率并不是 MySQL 能控制的。

不同策略平衡了可靠性与性能，适用于不同业务场景（如金融系统选 1，高并发场景选 2 或 0）

## 3.2、Undo Log

Undo Log（回滚日志）是 InnoDB **事务原子性的基石**，记录事务修改前的数据镜像。核心职责：

**(1) 事务回滚：恢复到修改前状态**

**(2) MVCC 多版本：实现非锁定一致性读**

**(3) 崩溃恢复：与 Redo Log 协同保证数据完整性**

### 3.2.1 事务回滚（原子性）‌

在 undo log 日志中记录事务中的反向操作

*   事务进行 insert 操作，undo log 记录 delete 操作
    
*   事务进行 delete 操作，undo log 记录 insert 操作
    
*   事务进行 update 操作（value1 改为 value2 ），undolog 记录 update 操作（value2 改为 value1 ）
    

开启事务后，对表中某条记录进行修改（将该记录字段值由 value1 ——> value2 ——> value3 ），如果从整个修改过程中出现异常，事务就会回滚，字段的值就回到最初的起点（值为 value1 ）

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHk7RyzZNAPAhPFOKMtWRjw57via2CDiawula8rVZda2qrL7B5MAriczcjQ/640?from=appmsg&watermark=1#imgIndex=24)

*   trx_id 代表事务 id，记录了这一系列事务操作是基于哪个事务；
    
*   roll_pointer 代表回滚指针，就是当要发生 rollback 回滚操作时，就通过 roll_pointer 进行回滚，这个链表称为版本链。构建多版本链，支持精确回滚到特定版本
    

### 3.2.2 Undo Log MVCC 支持（隔离性）‌

Undo Log 为 MVCC 提供多版本数据快照，实现非阻塞读与隔离性。

*   版本链复用‌：每个事务通过 trx_id 和 roll_pointer 访问对应版本数据，避免读写冲突
    
*   ReadView 机制‌：结合隐藏字段（如 DB_TRX_ID）和 Undo Log 版本链，决定事务可见的数据版本
    
*   隔离级别适配‌：支持可重复读（RR）和读已提交（RC）等隔离级别，减少锁竞争
    

MVCC 能让读写不冲突，全靠 Undo Log 形成的 “版本链”。举个例子：

*   事务 1 改了 id=1 的数据（name 从'a' 变'b'），生成一条 Update Undo Log（记着 name='a'）；
    
*   事务 2 同时读 id=1，它的 Read View 会判断 “事务 1 没提交，不能看新值”，就顺着版本链找 Undo Log 里的老版本（name='a'）；
    
*   直到事务 1 提交，且没有事务再引用这个老版本，Purge 线程才会删掉这条 Undo Log。
    

MVCC 和事务的隔离性，请参见尼恩团队另外一篇重要文章：

[MVCC 学习圣经：一文穿透 MySQL MVCC，吊打面试官](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247501901&idx=1&sn=db7a9d223ee5d3bd9ee96bd7b713aa26&scene=21#wechat_redirect)

### 3.2.3 内存优化机制

Undo Log 通过内存缓存和异步清理优化性能

*   Buffer Pool 缓存‌：Undo 页缓存在内存中，加速回滚和 MVCC 访问
    
*   Redo Log 保护‌：Undo 页的修改会记录到 Redo Log，确保崩溃后仍可恢复
    
*   异步清理‌：事务提交后，Purge 线程回收不再需要的 Undo 页，减少内存占用
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHXtlR7SoUWuZdhFaxoWkNn1QGGWECXyA5VuictgaVpRyaklM4qtxJXZA/640?from=appmsg&watermark=1#imgIndex=25)

## 3.3、bin log

### 3.3.1 什么是 bin log 日志

Binlog（Binary Log） 是 MySQL 的核心日志机制，它不属于 Inno DB 组件，属于 MySQL 服务层，这里简单介绍下

Binlog（Binary Log） 是 MySQL 的核心日志机制，记录所有对数据库的 DDL 和 DML 变更操作（如增删改表结构、数据），但不包括查询语句（如 `SELECT`）。其核心作用是实现 数据恢复 和 主从复制一致性。

redo log 和 bin log 的区别：

*   redo log（重做日志）让 InnoDB 存储引擎拥有了崩溃恢复能力。
    
*   binlog（归档日志）保证了 MySQL 集群架构的数据一致性。
    

通过一张图比较下二者区别

<table><thead><tr><td><span><strong><span leaf="">特性</span></strong></span></td><td><span><strong><span leaf="">Bin Log</span></strong></span></td><td><span><strong><span leaf="">Redo Log</span></strong></span></td></tr></thead><tbody><tr><td><section><span><span leaf="">‌归属‌</span></span></section></td><td><section><span><span leaf="">MySQL Server 层</span></span></section></td><td><section><span><span leaf="">InnoDB 存储引擎</span></span></section></td></tr><tr><td><section><span><span leaf="">‌日志类型‌</span></span></section></td><td><section><span><span leaf="">逻辑日志（SQL 或行变更）</span></span></section></td><td><section><span><span leaf="">物理日志（数据页修改）</span></span></section></td></tr><tr><td><section><span><span leaf="">‌写入方式‌</span></span></section></td><td><section><span><span leaf="">追加写入（文件无限增长）</span></span></section></td><td><section><span><span leaf="">循环写入（固定大小）</span></span></section></td></tr><tr><td><section><span><span leaf="">‌持久化时机‌</span></span></section></td><td><section><span><span leaf="">事务提交时</span></span></section></td><td><section><span><span leaf="">事务提交时（强制刷盘）</span></span></section></td></tr><tr><td><section><span><span leaf="">‌主要用途‌</span></span></section></td><td><section><span><span leaf="">主从复制、时间点恢复</span></span></section></td><td><section><span><span leaf="">崩溃恢复、事务持久性</span></span></section></td></tr><tr><td><section><span><span leaf="">‌存储引擎依赖‌</span></span></section></td><td><section><span><span leaf="">与存储引擎无关</span></span></section></td><td><section><span><span leaf="">仅 InnoDB</span></span></section></td></tr></tbody></table>

bin log 日志格式

*   STATEMENT：记录 SQL 语句，日志量小但存在主从不一致风险（比如使用 `NOW()`）。
    
*   ROW：记录行级变更（旧值 / 新值），数据一致性高但日志量大。
    
*   MIXED：默认模式，自动切换 STATEMENT 和 ROW，平衡性能与一致性。
    

### 3.3.2 bin log 日志刷盘参数

**刷盘时机：**事务提交时，日志先写入内存缓存（`binlog_cache`），再根据 `sync_binlog` 参数决定是否持久化到磁盘。

**参数配置：**

*   `sync_binlog=0`：依赖系统刷盘，性能高但数据易丢失。
    
*   `sync_binlog=1`：每次提交立即刷盘，最安全但性能损耗大。
    
*   `sync_binlog=N`：累积 N 个事务后刷盘，折中方案。
    

这样，InnoDB 通过三大日志机制构建完整事务系统：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHRlmxzsfxt71m7k5faPggfofibphibOTibOXrmrwgYXuEEbYw8ztO4SC1g/640?from=appmsg&watermark=1#imgIndex=26)

## 3.4、MySQL 表空间

MySQL 表空间是 InnoDB 存储引擎管理数据的**核心物理容器**，分为五大类型：

**(1) 系统表空间（ibdata1）**

存储引擎元数据、双写缓冲区和 Change Buffer（MySQL 5.7 默认），通过`innodb_data_file_path`配置，**无法自动收缩**

**(2) 独立表空间（.ibd）**

启用`innodb_file_per_table=ON`后，每个表独占文件存放**数据 + 索引**，支持`OPTIMIZE TABLE`回收空间

**(3) 通用表空间**

用户用`CREATE TABLESPACE`创建，多个表共享. ibd 文件，适合管理大量小表

**(4) 撤销表空间（undo_*.ibd）**

MySQL 8.0 + 将 UNDO 日志从 ibdata1 剥离，默认 2 个文件动态循环写入，避免历史事务阻塞

**(5) 临时表空间（ibtmp1）**

存储临时表 / 排序数据，服务重启自动重建，`innodb_temp_data_file_path`控制大小

**版本演进**：

*   5.6 前：所有数据塞进 ibdata1（易暴涨）
    
*   5.7+：独立表空间成为默认
    
*   8.0：UNDO 日志独立，彻底解决 ibdata1 膨胀
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHU0jRNsJBUibw2lCNBVvF8L5PYR4A4CuiaqGwbODEMX1GOnSrAdVVwv1w/640?from=appmsg&watermark=1#imgIndex=27)

### 3.4.1 表空间结构

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHdgicgfo4FHeX7dzAeyNDTibvwNcID0vAgWjtz7tRRf8njxhpCTPT3zXw/640?from=appmsg&watermark=1#imgIndex=28)

表空间又由段 (segment)、区 ( extent)、页 (page) 组成，页是 InnoDB 磁盘管理的最小单位。

page 则是表空间数据存储的基本单位，innodb 将表文件（xxx.ibd）按 page 切分，依类型不同，page 内容也有所区别，最为常见的是存储数据库表的行记录。

表空间下一级称为 segment。segment 与数据库中的索引相映射。

Innodb 引擎内，每个索引对应两个 segment：**管理叶子节点的 segment 和管理非叶子节点 segment。**

**创建索引中很关键的步骤便是分配 segment，Innodb 内部使用 INODE 来描述 segment。**

segment 的下一级是 extent，extent 代表一组连续的 page，默认为 64 个 page，大小 1MB。

InnoDB 存储引擎的逻辑存储结构大致如下图 2 所示。

在我们执行 sql 时，不论是查询还是修改，mysql 总会把数据从磁盘读取内内存中，而且在读取数据时，不会单独加在一条数据，而是直接加载数据所在的数据页到内存中。

表空间本质上就是一个存放各种页的页面池。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHyNIibPPfpBCicO6yChYmmc0sE6Yiblau6Nk89ibr1QvxhWPBKaulLwX0HA/640?from=appmsg&watermark=1#imgIndex=29)

「page 页」是 InnoDB 管理存储空间的基本单位，也是内存和磁盘交互的基本单位。

也就是说，哪怕你需要 1 字节的数据，InnoDB 也会读取整个页的数据，

InnoDB 有很多类型的页，它们的用处也各不相同。

比如：有存放 undo 日志的页、有存放 INODE 信息的页、有存放 Change Buffer 信息的页、存放用户记录数据的页（索引页）等等。

InnoDB 默认的页大小是 16KB，在初始化表空间之前可以在配置文件中进行配置，一旦数据库初始化完成就不可再变更了。

```
SHOW VARIABLES LIKE 'innodb_page_size'
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHnoWjKdiaXkZ1ib1mbO0kQhAfMJ2Qqicuia4BV9ETaYVL8XPP4ZlwAE4Ipg/640?from=appmsg#imgIndex=30)

InnoDB 引擎 Row，Page 具体结构参考[腾讯 mysql 连环炮：索引、慢查询、深分页优化、sql 优化、并发事务问题、隔离级别、日志](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247505586&idx=1&sn=0ef67f21e190d796bbbc2ebbfc60b2cd&scene=21#wechat_redirect)

### 3.4.2 系统表空间

简单说，系统表空间是 InnoDB 存储引擎的 “大总管”——一块集中存放关键信息的磁盘空间，默认对应文件 `ibdata1`（在数据目录下）。它不像普通表那样只存自己的数据，而是管着一堆 InnoDB 运行必需的 “核心资产”，是数据库启动和运行的基础。

**系统表空间里到底存了啥？**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHO4pnjgVLhNxVqAj9ic1IuibRyHCoNLabunBibzyu8tfxXj0Yb0iaFNicm1g/640?from=appmsg&watermark=1#imgIndex=31)

*   **数据字典**：相当于数据库的 “户口本”，记录了所有表的结构（表名、字段、类型、索引信息等）。InnoDB 启动时必须读它，否则不知道有哪些表，没法干活。这部分是硬要求，必须存在系统表空间里，挪不走。
    

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHnLyd0cIEKagA9iaNoejlOIzzYIUkArTuh6gjsIH5o8zoWrlJlqPpKQA/640?from=appmsg&watermark=1#imgIndex=32)

*   **Undo 日志**：8.0 以下版本的 Undo Log 存储在系统表空间，事务的 “后悔药”。比如你执行 `UPDATE` 改了数据，Undo 日志就会记着 “之前的值是啥”，如果事务回滚（`ROLLBACK`），就靠它恢复原状。默认情况下，Undo 日志也存在系统表空间里。
    
*   **双写缓冲区（Doublewrite Buffer）**：防止数据 “写坏” 的保险。InnoDB 刷脏页到磁盘时，不会直接写数据文件，而是先写到双写缓冲区（相当于一个临时备份），确认写完了再同步到数据文件。如果中途断电，数据文件没写完整，下次启动可以从双写缓冲区恢复，避免数据损坏。这部分也固定在系统表空间。
    
*   **Change Buffer**：之前讲过的 “非唯一二级索引修改暂存区”，默认也存在系统表空间（它本质是一块特殊的内存区域，但会定期刷到磁盘，磁盘上的部分就存在这里）。
    
*   **用户表数据 / 索引（可选）**：如果创建表时没开 “独立表空间”（`innodb_file_per_table=OFF`），那么表的数据和索引会直接存在系统表空间里，和上面这些核心内容混在一起。
    

**系统表空间架构演进**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHpHVgH5khD0LM6qeY8oeYcgMBpEAyXT70sZyX5PlF3yu7RRr7Z4GHzA/640?from=appmsg&watermark=1#imgIndex=33)

<table><thead><tr><td><span><strong><span leaf="">组件</span></strong></span></td><td><span><strong><span leaf="">5.7 及更早</span></strong></span></td><td><span><strong><span leaf="">8.0+</span></strong></span></td></tr></thead><tbody><tr><td><section><span><span leaf="">数据字典</span></span></section></td><td><section><span><span leaf="">ibdata1</span></span></section></td><td><section><span><span leaf="">mysql.ibd</span></span></section></td></tr><tr><td><section><span><span leaf="">Undo Log</span></span></section></td><td><section><span><span leaf="">ibdata1</span></span></section></td><td><section><span><span leaf="">undo_001/002</span></span></section></td></tr><tr><td><section><span><span leaf="">用户表数据</span></span></section></td><td><section><span><span leaf="">可选共享</span></span></section></td><td><section><span><span leaf="">独立. ibd 文件</span></span></section></td></tr><tr><td><section><span><span leaf="">Change Buffer</span></span></section></td><td><section><span><span leaf="">ibdata1</span></span></section></td><td><section><span><span leaf="">ibdata1</span></span></section></td></tr><tr><td><section><span><span leaf="">双写缓冲区</span></span></section></td><td><section><span><span leaf="">ibdata1</span></span></section></td><td><section><span><span leaf="">#ib_16384_0.dblwr</span></span></section></td></tr><tr><td><section><span><span leaf="">临时表空间</span></span></section></td><td><section><span><span leaf="">ibtmp1</span></span></section></td><td><section><span><span leaf="">#innodb_temp</span></span></section></td></tr></tbody></table>

**怎么配置系统表空间？**

主要靠 `my.cnf`（或 `my.ini`）里的参数控制：

**(1) 指定文件路径和大小：**

```
```ini   [mysqld]   innodb_data_file_path = ibdata1:12M:autoextend  # 默认配置：ibdata1，初始12M，自动增长   # 也可以指定多个文件，比如：   # innodb_data_file_path = ibdata1:50M;ibdata2:50M:autoextend  # 两个文件，ibdata2满了自动涨
```

**(2) 是否用独立表空间存用户表：**

```
```ini   [mysqld]   innodb_file_per_table = ON  # 开独立表空间（默认值），用户表数据存在单独的 .ibd 文件，不占系统表空间
```

建议一直开着 `innodb_file_per_table=ON`，这样用户表数据存在独立的 `.ibd` 文件里（比如 `t1.ibd`），删表时 `.ibd` 会被删掉，不会让 `ibdata1` 越来越大。

### 3.4.3 独立表空间

简单说，独立表空间就是让每张表的 “数据和索引” 单独存成一个文件（比如 `t1.ibd`），不和系统表空间（`ibdata1`）混在一起。就像每个家庭有自己独立的户口本，不用都塞在一个大本子里，管理起来方便多了。

MySQL 5.6 之后，这个功能默认是打开的（`innodb_file_per_table=ON`），现在基本都是这么用。

独立表空间（File-Per-Table）是 InnoDB 的**表级存储革命**，每个用户表拥有专属的. ibd 文件，实现物理存储的完全隔离。核心价值：

**(1) 空间自治：表级空间管理**

**(2) 性能隔离：IO 操作分散**

**(3) 运维灵活：单表备份 / 迁移**

### 3.4.4 通用表空间

通用表空间（General Tablespaces）是 InnoDB 的**高级存储容器**，允许将多个表聚合存储在共享的物理文件中，突破 "一个表 = 一个文件" 的限制，实现存储级别的灵活整合。

**核心架构**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHPTYPJroaKhb1TLbZwbtLBibY1yy4JicQhQF7vvEgR03ibGxmzWGZJQEow/640?from=appmsg&watermark=1#imgIndex=34)

**关键操作命令**

```
-- 在数据目录下创建一个叫 app_data 的通用表空间，文件是 app_data.ibdCREATE TABLESPACE app_data ADD DATAFILE 'app_data.ibd' ENGINE=InnoDB;-- 也可以指定绝对路径（比如放另一个磁盘）CREATE TABLESPACE log_data ADD DATAFILE '/data/mysql/logs/log_data.ibd'  -- 自定义路径ENGINE=InnoDB;-- 新建表时直接放进 app_data 表空间CREATE TABLE user ( id int PRIMARY KEY,  name varchar(50)) TABLESPACE app_data;  -- 关键：指定表空间-- 把已有的独立表空间的表（t_order）移到 app_data 里ALTER TABLE t_order TABLESPACE app_data;SELECT   table_name,   tablespace_name FROM   information_schema.tables WHERE   table_schema = '你的库名';-- 先把表移走（比如移回独立表空间）ALTER TABLE user TABLESPACE = innodb_file_per_table;  -- 独立表空间的特殊名称-- 或者直接删表DROP TABLE t_order;-- 再删表空间DROP TABLESPACE app_data;
```

**架构精髓**：通用表空间如同数据库的 "多功能集装箱"，通过物理聚合打破存储孤岛。特别适用于：

1.  SaaS 系统（万级小表合并存储）
    
2.  时序数据（冷热分级归档）
    
3.  敏感数据（统一加密管理）
    
4.  黄金法则：当单个实例超过 500 个表时，通用表空间可显著降低文件系统压力！
    

### 3.4.5 撤销表空间

简单说，撤销表空间就是专门存 Undo 日志的 “独立仓库”。Undo 日志是事务的 “后悔药”——比如你执行 `UPDATE` 改了数据，它会记下 “改之前的值是啥”，万一事务回滚（`ROLLBACK`），就靠它恢复原状。

撤销表空间（Undo Tablespaces）是 MySQL 8.0+ 的**事务时光机**，独立存储 UNDO 日志，实现：

**(1) 原子性保障：事务回滚能力**

**(2) MVCC 支持：多版本并发控制**

**(3) 空间自治：独立回收机制**

MySQL 8.0 默认创建 2 个 Undo 表空间文件（`undo_001` 和 `undo_002`），每个初始大小为 16MB，通过参数 `innodb_undo_tablespaces` 可调整数量（范围 2-127），每个文件初始 16MB，支持自动扩展和截断回收。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHu0QSsfvvL9KBepa4WHibXtUVGpiaFMBlISFVNfrBUmyV01nIU5RhoibzA/640?from=appmsg&watermark=1#imgIndex=35)

**Undo 表空间的逻辑层级管理是咋样的？**

**回滚段（Rollback Segments）**：每个 Undo 表空间包含 128 个回滚段（由 `innodb_rollback_segments` 控制），每个回滚段管理 1024 个 Undo 段（Undo Segments）。

**Undo 页与日志记录**：Undo 段由多个 16KB 的页组成，按事务类型分为 Insert Undo 段（仅用于回滚）和 Update Undo 段（用于 MVCC），前者事务提交后立即释放，后者需等待无活跃读视图时清除。

通过多 Undo 表空间与回滚段的分区设计，理论上支持高达数万级并发事务（例如：128 表空间 × 128 回滚段 × 1024 Undo 段）。

如下图所示。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHcFzJAxdvEJ02Q2GPWAotaiarhFLYXdFzeiaYVib8NZFq9fbfKuUle83Cg/640?from=appmsg&watermark=1#imgIndex=36)

**关键说明**：

*   每个 Undo 表空间包含 **128 个回滚段**
    
*   每个回滚段管理 **1024 个 Undo 段**（按事务类型分类）
    
*   Undo 段由 **16KB 页** 组成，存储具体日志记录
    

**说说 Undo Log 与 MVCC 的协作机制？**

Undo Log 与 MVCC 的协作机制如下图所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHm3XJ4Ygzd1wbGcguJoeANVdjHibQka7xZRoeQPmCyHSJOoVZlNiabZYw/640?from=appmsg&watermark=1#imgIndex=37)

**运作原理**：

*   事务修改前将旧数据写入 Undo Log
    
*   读事务通过 Read View 判断可见性
    
*   多版本数据通过 Undo Log 链回溯访问
    

**系统表空间与 Undo 表空间存储有啥区别？**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHSmYfQdrG1SFun7ATzaMBWbibdDdWHRa2FahjQib8Blsw7vXZYVzibKnHw/640?from=appmsg&watermark=1#imgIndex=38)

<table><thead><tr><td><span><strong><strong><span leaf="">特性</span></strong></strong></span></td><td><span><strong><strong><span leaf="">Undo 表空间</span></strong></strong></span></td><td><span><strong><strong><span leaf="">系统表空间（历史方案）</span></strong></strong></span></td></tr></thead><tbody><tr><td><section><span><strong><span leaf="">存储内容</span></strong></span></section></td><td><section><span><span leaf="">仅 Undo Log</span></span></section></td><td><section><span><span leaf="">数据字典、双写缓冲、Undo Log 等混合内容</span></span></section></td></tr><tr><td><section><span><strong><span leaf="">空间管理</span></strong></span></section></td><td><section><span><span leaf="">支持自动截断，避免文件膨胀</span></span></section></td><td><section><span><span leaf="">无法自动回收，需手动调整或重建</span></span></section></td></tr><tr><td><section><span><strong><span leaf="">性能影响</span></strong></span></section></td><td><section><span><span leaf="">减少 I/O 竞争，提升并发处理能力</span></span></section></td><td><section><span><span leaf="">高频事务易导致文件过大，性能下降</span></span></section></td></tr><tr><td><section><span><strong><span leaf="">版本支持</span></strong></span></section></td><td><section><span><span leaf="">MySQL 5.7+ 默认方案</span></span></section></td><td><section><span><span leaf="">MySQL 5.6 及更早版本</span></span></section></td></tr></tbody></table>

### 3.4.6 临时表空间

临时表空间（Temporary Tablespaces）是 MySQL 的**高速暂存区**，专为临时数据处理设计，存储：

**(1) 用户创建的临时表（CREATE TEMPORARY TABLE）**

**(2) 优化器生成的内部临时表（排序 / 分组等）**

**(3) 在线 DDL 操作的中间数据**

**核心架构**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHRPTovmCA4gxOvInsKQmUGaLRK3ADwniasFN4JJxzI4GXtc32wnhp8Kg/640?from=appmsg&watermark=1#imgIndex=39)

<table><thead><tr><td><span><strong><span leaf="">MySQL 版本</span></strong></span></td><td><span><strong><span leaf="">临时表空间方案</span></strong></span></td></tr></thead><tbody><tr><td><section><span><span leaf="">≤5.7</span></span></section></td><td><section><span><span leaf="">共享 ibtmp1 文件</span></span></section></td></tr><tr><td><section><span><span leaf="">8.0+</span></span></section></td><td><section><span><span leaf="">全局 ibtmp1 + 会话级独立文件</span></span></section></td></tr></tbody></table>

InnoDB 临时表空间分为 **会话临时表空间** 和 **全局临时表空间**，分别承担不同角色：

**会话临时表空间（Session Temporary Tablespaces）**

*   用途：**存储用户显式创建的临时表（CREATE TEMPORARY TABLE）以及优化器生成的内部临时表（如排序、分组操作）**。
    
*   **生命周期**：会话断开时自动截断并释放回池，文件扩展名为 `.ibt`，默认位于 `#innodb_temp` 目录。
    
*   **分配机制**：首次需要创建磁盘临时表时，从预分配的池中分配（默认池包含 10 个表空间文件），每个会话最多分配 2 个表空间（用户临时表与优化器内部临时表各一）。
    

**全局临时表空间（Global Temporary Tablespace）**：

*   **用途**：存储用户临时表的回滚段（Rollback Segments），支持事务回滚操作。
    
*   **文件配置**：默认文件名为 `ibtmp1`，初始大小 12MB，支持自动扩展，由参数 `innodb_temp_data_file_path` 控制路径与属性。
    
*   **回收机制**：服务器重启时自动删除并重建，意外崩溃时需手动清理。
    

**Temporary Tablespaces 物理结构**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQH66wFFWVcx29o15NpOcRjiaB5GQs0LGpiaVtDK0oCwXo9hiaQybh8swyCA/640?from=appmsg&watermark=1#imgIndex=40)

**图示说明**：

*   **全局临时表空间**：`ibtmp1` 存储用户临时表的回滚段
    
*   **会话临时表空间**：`#innodb_temp` 目录下预分配 10 个 `.ibt` 文件池（默认配置）
    
*   每个会话最多激活 2 个临时表空间（用户临时表 + 优化器内部临时表）。
    

**会话级临时表空间生命周期**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHLlrCkF39nxIjAJnUwHIcKwwqxKQYIcM1nibzCsH3bQLETKadFlj4VCQ/640?from=appmsg&watermark=1#imgIndex=41)

**关键点**：

**(1) 首次需要磁盘临时表时从池中分配**

**(2) 会话断开连接后立即归还空间**

**(3) 文件物理保留但内容截断（类似内存池机制）**

**临时表空间使用查询流程**

前面说过临时表空间可**存储用户显式创建的临时表（CREATE TEMPORARY TABLE）以及优化器生成的内部临时表（如排序、分组操作）**。

那它的查询过程是怎样的呢？

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHjG9oppyagnAl4MicRLrNKDbN24fHVTAFUErhia6BaPnY6fOPH3YnWKOw/640?from=appmsg&watermark=1#imgIndex=42)

## 3.5 Doublewrite Buffer

简单说，Doublewrite Buffer 是 InnoDB 防止数据 “写坏” 的一道保险。

当 InnoDB 把内存里的脏页（改过但没刷到磁盘的数据）写到磁盘时，不是直接写到数据文件（.ibd），而是先写一份到 Doublewrite Buffer，确认安全后再同步过去。

这就像你保存重要文档时，先存到 U 盘一份，再存到电脑硬盘——万一存硬盘时突然断电，至少 U 盘里还有备份，不会丢数据。

双写缓冲（Doublewrite Buffer）是 InnoDB 的**数据安全卫士**，通过 "先写副本再写正本" 的两段式写入机制，解决**部分页写入（Partial Page Write）** 问题，确保数据页崩溃恢复的完整性。

**核心工作流程图**

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHYjAKxceib7TEy6iavyNI2qqO1oQmxDqcmPt0XuksABsUcdGiawuABIhmg/640?from=appmsg&watermark=1#imgIndex=43)

### 3.5.1 为什么写数据需要 Double write Buffer

MySQL 程序是跑在 Linux 操作系统上的，理所当然要跟操作系统交互，

一般来说，MySQL 中一页数据是 16kb，操作系统一个页是 4kb，所以，mysql page 刷到磁盘，要写 4 个文件系统里的页。

如图所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQH6ChcJyiaD5aarQyyCmvl2yqhH3g8ic7bHy5FMAzLvqv3BPNicmuSsoP6w/640?from=appmsg&watermark=1#imgIndex=44)

**需要注意的是，这个操作并非原子操作，比如我操作系统写到第二个页的时候，Linux 机器断电了，这时候就会出现 partial page write（部分页写入） 问题了,  造成” 页数据损坏 “。**

**并且,  这种” 页数据损坏 “靠 redo 日志是无法修复的**。

Redo log 中记录的是对页的局部修改操作，而不是页面的全量记录。 换句话说，Redo log 记录的是修改前后的差异，而不是整个数据页的内容。

如果发生 partial page write（部分页写入）问题时，出现问题的页面的全量记录，这里包括哪些没有被 Redo log 记录 未修改过的数据，此时重做日志 (Redo Log) 无能为力。

Doublewrite Buffer 的出现就是为了解决上面的这种情况。

虽然名字带了 Buffer，但实际上 Doublewrite Buffer 是**内存 + 磁盘**的结构。

Doublewrite Buffer 是一种特殊文件 flush 技术，带给 InnoDB 存储引擎的是数据页的可靠性。

它的作用是，在把页写到磁盘数据文件之前，**InnoDB 先把它们写到一个叫 doublewrite buffer（双写缓冲区）的共享表空间内，在写 doublewrite buffer 完成后，InnoDB 才会把页写到数据文件的适当的位置。**

如果在写页的过程中发生意外崩溃，InnoDB 在稍后的恢复过程中在 doublewrite buffer 中找到完好的 page 副本用于恢复。

### 3.5.2 Doublewrite Buffer 原理

Doublewrite Buffer 采用 **内存 + 磁盘双层结构**，关键组件如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHXG7iaQWx5H6mb3Jo3LgakZkENwpW5WZDT0hgibMRb3wMoDAwN8Se1ASg/640?from=appmsg&watermark=1#imgIndex=45)

**内存结构**

*   容量固定为 **128 个页（2MB）**，每个页 16KB。
    
*   数据页刷盘前，通过 `memcpy` 拷贝至内存 Doublewrite Buffer。
    

**磁盘结构**

*   位于系统表空间（`ibdata`），分为 **2 个区（extent1/extent2）**，共 2MB。
    
*   数据以 **顺序写** 方式写入，避免随机 I/O 开销。
    

工作流程如下图所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHDSGYmhuPobw7pDoxsWHhDFuB3jDt5l17icAib5lDREQgqyIPd9ia84JIw/640?from=appmsg&watermark=1#imgIndex=46)

**如上图所示，当有数据修改且页数据要刷盘时：**

**(1) 第一步：记录 Redo log。**

**(2) 第二步：脏页从 Buffer Pool 拷贝至内存中的 Doublewrite Buffer。**

**(3) 第三步：Doublewrite Buffer 的内存里的数据页，会 fsync 刷到 Doublewrite Buffer 的磁盘上，分两次写入磁盘共享表空间中 (连续存储，顺序写，性能很高)，每次写 1MB；**

**(4) 第四步：Doublewrite Buffer 的内存里的数据页，再刷到数据磁盘存储 .ibd 文件上（离散写）；**

时序图如下：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WPvQ6ygL2YYybcZQcLOaTQHcdnFIF2WNONpw1MHEhLXOVniblkM5Xcb9zibLROX4libqbJxOictUxS0VA/640?from=appmsg&watermark=1#imgIndex=47)

**崩溃恢复**

如果第三步前，发生了崩溃，可以通过第一步记录的 Redo log 来恢复。

如果第三步完成后发生了崩溃， InnoDB 存储引擎可以从共享表空间中的 Double write 中找到该页的一个副本，将其复制到独立表空间文件，再应用 Redo log 恢复。

在正常的情况下，MySQL 写数据页时，会写两遍到磁盘上，第一遍是写到 doublewrite buffer，第二遍是写到真正的数据文件中，**这就是 “Doublewrite” 的由来。**

**Doublewrite Buffer 通过 两次写 机制，在内存和磁盘间构建冗余副本，成为 InnoDB 保障数据完整性的基石。**

**其架构设计平衡了性能与可靠性，尤其在高并发或异常宕机场景下表现突出。**

### 3.5.3 Doublewrite Buffer 相关参数

以下是一些与 Doublewrite Buffer 相关的参数及其含义：

*   `innodb_doublewrite`： 这个参数用于启用或禁用双写缓冲区。设置为 1 时启用，设置为 0 时禁用， 默认值为 1。
    
*   `innodb_doublewrite_files`： 这个参数定义了多少个双写文件被使用。默认值为 2，有效范围从 2 到 127。
    
*   `innodb_doublewrite_dir`： 这个参数指定了存储双写缓冲文件的目录的路径。默认为空字符串，表示将文件存储在数据目录中。
    
*   `innodb_doublewrite_batch_size`: 这个参数定义了每次批处理操作写入的字节数。默认值为 0，表示 InnoDB 会选择最佳的批量大小。
    
*   `innodb_doublewrite_pages`：这个参数定义了每个双写文件包含多少页面。默认值为 128。
    

### 3.5.4 Doublewrite Buffer 和 redo log

在 MySQL 的 InnoDB 存储引擎中，Redo log 和 Doublewrite Buffer 共同工作以确保数据的持久性和恢复能力。

**(1) 首先 wal 架构：**

当有一个 DML（如 INSERT、UPDATE）操作发生时， InnoDB 会首先将这个操作写入 redo log（内存）。这些日志被称为未检查点（uncheckpointed）的 redo 日志。

**(2) 然后，在修改内存中相应的数据页之后，需要将这些更改记录在磁盘上。**

但是直接把这些修改的页写到其真正的位置可能会因发生故障导致页部分更新，从而导致数据不一致。

因此，InnoDB 的做法是先将这些修改的页按顺序写入 doublewrite buffer。

这就是为什么叫做 "doublewrite" —— 数据实际上被写了两次，先在 doublewrite buffer，然后在它们真正的位置。

**(3) 一旦这些页被安全地写入 doublewrite buffer，它们就可以按原始的顺序写回到文件系统中。**

即使这个过程在写回数据时发生故障，我们仍然可以从 doublewrite buffer 中恢复数据。

**(4) 最后，当事务提交时，相关联的 redo log 会被写入磁盘。**

这样即使系统崩溃，redo log 也可以用来重播（replay）事务并恢复数据库。

在系统恢复期间，InnoDB 会检查 doublewrite buffer，并尝试从中恢复损坏的数据页。

如果 doublewrite buffer 中的数据是完整的，那么 InnoDB 就会用 doublewrite buffer 中的数据来更新损坏的页。

否则，如果 doublewrite buffer 中的数据不完整，InnoDB 也有可能丢弃 buffer 内容，重新执行那条 redo log 以尝试恢复数据。

所以，Redo log 和 Doublewrite Buffer 的协作可以确保数据的完整性和持久性。如果在写入过程中发生故障，我们可以从 doublewrite buffer 中恢复数据，并通过 redo log 来进行事务的重播。

### 3.5.5  Doublewrite Buffer 总结

Doublewrite Buffer 是 InnoDB 的一个重要特性，用于保证 MySQL 数据的可靠性和一致性。

它的实现原理是通过将要写入磁盘的数据先写入到 Doublewrite Buffer 中的内存缓存区域，然后再写入到磁盘的两个不同位置，来避免由于磁盘损坏等因素导致数据丢失或不一致的问题。

总的来说，Doublewrite Buffer 对于改善数据库性能和数据完整性起着至关重要的作用。尽管其引入了一些开销，但在大多数情况下，这些成本都被其提供的安全性和可靠性所抵消。

## 说在最后：有问题找老架构取经‍

按照此文的套路去回答，一定会 **吊打面试官，让面试官爱到 “不能自已、口水直流”**，然后实现”offer 直提”。

在面试之前，建议大家系统化的刷一波 5000 页《[尼恩 Java 面试宝典 PDF](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247497474&idx=1&sn=54a7b194a72162e9f13695443eabe186&scene=21#wechat_redirect)》，里边有大量的大厂真题、面试难题、架构难题。

很多小伙伴刷完后， 吊打面试官， 大厂横着走。

在刷题过程中，如果有啥问题，大家可以来 找 40 岁老架构师尼恩交流。

另外，如果没有面试机会， 可以找尼恩来改简历、做帮扶。

前段时间，尼恩 刚辅导一个 **[外包 + 二本小伙](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486904&idx=1&sn=e7c878f99ed101ebfbd1d80ec0e07401&scene=21#wechat_redirect)** [进](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486904&idx=1&sn=e7c878f99ed101ebfbd1d80ec0e07401&scene=21#wechat_redirect) **[美团](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486904&idx=1&sn=e7c878f99ed101ebfbd1d80ec0e07401&scene=21#wechat_redirect)**[： 一步登天  进了顶奢大厂（ 美团） ， 26 岁小 2 本 逆天改命](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486904&idx=1&sn=e7c878f99ed101ebfbd1d80ec0e07401&scene=21#wechat_redirect)

跟着 尼恩  狠狠卷，实现 “offer 自由”  ， 逆天改命  很容易的。

## 惊天大逆袭： 通过  Java+AI  实现弯道超车，  完成转架构 

 [会 AI 的程序员，工资暴涨 50%！](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486827&idx=1&sn=db3f268c6752ddb074a98c7b13a84011&scene=21#wechat_redirect)

  

[极速上岸： 被裁 后， 8 天 拿下 京东，狠涨 一倍 年薪 48W， 小伙伴 就是 做对了一件事](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486913&idx=1&sn=edd6774e9d17c39327e8dcb95fdd68d9&scene=21#wechat_redirect)

  

[外包 + 二本 可以进 美团： 26 岁小 2 本 一步登天， 进了顶奢大厂（ 美团） ， 太爽了](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486904&idx=1&sn=e7c878f99ed101ebfbd1d80ec0e07401&scene=21#wechat_redirect)

  

[暴涨 150%，4 年 CRUD 一步登天， 进  ‘宇宙厂’， 26 岁 小伙 6 个月 大逆袭](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486898&idx=1&sn=c1f7b50eb13ad990a19f099d9e608231&scene=21#wechat_redirect)

  

[Java+Al 大逆袭 1： 34 岁无路可走，一个月翻盘，拿 3 个架构 offer，靠  Java+Al   逆天改命！！！](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486848&idx=1&sn=ff43d058271b0801f84aba3f3d957855&scene=21#wechat_redirect)

  

java+AI 大逆袭 2：[：3 年 程序媛 被裁， 25W-》40W 上岸， 逆涨 60%。 Java+AI 太神了， 架构小白 2 个月逆天改命](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486858&idx=1&sn=9cd122c18d166433b43d0952d3a7a7a8&scene=21#wechat_redirect)

  

  

[Java+AI 逆袭 ： 36 岁 / 失业 7 个月 / 彻底绝望 。狠卷 3 个月 Java+AI ，终于逆风翻盘，顺利 上岸](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486870&idx=1&sn=990a540a155cdb8254ebce6c75816882&scene=21#wechat_redirect)

  

[Java+AI 逆袭 ： 闲了一年，41 岁 / 失业 12 个月 / 彻底绝望 。狠卷 2 个月 Java+AI ，终于逆风翻盘](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486880&idx=1&sn=a37033157c766ac5ae7c9195fe776693&scene=21#wechat_redirect)

## 冲大厂 案例：  全网顶尖、高薪案例， 进大厂拿高薪， 实现薪酬腾飞、人生逆袭 

[涨一倍：从 30 万 涨 60 万，3 年经验小伙 冲大厂成功，逆天了 ！！！](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486818&idx=1&sn=ec5ff9c809133d903bdecddcd25c440e&scene=21#wechat_redirect)

[阿里 + 美团 offer：25 岁 屡战屡败 绝望至极。找尼恩转架构升级，1 个月拿到阿里 + 美团 offer，逆天改命年薪 50W](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486794&idx=1&sn=fc1a1f843b0a91b61f7124ac0c2b51e2&scene=21#wechat_redirect)

  

[阿里 offer：6 年一本 不想 混小厂了。狠卷 1 年  拿到 得物 + 阿里 offer ， 彻底上岸 ，逆天改命](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486782&idx=1&sn=52ed05742c8afe7abf3b50405e78c853&scene=21#wechat_redirect)

## 大龄逆袭的案例： 大龄被裁，快速上岸的，远离没有 offer 的焦虑、恐慌 

[47 岁超级大龄，被裁员后 找尼恩辅导收  2 个 offer，一个 40 多 W。 35 岁之后，只要 技术好，还是有饭吃，关键是找对方向，找对路子](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486323&idx=1&sn=a088584f97b657add9a058c149b72067&scene=21#wechat_redirect)

[大龄不难：39 岁 / 15 年老码农，15 天时间 40W 上岸，管一个 team，不用去 铁人三项了！](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486310&idx=1&sn=505e479da54cf1365afd4d39bddc18bd&scene=21#wechat_redirect)

## 草根逆袭， 100W 年薪 天花板 案例。 他们 如何 实现薪酬腾飞、人生逆袭？ 

[专科生 100 年薪 ：35 岁专科 草根逆袭，2 线城市年薪 100W 逆天改命， 从 超低起点 塔基（8W）--》塔腰 -》塔尖（100W）](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486860&idx=1&sn=940405ff775e014bac52ba0636ea69e7&scene=21#wechat_redirect)

# **[年薪 100W 的底层逻辑：](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486315&idx=1&sn=52cc0a640c1499115f5a45b82097d857&chksm=97b57ae3a0c2f3f502293e84cde4bc892c45d06dee3579471a522ab497fb53e32640fb00c339&scene=21#wechat_redirect)** [大厂被裁，他们两个，如何实现年薪百万？](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486315&idx=1&sn=52cc0a640c1499115f5a45b82097d857&chksm=97b57ae3a0c2f3f502293e84cde4bc892c45d06dee3579471a522ab497fb53e32640fb00c339&scene=21#wechat_redirect) 

**[年薪 100W](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485942&idx=1&sn=faa999fc9508435db7a88af2716492d9&chksm=97b5787ea0c2f168ac80d344670df16be12614d37161a893ebed88d4c915384fcbc024996403&scene=21#wechat_redirect)****[：](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485942&idx=1&sn=faa999fc9508435db7a88af2716492d9&chksm=97b5787ea0c2f168ac80d344670df16be12614d37161a893ebed88d4c915384fcbc024996403&scene=21#wechat_redirect)**[40](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485942&idx=1&sn=faa999fc9508435db7a88af2716492d9&chksm=97b5787ea0c2f168ac80d344670df16be12614d37161a893ebed88d4c915384fcbc024996403&scene=21#wechat_redirect)[岁小伙，被裁 6 个月，猛卷 3 月，100W 逆袭 ，秘诀：升级首席架构 / 总架构](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247485942&idx=1&sn=faa999fc9508435db7a88af2716492d9&chksm=97b5787ea0c2f168ac80d344670df16be12614d37161a893ebed88d4c915384fcbc024996403&scene=21#wechat_redirect)

[最新的 100W 案例：环境太糟，如何升 P8 级，年入 100W？](http://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486369&idx=1&sn=0b32e8907dbcf26eaf33ea3f37ddd099&chksm=97b57a29a0c2f33f468d29dafe739204543873efb459f324871c3c68a3ecc178b1a9ab08a2fa&scene=21#wechat_redirect)

**职业救助站**

实现职业转型，极速上岸

![](https://mmbiz.qpic.cn/mmbiz_jpg/9qY2Gt3PBs3icE6WcM4nBOdRRcczOW4wkNDgCMZhBfqeet9suwGHfgQduP33YYqqZLOppecMI72Pv5ibNkPeIk6g/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1#imgIndex=48)

关注**职业救助站**公众号，获取每天职业干货  
助您实现**职业转型、职业升级、极速上岸**  
---------------------------------

**技术自由圈**

实现架构转型，再无中年危机

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/xlgvgPaib7WMtyFcJx45oIeWL1d6AENcdoAvuTPuq46E9n8pPe4SHXoSicuEVvhibHRSIzmEBPlC7POcDaqD6ID7w/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1#imgIndex=49)

关注**技术自由圈**公众号，获取每天技术千货  
一起成为牛逼的**未来超级架构师**

**几十篇架构笔记、5000 页面试宝典、20 个技术圣经  
请加尼恩个人微信 免费拿走**

**暗号，请在 公众号后台 发送消息：领电子书**

如有收获，请点击底部的 " 在看 "和" 赞 "，谢谢