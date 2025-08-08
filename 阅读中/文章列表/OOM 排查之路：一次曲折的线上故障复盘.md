---
source: https://mp.weixin.qq.com/s/lMV9vIM-aPAZ0ftzKSERqg
create: 2025-08-04 22:48
read: false
knowledge: false
---
![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naIBeQJB5fNWyGOLK6Zun0MxibqKtdcowbV5MS5X3fj9SibVcr6HyYC4kkX4lec883VgDKwsc3mSPGmw/640?wx_fmt=jpeg&from=appmsg&randomid=tf6h4w31)

我们的服务整合了 Paimon 数据湖与 RocksDB，通过 SDK 负责数据的查询与写入。近期，该系统在线上环境连续发生了三次内存溢出（OOM）故障。排查过程颇为曲折，笔者与团队成员尝试了多种方法，走了不少弯路，最终成功定位到问题根源并将其妥善解决。

本文旨在将这段 “曲折” 的排查经历抽丝剥茧，分享我们是如何一步步逼近真相并最终解决问题的，希望能为使用相似技术栈的朋友带来一些启发。

一、问题的发现 & 解决

**1.1 第一次 OOM**

### 现象

某天早上，收到了服务的线上告警，发现大批量 RPC 请求都失败了，登录相关的服务平台才发现，所有对外的 RPC 服务，全部都下线了。

根据故障排查的经验，此时应该先对服务进行止血，并且保留现场用于排查问题，于是在重启一台机器后，观察另一台机器的监控指标，在监控指标中，我们注意到了一个异常现象。

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naKP15JoCaKYT4A6AORN3c1CBt2icPtFUoibldUiaXStKDEUuXic1yZmohDgVcYf0v7OS7UnP7gQrucX0g/640?wx_fmt=png&from=appmsg&randomid=e8q4dzqf)

Java 线程数量如上图所示，Runnable 的线程数量在某个时间点突增（上图只截取了一部分时间的监控，实际上线程数量会在一些固定时间点突增）。

### 排查

这些固定时间，自然成为了我们首先怀疑的方向，这些固定时间，均是在整点附近，而我们的服务是在整点，通过定时任务调度 SDK 向 Paimon 表中写入数据。

我们询问了提供相关 SDK 的团队同学，一起排查后发现，我们所建的 Paimon 表是依赖公司内部其他中间建表的，在没有指定 bucket 数量时，会默认 100 个 bucket，而 SDK 会在每张表的每一个 bucket 在写的时候都会开一个线程，最终就会有 表数量 x 100 个线程在跑，这个数量也符合我们观察到的 Java 线程数。

### 解决

后续和相关同学沟通后，决定减少 bucket 数量，根据查阅相关资料，Paimon 表的 bucket 数量应该参考以下设置：

*   数据量较小的场景（OLTP 场景）
    

*   设置较小的 bucket 数量，一般在 4-16 之间，较少的 bucket 数量也可以对查询效率有一些提升。
    

*   海量数据（高并发流式写入场景）
    

*   设置 64、128、256 个 bucket
    

总体来说可以参考以下公式，设置为 2 的次方个 bucket

bucket 数量 ≈ (预计的最大写入并行度) * N (其中 N 通常取 1 到 4)

在调整 bucket 数量后，修复上线，线程数降到了理想范围，解决了线程数量突增的问题。

**1.2 第二次 OOM**

### 现象

上次 OOM 问题解决后，我们加强了服务的相关告警。然而时隔 20 多天后，线上服务又告警了，现象依旧是所有对外的 RPC 服务全部下线了。

登录监控平台查看相关 JVM 信息，Java 线程数一切正常，但是内存占用率已经到了 95%+。

登录机器，使用如下命令，发现 Java 进程被 Kill 了。

```
dmesg | grep -i "killed process"
```

将监控平台对内存利用率的查询时间周期拉长后发现，自从上次重启之后，内存利用率一直在缓慢上升。

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naKP15JoCaKYT4A6AORN3c1C93DGQT4vI38SFnIKJt5iba5NmGIA73saH6TFmsTSQQ00pyuASZYSOJA/640?wx_fmt=png&from=appmsg&randomid=d4yzc6e9)

因为内存利用率是缓慢上升，而非突增，只能随着时间的推移，不断地排查内存泄漏的原因，于是我们开启了一段为期半个月的内存泄漏排查旅程。

### 排查

#### 堆内排查

首先，我们对 JVM 内存相关的监控指标进行了排查，观察是否由堆内存泄漏导致的 OOM。

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naKP15JoCaKYT4A6AORN3c1CqXAEcpwOia0BY62IEvXUsekibfFpRCfbuRSRlny0uEN7ogHdj2ONOCicQ/640?wx_fmt=png&from=appmsg&randomid=znur7876)

监控突变如上所示，从图中可以看到，“已使用堆内存” 呈现周期性的波动，基本可以确认是正常的 GC 导致的波动，并且机器的内存是 8G，堆内存最大也不过 4G。而老年代的内存占用量也在 0 左右，并未出现波动。

基于以上分析，我们可以明确排除因 Java 对象持续堆积而导致的堆内存泄漏。故障的根源必定在于堆外内存。

#### 堆外排查

线程数量分析

对 Java 线程数量进行分析，可以看到在上次调整 bucket 数量之后，线程数量十分稳定，可以排除 Java 线程数量增长导致的 OOM。

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naKP15JoCaKYT4A6AORN3c1CqaKFhPeIPvAOTb1nR9aPakaufKPjibrJnVZTicruVKzviakG39K0XWRyA/640?wx_fmt=png&from=appmsg&randomid=f9voc1ty)

DirectMemory 和 JNIMemory

使用集团内部的 MAT 文件，分析了 Dump 文件，发现堆外内存中都是 java.nio.DirectByteBuffer。

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naKP15JoCaKYT4A6AORN3c1CHL7pzLRZkyZw3W4xKaeiaiaZNE0gGiaLFiakagicA64jpeblYVwW1wYytYg/640?wx_fmt=png&from=appmsg&randomid=k9acmcx4)

这个类是 NIO 的类，阅读相关文章后，找到相关资料，其中提到集团内部的 RPC 框架使用 Netty，可能会申请堆外内存，且无法监控到，慢慢导致能存利用率上升。

使用 Arthas 的 memory 命令分析了系统的内存分布，结果如下：

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naKP15JoCaKYT4A6AORN3c1CaRMRtDsb4N9IH6ot6uick1Fo1RJvoBK1vW9VQGrX524psnU0OtYnS8Q/640?wx_fmt=png&from=appmsg&randomid=y47tqbn1)

可以看到，direct 占了 312M，而其他应用的内存分布如下：

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naKP15JoCaKYT4A6AORN3c1Cr3PbImx4no20MPDGICuRBRibz7wVe8Gk1Vy7UaTFPSIKOozLm3dxZIg/640?wx_fmt=png&from=appmsg&randomid=4f6rdyfd)

direct 只有 8M，这两者相差较多。

继续分析 Netty 占用，结果如下：

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naKP15JoCaKYT4A6AORN3c1Cb3ibwcRE0gibzJnCLK0UbAYyjr98eAyfXlqHQtFrnbryzMibCuIh7HmTw/640?wx_fmt=png&from=appmsg&randomid=nvv04xci)

将所有的 netty 占用加起来，确实占用了 300M。但 300M 也远远不会让我们的应用 OOM，显然这不是系统 OOM 根因。

使用 NMT 工具排查，先记录了 baseline，在一天过后执行了一次 diff。

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naKP15JoCaKYT4A6AORN3c1CEo9acRhUIqVnRJIjJjy2FovUn4ibwiagKaRa2von8sgbvicVdmX5xTvkg/640?wx_fmt=png&from=appmsg&randomid=1mj1wgip)

可以看到，committed 一天不过增长了 57M，这也和内存利用率的上涨对应不上。

用 async-profiler，抓取了一段时间系统运行堆栈的内存分布火焰图，来观察哪些类的上涨比较多，当内存的 RES 上涨 100m 后，产出了火焰图，发现火焰图中记录的总共只有 4M，这和 RES 上涨差的也很多。

最后用 pmap 命令对比了内存上涨前后的 diff，也并未发现异常问题。

### 解决

尽管我们已将问题初步定位于堆外内存，但由于堆外内存泄漏的成因复杂且监控手段有限，此次排查并未直接定位到根本原因。

最后，我们与 JVM 专家团队紧密协作，制定了一系列手段来解决内存利用率上涨的问题。

1. 适度调低 JVM 堆内存上限（`-Xmx`），将更多物理内存预留给堆外空间使用。

2. 加上 -XX:+AlwaysPreTouch 参数。

默认情况下，JVM 向操作系统申请的堆内存是 “懒加载” 的，只有在实际使用时才会触发物理内存的分配。这会导致监控到的容器内存曲线随时间推移而 “自然” 增长，对我们判断是否存在 “额外” 的内存泄漏造成视觉干扰。启用`AlwaysPreTouch`能让 JVM 在启动时就一次性占用所有分配的堆内存。

3. 增加机器内存。

4. 升级 RPC 框架的 netty 共享 sar 包，减少 netty 占用。

**1.3 第三次 OOM**

### 现象

在采用了一系列手段后，本以为不会发生 OOM，然而观察到线上机器的内存利用率仍然不断上涨。

机器 A 的内存利用率：

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naKP15JoCaKYT4A6AORN3c1CYAxaqvqtXUCSK8N9DRnib5evlibtxFHZmfMAGorlxp0eA7dyCb5GOdyg/640?wx_fmt=png&from=appmsg&randomid=zglfb4pl)

机器 B 的内存利用率：

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naKP15JoCaKYT4A6AORN3c1Cq0GCTEV5aLd9R0nKBjIO3q12wBAnStBYLQ7Eed3OQY9J9pkP3jibTPg/640?wx_fmt=png&from=appmsg&randomid=974pd698)

我们的服务在线上共有两台机器，在此我们将其称之为机器 A 和机器 B，两台机器的七日内存利用率监控如上图所示。

从图中可以明显看到，机器 A 的内存利用率基本是呈阶梯状，每隔一段时间就会上涨一部分；机器 B 的内存利用率并未有什么大的变化。

这时，我们想到，第二次排查时使用的机器，并未有线上真实流量，但机器 A 和机器 B 是线上的真实机器。而机器 A 和机器 B 的唯一差别就是，线上大部分通过 RocksDB 写 Paimon 的请求，都由机器 A 执行，而机器 A 的内存利用率的阶梯上涨时间，都是写 Paimon 发生的时间。

通过控制变量法，我们再次将问题锁定在了相关团队提供的 SDK 上。

### 排查

1. NMT 分析

使用 ps -aux 命令后，看到 Java 进程的 RSS 有 12GB。

使用 NMT 工具后，产出了如下报告：

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naKP15JoCaKYT4A6AORN3c1CicElibZ8qAqjkdP5GdT50KNeQgRaLJAoSich9M80bkRhAqNF4cCV3BD5A/640?wx_fmt=png&from=appmsg&randomid=skmu9ynj)

可以看到 committed 约为 7G，这 5 GB 的差距主要由以下几部分组成，这些都是 NMT 无法追踪的：

*   第三方本地库（Native Libraries）内存占用
    

当 Java 代码通过 JNI（Java Native Interface）调用 C/C++ 等语言编写的本地库时，这些本地库内部使用 malloc、new 等方式申请的内存，完全不受 JVM 的 NMT 系统追踪。然而，这部分内存确实是属于该 Java 进程的，因此会被操作系统计入 RSS。

而在 NMT 文件中，可以明显看到以下内容：

```
[0x00007f850c52a70f] rocksdb::JniCallback::getJniEnv(unsignedchar*) const+0x17f
```

我们的应用确实使用了 RocksDB，向 Paimon 写数据。RocksDB 是一个用 C 编写的高性能嵌入式键值数据库。它在运行时会管理大量的堆外内存（off-heap memory）用于缓存（Block Cache）、索引（Index）、布隆过滤器（Bloom Filter）等。这些内存都是 RocksDB 自己通过 C 的内存分配器向操作系统申请的，JVM 的 NMT 对此一无所知。这部分内存往往非常大，几 GB 是很常见的。

*   Direct ByteBuffer（直接内存）
    

通过 java.nio.ByteBuffer.allocateDirect() 分配的内存虽然是堆外内存，但现代的 JDK 版本中，NMT 通常能将其统计在 Internal 或 Other 类别下。

```
Other (malloc=67369KB ...)
Internal (malloc=16291KB ...)
```

我们的 NMT 文件中，这两部分并未占用大量内存。

*   加载的共享库 / 动态库
    

Java 进程本身依赖于大量的共享库（.so 文件），例如：libc.so (C 标准库)、libpthread.so (线程库) 、libjvm.so (JVM 核心库) 、应用依赖的其他任何原生库文件

操作系统会将这些库的代码段和数据段映射到进程的地址空间。这部分内存也会被计入 RSS，而 NMT 不会统计它们。虽然多个进程可以共享同一个库的物理内存页，但 RSS 的计算方式仍会将这部分计入。

*   JNI 本身的开销
    

JNI 调用本身以及 JNI 句柄等也会占用少量内存，这部分也不在 NMT 的统计范围内。

*   内存分配器的碎片化
    

JVM 向操作系统申请内存时，底层通常使用 glibc 的 malloc。malloc 本身为了性能，可能会预先分配比请求稍大的内存块，并且在释放后可能会持有这些内存以备将来使用（形成内存池），而不是立即归还给操作系统。这就导致了 RSS（OS 看到的）会比 JVM 实际使用的（NMT 看到的）要高。

实际上，RocksDB 使用 JNI 分配内存和我们根据内存利用上涨的猜测结果相似，为了确定问题的根源，我们又采用了 async-profiler 分析。

2. async-profiler 火焰图

抓取了通过 RocksDB 写 Paimon 时期的堆栈，可以看到：  

![](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naKP15JoCaKYT4A6AORN3c1CD5qRll9ZqoFoSlVicjxDTxiaib5INY2dtzvLSrKeDblNN8jA5Ryt4p5mw/640?wx_fmt=png&from=appmsg&randomid=jobye20i)

火焰图中大部分消耗也在 RocksDB。

3. 询问 SDK 提供团队

我们带着以上两个排查结果，再次询问了 RocksDB-Paimon SDK 提供团队的同学，他们查看了源代码后发现，我们现在使用的 SDK 存在 RocksDB 使用 JNI 分配内存，但无法释放的问题。

至此，困扰我们几个月的内存泄露问题终于找到了真正的原因。

#### 解决

和相关团队同学沟通后，我们最终决定转换写 Paimon 的方式，从 通过自己的应用写 Paimon（旧架构），转换为应用发送消息通过 Flink 写 Paimon（新架构）。

实际上，在业界一般来说都是通过 Flink 来写 Paimon，因为 Flink 为数据入湖提供了一整套成熟、强大的能力，这是在业务应用中自行实现难以比拟的。

Flink 写 Paimon 有以下优点：

1. 资源管理与并发控制：

*   无需关注线程管理：Flink 通过其 TaskManager 和 Slot 机制，对任务的并发进行统一、精细化的管理。
    
*   背压机制：Flink 内置了强大的反压机制。当 Paimon 写入变慢时（例如 Compaction 压力大），Flink 能自动感知并将压力向上游传递，减缓数据消费速度，防止因数据堆积导致内存溢出，保证了整个数据链路的稳定性。  
    

2. 状态管理与 Exactly-Once 语义：

*   状态后端：Flink 拥有状态管理能力（其本身就是 Paimon State Backend 的来源）。无论是内存、文件系统还是 RocksDB，Flink 都能高效、可靠地管理算子的状态，这对于需要进行去重、聚合后再写入的复杂场景至关重要。
    
*   一致性保障：结合 Paimon 的 Two-Phase Commit (2PC) Sink 实现，Flink 可以轻松实现端到端的 Exactly-Once 写入语义，确保数据在任何故障情况下都不重不丢。
    

3. 批流处理能力：

*   Paimon 的核心设计理念是 “流批一体的湖存储”。Flink 作为流批一体计算引擎，可以与 Paimon 很好的结合。
    

二、排查工具

在排查问题的过程中，我们使用了很多工具来帮助我们排查问题，接下来是对这些工具的介绍。

**2.1 MAT（Memory Analyzer Tool）**

MAT 是一个高性能、功能丰富的 Java 堆内存分析器。它能帮助开发者和运维工程师在复杂的内存快照（.hprof 文件）中快速定位内存泄漏的根源，并分析应用在特定时刻的内存消耗详情。

一般我们可以通过在机器上生成 Dump 文件，然后通过 MAT 对 Dump 文件进行分析。

MAT 一般有以下功能：

*   泄漏嫌疑报告
    

这是 MAT 的 “一键诊断” 功能。当你打开一个 Dump 文件时，MAT 会自动分析并生成一个报告，直接指出最可疑的内存泄漏点，通常会以饼图展示内存占用最高的几个对象。这是排查的绝佳起点。

*   支配树
    

这是 MAT 最核心、最强大的功能。支配树会重新组织内存中的对象关系，清晰地展示出 “谁支配谁”。如果对象 A 支配对象 B，意味着对象 A 是 B 能存活在内存中的唯一 “看门人”。一旦 A 被回收，B 以及 B 所引用的所有对象都会被回收。

通过查看支配树，我们可以快速找到那些 “支配” 了大量内存的对象，它们通常就是内存泄漏的源头。按“Retained Heap”（保留堆大小）排序，排在最前面的就是最大的嫌疑对象。

*   直方图
    

以类的维度展示内存快照。你可以看到每个类的实例数量、占用的浅堆（Shallow Heap，对象自身大小）和深堆（Retained Heap，对象自身 + 其引用的所有对象总大小）。

*   对象查询语言
    

MAT 提供了一套类似 SQL 的查询语言，允许你对堆内存中的对象执行复杂的查询。

*   GC 根路径分析
    

这是定位内存泄漏的 “铁证”。对于任何一个怀疑是泄漏的对象，可以使用此功能，MAT 会清晰地展示出从该对象到 GC 根（如一个正在运行的线程、一个静态变量等）的完整引用链。这条引用链就是导致该对象无法被回收的原因。

**2.2 NMT（Native Memory Tracking）**

NMT(NativeMemoryTracking) 是一款 JVM 提供的一款 Native Memory 跟踪工具，可以帮助定位由 非 Java 堆内存 引发的内存泄漏或性能问题，例如元空间（Metaspace）、线程栈、JNI 代码、JVM 内部结构等的内存分配。

在我们的项目中加上 -XX:NativeMemoryTracking=detail 参数以后，就可以开启 NMT，但这会带来 5%-10% 的性能损耗。

NMT 的常用指令如下

```
# 查看当前内存概览（summary 模式）
jcmd <PID> VM.native_memory summary
 # 查看详细内存分配记录（detail 模式，需 JVM 启动时启用 detail）
jcmd <PID> VM.native_memory detail
 # 生成基线数据（用于后续对比）
jcmd <PID> VM.native_memory baseline
 # 对比当前内存与基线数据
jcmd <PID> VM.native_memory summary.diff
 # 关闭 NMT（释放资源）
jcmd <PID> VM.native_memory shutdown
```

一般来说，我们可以在应用启动时生成一个 baseline，之后在应用运行一段时间后，采用 diff 命令，对比内存的增长情况。

对比时，我们主要关注 committed 的变化，找到增长较多的区域。

**2.3 Arthas**

arthas 我们都不陌生，一般在我们的业务中，大家用它来查看接口耗时和方法追踪，但实际上，它的功能之强大远超我的想象。

arthas 命令列表

可以看到使用 arthas 可以帮助我们对 jvm、class 等进行分析，帮助我们排查内存问题。

```
# 实时查看 JVM 内存、线程、GC 状态
dashboard

# 查看 JVM 参数和内存池详细信息
jvm

# 查看各内存池（堆、元空间、线程栈等）的使用情况
memory

# 查看某个类的静态变量
ognl '@com.example.CacheManager@cache.size()'
# 强制触发 Full GC
ognl '#runtime=@java.lang.Runtime@getRuntime(), #runtime.gc()'
```

**2.4 async-profiler**

async-profiler 是一个基于 采样 的高性能 Java 分析工具，专注于 CPU 火花、内存分配 和 锁竞争 的低开销分析。它通过 异步信号（如 perf 事件或 itimer） 直接从 JVM 或原生代码中采集堆栈信息，无需修改代码或引入依赖，适合生产环境使用。

arthas 内部也集成了 async-profiler，我们一般可以使用 async-profiler 生成火焰图，帮助我们对内存进行分析。

最新版本的 async-profiler 提供了 native memory 分析能力，通过：

```
# 开启分析
asprof start -e nativemem -f app.jrf <PID>

# 生成火焰图
asprof stop -e nativemem -f app.jrf <PID> > app-leak.html
```

可以对 native memory 进行分析。

**2.5 linux 指令**

常用的内存分析指令如 top、pmap 等，也可以对内存进行分析，尤其是 pamp，可以使用 pmap 对进程的内存块进行观察对比，便于我们找出可以内存地址，再查找其中对象。

三、排查思路总结

对于内存问题的排查，工具千千万万，排查手段也非常多，笔者认为最重要的不是工具如何使用，而是如何找到正确的排查方向。在这里，分享一下我个人的思路，也许不够专业，但还是希望给阅读文章的同学一些参考，在面对内存问题时，不至于无从下手。

**3.1 发生问题，保留现场**

无论什么时候，发生问题一定要记得保留现场，不要因为服务崩溃，就急于重启机器。我们可以保留一台故障机器，用于问题排查 ，因为一旦丢失现场，就相当于侦探破案时犯罪现场被毁，会损失大量线索，极大地阻碍我们排查问题的进度。

但如果现场已经丢失，我们需要想办法复现这个问题，这是帮助我们最快找到问题原因的方法，模拟问题发生时的场景，如加大流量压测，根据日志模拟请求等。

**3.2 查看系统监控，初步定位问题**

公司内部一般会提供应用监控工具，没有的话，我们也应该自己尝试搭建一些简易的监控告警。如内存利用率上涨问题，我们可以观察堆和非堆内存与整体内存利用率上涨趋势的差异，初步定位问题。

**3.3 使用工具，进行分析**

初步定位到问题后，可以使用相关工具进行分析，如果是堆内问题，那么 MAT 就可以很好的帮助我们进行分析，堆外问题的情况相对来说比较复杂，可能需要使用 NMT 和 gperftools 等工具进行分析。

**3.4 阅读文章，请教专家**

大部分情况下，我们都可以找到问题的所在，但也会遇到一筹莫展的时候，这时候我们可以在 Google 上搜索相关文章，也许会有人遇到过和我们相同的问题，也可以在公司内部找技术支持团队帮助，当然，大模型也是帮助我们分析问题的利器，大部分时候，都可以帮我们快速定位到问题。

**3.5 总结问题，记录心得**

每一次问题排查，都需要记录下自己的排查过程，这对于我们个人来说，是帮助我们积累经验，理顺自己每次排查问题的思路，对于其他同学来说，也可以为他们提供经验。

# 参考文章

*   NMT(NativeMemoryTracking) ：https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/tooldescr007.html
    
*   arthas 命令列表：https://arthas.aliyun.com/doc/commands.html#jvm-%E7%9B%B8%E5%85%B3
    

**Kimi K2，开源万亿参数大模型**

先进的混合专家（MoE）语言模型，在前沿知识、推理和编码任务中性能卓越，并优化了工具调用能力。本方案支持云上调用 API 与部署方案，无需编码，最快 5 分钟即可完成，成本最低 0 元。

点击阅读原文查看详情。