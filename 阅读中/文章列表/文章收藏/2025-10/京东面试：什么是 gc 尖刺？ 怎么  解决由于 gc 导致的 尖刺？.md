---
source: https://mp.weixin.qq.com/s/jDIPRupM6MM-GGDGaEcEUg
create: 2025-10-12 14:59
read: false
knowledge: false
---
FSAC 未来超级架构师

架构师总动员  
实现架构转型，再无中年危机

## 尼恩说在前面

在 40 岁老架构师 尼恩的**读者交流群** (50+) 中，最近有小伙伴拿到了一线互联网企业如得物、阿里、滴滴、极兔、有赞、希音、百度、网易、美团的面试资格，遇到很多很重要的面试题：

*   什么是 gc 尖刺？ 怎么  解决由于 gc 导致的 尖刺？
    
*   GC 毛刺见过吗， 如何排查？
    

最近有小伙伴在面试京东、  阿里、希音等大厂，又遇到了相关的面试题。

小伙伴 没系统梳理， 支支吾吾的说了几句，面试官不满意， **挂了**。

接下来 尼恩结合互联网上的一些实际案例， 大家做一下系统化、体系化的梳理。  使得大家内力猛增，可以充分展示一下大家雄厚的 “技术肌肉”，**让面试官爱到 “不能自已、口水直流”**，然后实现”offer 直提”。

当然，这道面试题，以及参考答案， 会收入咱们的 《[尼恩 Java 面试宝典 PDF](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247497474&idx=1&sn=54a7b194a72162e9f13695443eabe186&scene=21#wechat_redirect)》V175 版本，供后面的小伙伴参考 ，帮助大家进大厂 / 做架构。

最新《尼恩 架构笔记》《尼恩高并发三部曲》《尼恩 Java 面试宝典》的 PDF，请关注本公众号【技术自由圈】获取，回复：领电子书

## 特别说明：

GC 毛刺 很多的场景。

下面的大厂 案例， 介绍的是 一个场景，这个场景是：   大规模的长命小对象 ， 在 新生代 ygc 频繁复制，导致的 GC 毛刺。

特别说明：

其他的 GC 毛刺 场景 后面尼恩再找一些案例来进行展示。

本文的案例 ，以及其他的 gc 毛刺案例， 都来自互联网，不是尼恩原创 案例。

如果原作者不愿意 尼恩用来作为 学习材料 放在公众号， 可以找尼恩反馈，尼恩立即从本公众号撤下来。

## 一、什么是 gc 尖刺？

GC 尖刺（Garbage Collection Spike） ，有时也被称为 **GC 毛刺或 GC 突刺** ， 并不是某个官方术语，而是线上运维的 “体感” 说法。

大概意思是： 在一条本来平稳的 RT（响应时间）或 CPU 曲线上，突然竖起一根像刺一样的尖峰，持续时间从几十毫秒到几秒不等，看上去很多 突刺。

gc 尖刺 根因 是： 垃圾回收器在某一刻发生了长时间停顿（Stop-The-World，简称 STW）。

由于 **Stop-The-World（STW）** 暂停，导致应用程序 RT（响应时间）或 CPU 曲线上 出现的**突然而显著的峰值**。

简单来说，就是 GC 过程中，JVM 会暂停所有应用线程来执行垃圾回收，如果这次暂停时间过长，就会像路上的突然堵车一样，导致系统性能出现瞬间的 “卡顿”。

GC 尖刺的背后，往往是内存管理不当或垃圾回收器配置不佳。

### 一些典型 GC 尖刺 诱因：

**1、内存分配问题**

*   **短命大对象**：在循环或高频方法中持续创建大对象（如大的数组、集合），这些对象可能迅速占满新生代，导致 Minor GC 频繁，且每次回收耗时增加。更糟的是，如果大对象过早晋升到老年代，还会引发不必要的 Full GC，导致 gc 尖刺。
    
*   **内存泄漏**：由于代码缺陷（如未清理的静态集合、未关闭的资源、`ThreadLocal`使用不当），导致对象无法被回收。老年代内存被无效对象逐渐填满，最终触发长时间停顿的 Full GC，但回收效果甚微，内存使用率居高不下，导致 gc 尖刺。
    
*   **大规模的长命小对象在年轻代复制**： 本文的例子中，出现了  大规模长命小对象（约 500MB），在年轻代的 eden 和 幸存者区来回复制，导致 gc 尖刺。
    

**2、垃圾回收器配置与选择**

*   **堆内存设置不合理**：堆内存过小会导致 GC 频繁发生；堆内存过大则会使单次 GC 需要处理的数据量增多，可能导致 STW 时间变长。
    
*   **GC 参数不匹配**：例如，G1 垃圾回收器的 `MaxGCPauseMillis`（预期最大停顿时间）设置过小，可能会迫使 GC 更频繁地工作以试图达到目标，反而影响整体吞吐量并可能引发问题。
    
*   **GC 器选择不当**：像 ZGC 和 ShenandoahGC 这类低停顿回收器，虽然 STW 时间极短，但在高吞吐量计算密集型场景下，其并发执行会与业务线程竞争 CPU 资源，可能导致整体响应时间上升和周期性尖刺。
    

**3、其他问题：如系统资源与外部因素**

*   **日志打印过量**：大量同步日志写入会争抢磁盘 I/O 锁，导致线程阻塞。同时，日志文件快速增大触发的滚动清理操作也会消耗大量 CPU 和 I/O 资源，间接引发或加剧 GC 压力。
    
*   **定时任务处理大数据集**：定时任务一次性加载和处理大量数据（如从数据库捞出数十万条记录），会在短时间内产生海量对象，给 GC 带来巨大压力。
    

### GC 尖刺的危害：

GC 尖刺的危害是直接且严重的，尤其在高并发、低延迟要求的系统中：

*   **接口响应时间剧烈抖动**：最直接的表现就是应用服务的 P99、P999 延迟（如 99% 或 99.9% 请求的响应时间）出现周期性或突发性的尖峰，导致用户体验下降。
    
*   **系统吞吐量下降**：频繁且长时间的 GC 会占用大量系统资源（CPU 资源被大量用于垃圾回收而非业务处理），导致系统整体处理能力（QPS/TPS）降低。
    
*   **上游调用超时与故障扩散**：若 GC 导致服务响应超时，可能引起上游调用方（如网关、其他微服务）连锁超时失败，在分布式系统中可能引发雪崩效应。
    

## 二、问题复盘

在高并发、低延迟的服务中，GC 的行为会直接影响服务的响应时间和稳定性。

本文场景  讨论的场景， 是 源于一个真实的大厂 高并发系统（系统 A），该系统的 QPS 日常在十万级别，大促期间甚至会超过 40W，且对响应时间有毫秒级的严格要求。

任何由于 GC 垃圾回收引起的停顿， 都可能导致超时和业务成功率下降。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP4IuicHhJ07Jbf2oV1ibVsicz414mjpFicwMAlPLlo0qkUicQQ3HTyMY8iciabv2cjJR7P2ZFqicz1IIYOIQ/640?from=appmsg&watermark=1#imgIndex=1)

在大促期间（QPS 40W），的巡检监控中，发现上游调用方出现零星超时告警。

通过监控系统定位到系统 A 在特定时段出现了周期性响应时间毛刺（如下图所示），这些毛刺与 GC 日志中的 Full GC 时间点高度吻合，初步判断是 GC 停顿引发了服务抖动。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP4IuicHhJ07Jbf2oV1ibVsiczjsYfackVKF4nHUQuunS7bsh4lIf4oZKic94aRCyWTORpcUDEYEUfHXg/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=2)

**问题根因：**大规模 长生命小对象引发新生代复制风暴

先说结论：我们发现系统 A 中缓存了一批业务索引数据，这些数据具有以下特点

*   大规模  小对象：  体积小（每个对象几 KB 至几十 KB）、总量大（约 500MB）
    
*   长生命对象 ：一旦加载，长时间存活（通常贯穿整个服务生命周期）
    
*   在业务逻辑中频繁被使用
    

默认情况下，这些对象会在新生代的 Eden 区创建，由于存活时间长，它们会在 Survivor 区来回复制，直至年龄达到阈值后才被晋升到老年代。

在这个过程中，会产生两方面开销：

**(1) 复制开销：大规模对象在 Survivor 区之间来回复制，CPU 消耗显著；**

**(2) 晋升开销：对象年龄达到阈值时，批量复制到老年代，容易引发停顿。**

尤其是在 Survivor 区空间不足或对象复制频率较高时，Young GC 耗时明显增加，严重时甚至会触发提前晋升或直接进入老年代，引发了 GC 尖刺问题

解决这类问题，大致分为以下三步处理

*   排查问题：定位根因
    
*   分析问题：找到解决方案
    
*   优化过程：解决问题
    

优化的思路：  尽早晋升，也就是  让 大规模的长命小对象（业务索引数据）尽早晋升到老年代,  或者 让索引直接分配到老年代，从而加速 加速索引复制。  当然， 也会考虑 升级 GC ， 升级通过  断流发布 + 主动预热 规避 GC。  接下来和大家一一介绍。

在不加一台机器、不改变流量大小的前提下，系统成功率（抖动时）逐步优化效果为：95% => 98% => 99.5% => 99.995%，保障系统高可用。

下面将详细介绍整个排查和优化过程。

## 三、排查过程

### 1、初步常规分析

首先从上游业务报警入手，发现报错均为同步调用超时（TimeoutException），因此聚焦系统 A 自身状态，开展第一轮排查：

*   对比故障时间点前后流量监控，未见明显峰值，CPU 使用率和系统负载均处于正常水位，可排除流量激增导致过载的可能。
    
*   系统 A 为纯内存计算型服务，无数据库、缓存或 RPC 调用，不存在外部组件拖慢整体响应的因素。
    
*   系统虽高并发，但请求间无同步互斥逻辑，不存在分布式锁或线程锁竞争导致的阻塞超时。
    

经过首轮排查，已排除流量激增、外部服务有瓶颈、并发锁等可能影响因素，但并未定位到根因，需进一步向内挖掘系统自身状态。

### 2、定位根因

在排除常规疑点后，我们开始查看系统内部日志与监控，发现**关键日志证据**：发现系统在抖动发生前执行了一次索引发布（热更新），如下图所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP4IuicHhJ07Jbf2oV1ibVsiczAgNibOKYyQBib7sVvjBibqOdicFxNYib9PoXicuecbYOCngfJ5M6U8cOVLIg/640?wx_fmt=png&from=appmsg#imgIndex=3)

说明：该系统每隔 15 分钟会全量替换内存中的业务索引（一个约 500MB 的复杂 Map 结构），此过程瞬间产生大量新对象。

检查对应时间点的 GC 日志，发现 Young GC 耗时异常，其中 Object Copy 阶段耗时超过 200ms，如下图：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP4IuicHhJ07Jbf2oV1ibVsiczkOEzcEJLGVtiamW5jJXHibicPEJia5H9llWFia5hq8rvzVPnh6cLl0UKQfw/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=4)

Object Copy 是 YGC 的关键阶段：存活对象会从 Eden 区复制到 Survivor 区（或晋升老年代）。大量存活对象导致复制开销陡增，引发 GC 尖刺

**定位根因**：系统在索引发布后，新生成的索引对象在 Young GC 中反复复制，且由于对象数量大、存活时间长，导致 Copy 阶段 STW 过长，业务线程暂停，上游超时增多。

也就是：**大规模长生命对象在新生代频繁复制，引起 GC 停顿放大，最终导致服务超时**。

本系统用到的是 G1，关于 G1 详细内容参考：

*   [美团面试：G1 垃圾回收 底层原理是什么？说说你的调优过程？](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247503490&idx=1&sn=fe8dcd5a67b7bd7b1d5bebecd21a3086&scene=21#wechat_redirect)
    
*   [京东面试： 垃圾回收器 CMS、G1 区别是什么？](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247505279&idx=1&sn=c80bb324794677953fed9c86958b93d3&scene=21#wechat_redirect)
    

## 四、问题的定位与分析

### 1、常规优化思路分析

面对 GC 暂停时间过长的问题，通常有以下几种优化思路：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP4IuicHhJ07Jbf2oV1ibVsiczBa8befF4nsdAvx8bYR1GXZJ27dAkTianGBUMMFTRYUr7627pkZPgosg/640?from=appmsg&watermark=1#imgIndex=5)

然而，在本次场景中，上述常规方案大多难以直接应用或效果有限。

原因如下：

*   首先，经过细致排查，代码层面并未发现明显缺陷，索引结构也已高度压缩，没有进一步优化的空间。同时，受限于业务特性，索引更新机制必须采用全量替换，无法实现增量更新。
    
*   其次，单纯增加机器数量虽然可以通过分流请求来减少单机在 STW 期间影响的请求量，但这本质上是一种规避而非解决，不仅无法从根本上消除 GC 停顿，还会造成资源利用率下降和成本上升。
    
*   此外，堆外内存方案虽然能规避 GC 管理，但需要频繁的序列化和反序列化操作。在高并发访问的场景下，这部分额外开销对延迟的影响无法忽视，与系统所需的毫秒级响应目标相悖。
    

因此，综合评估后，我们决定将优化重点放在 JVM 参数调优上：通过精细调整垃圾回收器的行为模式，优化内存分配和晋升策略，尽可能降低大规模对象复制带来的负面影响，从而在现有架构下保障服务的高可用性。

### 2、GC 日志深度解析与根因推演

基于前期分析，问题的核心在于 YGC 的 Object Copy 阶段：大规模索引对象的复制操作耗时过长，导致 STW 时间增加，进而引发上游请求超时。本节将通过详细分析 GC 日志，还原完整的 GC 行为模式。

当前 JVM 核心参数配置如下：

```
-Xms12g -Xmx12g                  # 堆内存固定为12GB
-XX:MetaspaceSize=512m           # 元空间初始大小512MB
-XX:MaxMetaspaceSize=512m        # 元空间最大限制512MB  
-XX:+UseG1GC                     # 使用G1垃圾回收器
-XX:G1HeapRegionSize=16M         # 设置Region大小为16MB
-XX:MaxGCPauseMillis=100         # 目标最大暂停时间100ms
-XX:InitiatingHeapOccupancyPercent=45 # 老年代占用45%时启动混合GC
-XX:+HeapDumpOnOutOfMemoryError  # OOM时生成堆转储
-XX:MaxDirectMemorySize=1g       # 最大直接内存限制1GB


```

通过内部监控平台 ATP 对 GC 日志进行可视化分析，下图中标出了各 GC 事件的时间点和变化曲线：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP4IuicHhJ07Jbf2oV1ibVsiczYvkaJfROfZuol43FH4P59vkzv2HxxWWq4uap9MYgqwuibLCDGc1ibniaw/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=6)

图中清晰展示了以下关键信息：

**① 蓝色圆点（YGC 事件）**：每个圆点代表一次 Young GC 事件。图中可见大量密集分布的蓝点，表明 YGC 发生频率高且耗时极短（毫秒级），能够迅速完成年轻代垃圾回收——这是高吞吐、低延迟系统的理想表现，符合预期。

**② 粉色折线（堆内存占用）**：该折线反映堆内存使用量的动态变化，呈现规律的锯齿形态——快速上升后骤降。这种模式符合预期：由于系统流量大，请求处理过程中会持续产生大量短期存活的临时对象，使内存占用快速上升；当 Eden 区空间不足时触发 YGC，迅速回收这些对象，使内存占用回落至低点。

**③ 异常蓝点（长耗时 YGC）**：部分蓝点明显远离横轴，表示这些 YGC 的耗时显著高于正常水平。这些点与之前在日志中手动识别出的长耗时记录相符，是导致服务抖动的直接原因，需要重点关注。

**④ 紫色折线（老年代占用）**：该曲线反映老年代内存使用情况。正常情况下，因绝大多数对象在年轻代就被回收，老年代占用率增长缓慢。但值得关注的是，每次长耗时 YGC 出现时，紫色折线都呈现明显的阶梯式跃升，表明此时有大量对象晋升至老年代，这一现象需要重点关注。

进一步观察发现，长耗时 YGC 总是成对出现，且具有 “第一次晋升量少、第二次晋升量多” 的规律，如下图所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP4IuicHhJ07Jbf2oV1ibVsiczrDj05lAKdItzUb3uS3A7sn73iawRqDZjeRwibia61FD2MTsv53Pqeyr2A/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=7)

综合所有线索，可以完整推演出问题发生的过程：

**阶段一**：系统创建新索引对象（约 500MB），这些对象被分配在 Eden 区

**阶段二**：Eden 区空间不足触发第一次 YGC，新索引作为存活对象被复制到 Survivor 区，大量对象复制导致 STW 长达 200ms+

**阶段三**：业务代码完成索引切换，将 GcRoot 指向新索引，同时断开旧索引引用

**阶段四**：再次触发 YGC，新索引从 Survivor 区晋升到老年代，旧索引被回收，再次产生 200ms+ STW

**阶段五**：新索引稳定存在于老年代，后续 YGC 只需处理小对象，恢复毫秒级响应

通过这一分析，我们准确定位了 GC 尖刺的根本原因：大规模长生命周期对象在年轻代经历了两次完整的复制过程（Survivor 区复制和老年代晋升），导致双倍的 STW 停顿时间。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP4IuicHhJ07Jbf2oV1ibVsiczX5Wxu1tm55Ij8iaVgIU0qDpH1VxSPduz8b6oBjicMsjBW2giaVBCGKV5g/640?from=appmsg&watermark=1#imgIndex=8)

## 五、优化过程

在明确了问题根源——每次新建的大索引对象在年轻代中经历多次复制，引发长时间 Young GC 停顿——之后，

我们围绕减少复制次数、降低暂停时间的目标，设计了如下优化方案。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP4IuicHhJ07Jbf2oV1ibVsicz45VlENiahSNuCJMUPsNlP4ib3iaVxo33JgnxWBlYGy5Fwq98FHo9yUSJQ/640?from=appmsg&watermark=1#imgIndex=9)

### 1、策略一：让索引尽早晋升至老年代

默认情况下，新创建的对象在经历一定次数的 Young GC 后才会晋升到老年代。我们的核心思路是**改写这个流程，让大索引以最快路径进入老年代，避免在年轻代中反复复制**。

#### 1.1 调整晋升阈值：MaxTenuringThreshold

`MaxTenuringThreshold`参数用于设定对象在晋升至老年代前，能在年轻代中经历的最大 GC 次数。默认值通常为 15，意味着对象需要在 Survivor 区之间来回拷贝多次，才有可能晋升。

通过分析线上 GC 日志，我们发现 G1GC 对大型索引对象进行了优化（Direct Tenuring）。该索引的实际流转路径为：`Eden → S0 → Old`，仅经历了 2 次复制，而非默认的 15 次。这相当于 JVM 自动将 `MaxTenuringThreshold`动态调整为了 1，其过程如下：

**阶段一**：新索引在 Eden 区创建，年龄（Age）为 0。

**阶段二**：发生第一次 Young GC，索引存活。由于当前年龄（0）小于阈值（1），索引从 Eden 被复制到 S0 区，年龄增长为 1。

**阶段三**：发生第二次 Young GC，索引依然存活。此时年龄（1）等于阈值（1），索引被复制到 Old 区，完成晋升。

我们尝试手动设置 `-XX:MaxTenuringThreshold=1`进行验证，GC 日志证实索引的流转路径仍是 `Eden → S0 → Old`。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP4IuicHhJ07Jbf2oV1ibVsiczJutzpehc8WQVlSW1W1XUkAMQGKF4WfWxkSYDFOC92IxsaIuIbKfQZA/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=10)

能不能进一步的优化？

很容易想到，能否将阈值设置为 0，让索引在第一次 GC 时就直接从 Eden 晋升到 Old，完全跳过 Survivor 区？

这样，复制次数将从 2 次降为 1 次，预计暂停时间可减少近一半。

将参数修改为 `-XX:MaxTenuringThreshold=0`后，流程变为：

**阶段一**：新索引在 Eden 区创建，年龄为 0。

**阶段二**：发生 Young GC，索引存活。由于当前年龄（0）已等于阈值（0），索引被直接复制到 Old 区。

实验结果的 GC 日志显示，Young GC 后年轻代的使用量骤降至接近零，证明大型索引已被直接晋升到老年代，优化生效。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP4IuicHhJ07Jbf2oV1ibVsiczRJQpIHKhINJXQFrsaErJm7LOMJCLVk9VfuuwsXQ8twzhHnbNrtxl6w/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=11)

**优化效果总结：**

此优化在不修改任何业务代码、不增加硬件成本的前提下，通过调整一个 JVM 参数，便将因索引切换导致的长暂停 GC 次数减半。

从系统监控来看，索引切换期间的报错量显著减少，服务成功率从 95% 提升至 98%。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP4IuicHhJ07Jbf2oV1ibVsiczE3NNuBjuEGgMxRrsuNFQrvkrwckPWjdLn3zTb5c6zWhae4It5xyY5w/640?from=appmsg&watermark=1#imgIndex=12)

#### 1.2 其他相关参数实验

在通过 `MaxTenuringThreshold`成功优化后，我们进一步探索了其他能控制对象晋升策略的参数，以寻求更优解或替代方案。

**1） InitialTenuringThreshold**

此参数与 `MaxTenuringThreshold`作用类似，用于设定对象晋升的初始年龄阈值。

实测表明，设置 `-XX:InitialTenuringThreshold=1`同样可以将索引的复制次数从 2 次降为 1 次，优化效果与 `MaxTenuringThreshold=0`类似，都能有效提升系统稳定性。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP4IuicHhJ07Jbf2oV1ibVsiczx8sTveoPIWcCsqh1lfvzYD6UR6Kdicf6IGYU1ckV84pSZ1oWScWCnAg/640?wx_fmt=png&from=appmsg#imgIndex=13)

**2） AlwaysTenure**

这是一个更为极端的参数，其字面含义是 “总是晋升”。

开启后，所有在 Young GC 中存活的对象都会直接晋升到老年代，完全跳过 Survivor 区。

实测设置 `-XX:+AlwaysTenure`后，同样达到了减少一次复制的效果。其对象流转路径可概括为：

```
flowchart LR
    A[Eden] -- YGC / 存活 --> B[Old]


```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP4IuicHhJ07Jbf2oV1ibVsicz0z15Src2tIoTziaEsy8256ClqLbvEDeRqibfuCXWAP2A5nvY2HibIpia2Q/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=14)

**关于 AlwaysTenure 的说明：**

*   **多次 GC 现象：**因为索引对象庞大，而 Eden 区剩余空间有限，其构建过程可能横跨多次 Young GC。每次 GC 都会将已构建完成的部分索引直接晋升至老年代。因此，图中显示经历了 3 次 YGC 才将完整索引搬到老年代，这与 “减少单次晋升的复制次数” 的结论并不矛盾。它只是将原本需要在 2 次 GC 中完成的两次复制，变成了在 3 次 GC 中完成的三次直接晋升（每次复制一次）。
    
*   **设计思想：**`AlwaysTenure`的设计理念是禁用 Survivor 区，仅使用 Eden 和 Old 区。与之相反的参数是 `-XX:+NeverTenure`，它会试图让对象永远留在年轻代，禁用 Old 区。两者都是非常极端的策略，仅适用于特定的业务场景。
    
*   **对老年代的影响：**通常，降低晋升阈值会让更多短期存活的对象进入老年代，从而增加 Full GC 的风险。但我们的业务场景特殊性在于对象存活时间两极分化：RPC 请求产生的临时对象生命周期极短（毫秒级），而索引对象生命周期极长（数十分钟以上）。因此，在 Young GC 时，临时对象早已被回收，而索引对象是唯一需要被晋升的。修改这些参数并不会导致大量本应被回收的短命对象进入老年代，故不会增加 Full GC 的负担。
    

### 2、尝试让索引直接分配至老年代

通过上面调整晋升策略，我们成功地将索引的复制次数从 2 次减少到 1 次。一个更极端的想法随之产生：能否让索引在创建时就直接分配在老年代，实现 0 次复制，从而从根本上避免由复制引起的停顿？

这个理想的分配路径如下图所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP4IuicHhJ07Jbf2oV1ibVsiczsyic0MiaE80uu3ibfKmxM0mVQm18buIQ5UUeTjVlnjfj0d2uMcLxsjYxw/640?from=appmsg&watermark=1#imgIndex=15)

围绕此目标，我们进行了以下两种尝试，但均未取得预期效果。

#### 2.1 尝试一：PretenureSizeThreshold

`PretenureSizeThreshold`是一个经典的 JVM 参数，旨在让大于指定大小的对象直接在老年代分配，以避免在年轻代发生昂贵的复制操作。

遗憾的是，该参数在 G1 垃圾收集器下是不生效的。调整此参数后，通过监控发现系统稳定性指标并无改善，GC 日志也显示索引依然在年轻代中创建和流转。

#### 2.2 尝试二：G1HeapRegionSize 与 Humongous Object

G1GC 有一个内置机制用于处理大对象（Humongous Object）。它将堆划分为多个大小相等的 Region（区域）。当一个对象的大小超过单个 Region 容量的一半时，它就会被视为 Humongous Object，并被直接分配在老年代的特殊区域中。

理论上，通过调整 `-XX:G1HeapRegionSize`可以控制 Region 的大小，从而让我们的索引满足 Humongous Object 的条件。

我们增大了 `G1HeapRegionSize`以确保索引整体大小超过其一半，但优化后系统在索引切换时依然出现抖动。分析 GC 日志，索引的分配路径依然是 `Eden → Survivor → Old`，并未被识别为 Humongous Object。

根本原因在于索引对象的物理结构与逻辑结构上

*   逻辑结构：从业务视角看，我们有一个约 500MB 的 “大索引对象”。
    
*   物理结构：但从 JVM 的内存分配视角看，这个 “大索引” 实际上是由上百万个独立的小对象（如 Map Entry、自定义数据结构等）在程序运行过程中逐个构建而成的。JVM 每次通过 `new`关键字分配的是这些小型个体对象，它们的大小远小于 `G1HeapRegionSize`的一半，因此完全符合在 Eden 区分配的条件。
    

除非是像 `int[] arr = new int[1000000000];`这样，在代码层面明确声明分配的、单一的、巨大的连续数组，JVM 才能在一次分配中就识别出其大小并将其直接作为 Humongous Object 处理。

对于由海量小对象聚合而成的逻辑大对象，无法通过调整标准 JVM 参数让其直接在老年代分配。此优化路径在当前业务代码结构下不可行。

### 3、策略三：加速索引的复制过程

在不改变复制次数的情况下，我们尝试通过调整 GC 相关参数来提升复制速度。

<table><thead><tr><td><span><strong><span leaf="">参数名</span></strong></span></td><td><span><strong><span leaf="">作用</span></strong></span></td><td><span><strong><span leaf="">实测效果</span></strong></span></td></tr></thead><tbody><tr><td><section><span><code><span leaf="">-XX:MaxGCPauseMillis</span></code></span></section></td><td><section><span><span leaf="">设置 G1 的目标最大停顿时间</span></span></section></td><td><section><span><strong><span leaf="">效果不明显</span></strong><span leaf="">。目标停顿仅是期望，无法突破物理限制</span></span></section></td></tr><tr><td><section><span><code><span leaf="">-XX:ParallelGCThreads</span></code></span></section></td><td><section><span><span leaf="">设置 STW 阶段并行 GC 的线程数</span></span></section></td><td><section><span><strong><span leaf="">效果不明显</span></strong><span leaf="">。默认值已接近核心数，优化空间小</span></span></section></td></tr><tr><td><section><span><code><span leaf="">-XX:ConcGCThreads</span></code></span></section></td><td><section><span><span leaf="">设置并发标记阶段的线程数</span></span></section></td><td><section><span><span leaf="">对本问题中的 Young GC 暂停时间</span><strong><span leaf="">无直接影响</span></strong></span></section></td></tr></tbody></table>

由于索引复制本身是一个内存密集型操作，受限于硬件和内存带宽，单纯调整线程数或目标停顿时间收效甚微。

### 4、策略四：低停顿收集器 ZGC

ZGC 详细内容参考：

*   [阿里面试：如何选 GC？ZGC 底层原理是什么？染色指针、转发表 是什么 ？90% 的程序员都答错了](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247505032&idx=1&sn=24bffe9fdff44d710021109fcdc7a9aa&scene=21#wechat_redirect)
    
*   [大厂（转转、携程、京东）都用分代 ZGC，卡顿降低 20 倍，吞吐量提升 4 倍。分代 ZGC 这么牛？底层原理是什么？](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247505075&idx=1&sn=dbf6f4ab6cd846f62f287cd2fe9fe33d&scene=21#wechat_redirect)
    

此前基于 G1GC 的优化都是在传统垃圾回收器的框架内进行修补。无论是 G1 还是更早的 CMS，其核心停顿（STW）根源在于对象移动阶段必须暂停所有应用线程。对于需要移动数百 MB 存活数据的大索引场景，这种停顿几乎是不可避免的。

JDK 11 引入的 ZGC 旨在从根本上解决这一问题。其核心突破在于引入了着色指针（Colored Pointers） 和读屏障（Load Barriers） 机制，实现了并发转移。这意味着 ZGC 可以在应用程序线程正常运行的同时，在后台移动和整理内存中的对象。

其工作原理可简要概括为：

**(1) 当 ZGC 需要移动一个对象时，它开始复制数据，但旧地址依然暂时有效。**

**(2) 任何应用程序线程在访问对象时都会触发 “读屏障”。**

**(3) 读屏障会检查该对象是否正在被移动。如果是，它会自动将指针 “转发” 到对象的新地址（也就是指针自愈），确保应用程序总是访问到正确的数据。**

```
flowchart TD
    A[应用线程访问对象] --> B[触发读屏障]
    B --> C{对象是否正在移动?}
    C -- 是 --> D[由读屏障处理<br>等待完成或转发至新地址]
    C -- 否 --> E[直接访问]
    D --> F[正常读取数据]
    E --> F


```

这与 G1 必须 “停止世界→移动对象→更新所有指针→恢复世界” 的串行化流程形成了鲜明对比，理论上的停顿时间优势巨大。

将应用升级至 JDK 11 并启用 ZGC (`-XX:+UseZGC`) 后，效果立竿见影。服务成功率进一步提升至 **99.5%**。

然而，在索引切换的极短时间内，监控系统依然捕捉到了轻微的响应时间毛刺（RT 尖刺）。分析 ZGC 日志，我们发现其根源并非长时间的 STW，而是一种称为 **“分配停滞（Allocation Stall）”** 的现象。

**Allocation Stall**：当应用程序线程试图分配新对象（如执行 `new`语句），但当前堆内存中已无足够的可用空间时，该线程会被迫暂停（“停滞”），直到 ZGC 的垃圾回收周期完成并释放出足够的内存后，才能继续执行分配操作。可以通俗地理解为：“线程急着要内存，但内存没了，只能停下来等 GC 打扫完房间再继续”。

结合系统监控可以发现，每次索引切换构建约 500MB 新对象时，都会引发一次内存占用的瞬时尖峰，而每一次尖峰都精确对应了一次服务的 RT 毛刺。如下图所示：

ZGC 成功地解决了由对象复制引发的长时间 STW 停顿，这是本次优化中最显著的进步。然而，由于索引构建会产生瞬时巨大的内存分配需求，超出了 ZGC 即时回收的吞吐能力，从而引发了短暂的 Allocation Stall（分配停滞）。这成为了系统在极致性能追求下，剩余的一个微小但可感知的抖动来源。

## 六、追求极致：实现索引无感切换的终极方案

经过一系列 JVM 层面的调优，我们将服务成功率从最初的 95% 提升至 99.5%，成效显著。

*   MaxTenuringThreshold=0：提升至 95%
    
*   升级 ZGC ：提升至 99.5%
    

然而，对于追求极致稳定性的系统而言，剩余的 0.5% 的轻微抖动依然是亟待解决的问题。

究其根本，只要大索引的复制发生在服务接流期间，就存在引发延迟尖刺的风险。最终的解决思路不再是 “优化复制过程”，而是 “让复制在无人感知时发生”。

### 1、思路转变：从优化 GC 到规避 GC

既然 JVM 层面始终避免不了 1 次大索引复制，那能否避其锋芒，新的方案是：进行服务断流，在断流期间主动触发并完成索引的复制晋升过程。待服务重新接流时，年轻代中已无大对象，后续所有 Young GC 都将是毫秒级的快速回收。这样可以根治 GC 尖刺

运维平台提供的灰度断流发布模式为此方案提供了基础：每次只发布一批机器，并在其索引加载和切换期间切断流量，切换完成后再重新接入流量。

然而，仅依靠断流发布并不足够。因为索引的分配不一定会立即触发 YGC——只有在 Eden 区空间不足时才会触发。

为了更清晰地说明单纯依赖 “灰度断流” 发布策略的局限性，我们以一个具体的环境配置为例进行分析：假设 **Eden 区大小为 3GB**，**待加载的新索引约为 1GB**。索引切换前 Eden 区的初始占用情况，将直接决定发布时是否会遇到问题。

**Case 1：初始占用低，隐患潜伏**

索引切换前，Eden 区仅占用了 1GB，剩余空间充足。

加载 1GB 新索引后，Eden 区总占用上升至 2GB，仍未达到 3GB 的容量上限。因此，整个过程不会触发 Young GC。

当服务重新接流后，业务请求产生的对象会迅速占满 Eden 区剩余空间，此时 200ms 长暂停势必导致业务请求超时报错。如下图：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP4IuicHhJ07Jbf2oV1ibVsiczsKZV6jGxCU8ECTrfJ6Ohu81oXkFncT9CQQTTgnzNDibzV3cYLUtcVxg/640?from=appmsg&watermark=1#imgIndex=16)

在此场景下，断流发布没有起到任何规避风险的作用，系统抖动依然会发生。

**Case 2：初始占用高，部分缓解**

索引切换前，Eden 区已占用较高空间，例如 2.5GB。

在加载 1GB 新索引的过程中，Eden 区空间很快被耗尽，从而提前触发了一次 Young GC。这次 GC 会将新索引的一部分（例如 500MB）复制到老年代。

这相当于将一次大的复制操作拆分成两次较小的操作，一定程度上缓解了后续 GC 的停顿时间。但其效果依然不理想：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP4IuicHhJ07Jbf2oV1ibVsiczuTjhvhZYFvElicmvruxXtrqEyjwFBVHIYGH36gh4qxf45iaibFJ6sdGGg/640?from=appmsg&watermark=1#imgIndex=17)

此场景下，断流发布仅能部分缓解问题。复制操作虽被拆分但未消除，服务接流后可能仍会有一次较大的停顿。

通过以上分析可见，单纯依赖断流发布，其效果具有极大的偶然性，严重依赖于发布时 Eden 区的初始状态。我们将 “缓解程度” 定义为在断流期间能提前晋升到老年代的索引比例，那么它与 Eden 区初始占用的关系如下图所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP4IuicHhJ07Jbf2oV1ibVsiczCc6oWQfV43RfiaofDnaEd6MmkKuibSW0vMMYvk7S1Nrnf0raBchDV7zg/640?from=appmsg&watermark=1#imgIndex=18)

如图所示，只有当一个批次的 Eden 初始占用较高（落入右侧红色区域）时，该批次的发布才能获得部分缓解。这意味着整个发布过程的效果是不均匀、不可控的。要实现彻底的、确定的 “无感切换”，必须引入更主动的干预手段。

### 2、终极方案：断流发布 + 主动预热

前述分析表明，依赖系统自然状态是不可靠的。要实现确定的 “无感切换”，必须采取主动干预策略。我们的核心思路是：在可控的断流窗口期内，主动制造一次 “可控的危机”，强制触发一次 Young GC，确保新索引 100% 在此刻被复制到老年代。

类似 “预热” 的思路，每次新索引切换后、重新接流前，主动、快速地在 Eden 区创建大量临时的、短命的“预热对象”，瞬间耗尽 Eden 区的剩余空间。此举必然会立即触发一次 Young GC。

这次 GC 会带来两个确定的结果：

**(1) 回收所有无用的 “预热对象”。**

**(2) 将唯一存活的大型对象——新索引，复制到老年代。**

当服务重新接流时，年轻代（Eden 和 Survivor 区）已是 “空城”，只剩下即将被快速回收的业务小对象，从此再无长停顿之忧。

详细流程如下图所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP4IuicHhJ07Jbf2oV1ibVsiczA7Z1AGUbUC585ibQtwhOzwuvuBqzYfTJSkQoPkcYf5m7p7OnPJt89WQ/640?from=appmsg&watermark=1#imgIndex=19)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP4IuicHhJ07Jbf2oV1ibVsicz54GoicZAyWAy3KFB6LDDUVAYBqI4fv9JqQfvn9jOsO1lFJhpINmroSw/640?from=appmsg&watermark=1#imgIndex=20)

该方案的优势在于，其核心逻辑仅需在关键的索引切换方法中增加一个简短的循环即可实现，无需复杂架构改造。代码如下：

```
public boolean switchIndex(String indexPath){
    try {
        // 1.【断流】加载新索引
        MyIndex newIndex = loadIndex(indexPath);
        // 2.【断流】索引切换
        this.index = newIndex;
        // 3.【断流】Eden 区预热
        for (int i = 0; i < 10000; i++) {
            char[] tempArr = newchar[524288];
        }
        // 4.【断流】通知上层索引切换完成
        return true;
        // 5.【接流】重新接流，此后 YGC 都会很快
    } catch (Exception e) {
        return false;
    }
}


```

**实现注意**：预热对象的大小需精心设计。单个对象应显著大于通常的业务对象，以快速消耗内存，但又必须小于 `G1HeapRegionSize`的一半（通常为 1MB），以避免被 G1GC 直接当作大对象（Humongous Object）分配至老年代，从而绕过年轻代，使预热机制失效。

### 3、效果验证

部署并启用 “主动预热” 方案后，我们通过以下三个维度对优化效果进行了全面验证，结果令人振奋。

**1）GC 日志分析：复制操作被成功前置**

对比优化前后的 GC 日志，变化一目了然。下图所示的优化后日志清晰记录了断流期间发生的完整过程：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/xlgvgPaib7WP4IuicHhJ07Jbf2oV1ibVsicztmQo6Q9wXNfFia4r8F63Ag7yJico0Qt79yAcMveAgbTlSQicicWPmhULWw/640?wx_fmt=png&from=appmsg&watermark=1#imgIndex=21)

关键节点解读：

*   **① 索引加载**：新索引被创建，占据 Eden 区大部分空间。
    
*   **② 主动预热触发 YGC**：预热代码循环创建大量临时对象，迅速耗尽 Eden 区剩余空间，迫使 JVM 立即触发一次 Young GC。
    
*   **③ 索引晋升**：本次 YGC 将新索引（存活对象）完整地复制到老年代，同时回收所有临时预热对象。
    
*   **④ 接流后常态**：服务重新接流后，所有的 Young GC 都只用于回收业务请求产生的瞬时小对象，耗时均下降至毫秒级，变得快速而平稳。
    

**2）系统监控：消失的抖动曲线**

监控系统是最直观的成效证明。下图展示了优化后一段时间内的服务响应时间（RT）监控曲线，红色箭头标注了数次索引切换事件的发生时刻。

可以观察到，在索引切换时，RT 曲线依然平整，几乎没有出现任何毛刺或尖峰。这表明索引切换带来的延迟影响已经被完全控制在断流期内，对线上业务流量做到了真正的 “无感”。

**3） 业务指标：达到极致稳定**

最终，一切优化都体现在业务结果上。如下图所示，经过本轮彻底优化，服务的日常成功率稳定在 **99.995%** 以上，剩余极少数的失败通常源于网络抖动等外部因素，GC 引发的服务抖动问题已被完全根治。

历经从 JVM 参数调优到垃圾收集器升级，最终到 “发布策略 + 主动预热” 的架构与流程优化，我们成功地将一个因固有业务模式（定期加载大索引）而引发的性能瓶颈彻底化解，实现了技术上的极致追求。

## 七、总结

本文针对一个高并发（QPS 10W+）、低延迟（要求毫秒级响应）、高内存压力（每 15 分钟需切换 GB 级索引）的服务系统，因其频繁的索引切换导致的 GC 尖刺和系统抖动问题，进行了一系列从 JVM 层到架构层的深度优化实践。

我们系统地探索了多种解决方案，其效果对比如下：

<table><thead><tr><td><span><strong><span leaf="">优化手段</span></strong></span></td><td><span><strong><span leaf="">服务可用率（抖动时）</span></strong></span></td></tr></thead><tbody><tr><td><section><span><span leaf="">G1GC + 默认参数</span></span></section></td><td><section><span><span leaf="">95%（Baseline）</span></span></section></td></tr><tr><td><section><span><span leaf="">-XX:MaxTenuringThreshold=0</span></span></section></td><td><section><span><span leaf="">98%</span></span></section></td></tr><tr><td><section><span><span leaf="">-XX:InitialTenuringThreshold=1</span></span></section></td><td><section><span><span leaf="">98%</span></span></section></td></tr><tr><td><section><span><span leaf="">-XX:+AlwaysTenure</span></span></section></td><td><section><span><span leaf="">98%</span></span></section></td></tr><tr><td><section><span><span leaf="">ZGC + 默认参数</span></span></section></td><td><section><span><span leaf="">99.5%</span></span></section></td></tr><tr><td><section><span><span leaf="">G1GC + 灰度断流 + Eden 预热</span></span></section></td><td><section><span><span leaf="">99.995%</span></span></section></td></tr></tbody></table>

至此，未来无论系统 QPS 涨到多高、索引体积膨胀到多大、索引切换多么频繁，系统都能无感切换索引，稳定性不再受到任何影响。

## 说在最后：有问题找老架构取经‍

按照此文的套路去回答，一定会 **吊打面试官，让面试官爱到 “不能自已、口水直流”**，然后实现”offer 直提”。

在面试之前，建议大家系统化的刷一波 5000 页《[尼恩 Java 面试宝典 PDF](https://mp.weixin.qq.com/s?__biz=MzkxNzIyMTM1NQ==&mid=2247497474&idx=1&sn=54a7b194a72162e9f13695443eabe186&scene=21#wechat_redirect)》，里边有大量的大厂真题、面试难题、架构难题。

很多小伙伴刷完后， 吊打面试官， 大厂横着走。

在刷题过程中，如果有啥问题，大家可以来 找 40 岁老架构师尼恩交流。

另外，如果没有面试机会， 可以找尼恩来改简历、做帮扶。前段时间，**[指导一个小伙 暴涨 200% ，29 岁 / 7 年 / 双非一本  逆天改命。](https://mp.weixin.qq.com/s?__biz=MzIxMzYwODY3OQ==&mid=2247486433&idx=1&sn=0f630e8fc7620b3bf0e054827b48d91e&scene=21#wechat_redirect)**

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

![](https://mmbiz.qpic.cn/mmbiz_jpg/9qY2Gt3PBs3icE6WcM4nBOdRRcczOW4wkNDgCMZhBfqeet9suwGHfgQduP33YYqqZLOppecMI72Pv5ibNkPeIk6g/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1#imgIndex=22)

关注**职业救助站**公众号，获取每天职业干货  
助您实现**职业转型、职业升级、极速上岸**  
---------------------------------

**技术自由圈**

实现架构转型，再无中年危机

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/xlgvgPaib7WMtyFcJx45oIeWL1d6AENcdoAvuTPuq46E9n8pPe4SHXoSicuEVvhibHRSIzmEBPlC7POcDaqD6ID7w/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1#imgIndex=23)

关注**技术自由圈**公众号，获取每天技术千货  
一起成为牛逼的**未来超级架构师**

**几十篇架构笔记、5000 页面试宝典、20 个技术圣经  
请加尼恩个人微信 免费拿走**

**暗号，请在 公众号后台 发送消息：领电子书**

如有收获，请点击底部的 " 在看 "和" 赞 "，谢谢