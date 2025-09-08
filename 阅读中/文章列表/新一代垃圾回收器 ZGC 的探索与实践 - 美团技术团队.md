---
source: https://tech.meituan.com/2020/08/06/new-zgc-practice-in-meituan.html
create: 2025-07-11 22:30
read: true
knowledge: true
knowledge-date: 2025-08-27
summary: "[[阅读中/阅读总结/新一代垃圾回收器 ZGC 的探索与实践 - 美团技术团队|新一代垃圾回收器 ZGC 的探索与实践 - 美团技术团队]]"
tags:
  - JVM
  - Java
---
[ZGC](https://wiki.openjdk.java.net/display/zgc/Main)（The Z Garbage Collector）是 JDK 11 中推出的一款低延迟垃圾回收器，它的设计目标包括：

*   停顿时间不超过 10ms；
*   停顿时间不会随着堆的大小，或者活跃对象的大小而增加；
*   支持 8MB~4TB 级别的堆（未来支持 16TB）。

从设计目标来看，我们知道 ZGC 适用于大内存低延迟服务的内存管理和回收。本文主要介绍 ZGC 在低延时场景中的应用和卓越表现，文章内容主要分为四部分：

*   **GC 之痛**：介绍实际业务中遇到的 GC 痛点，并分析 CMS 收集器和 G1 收集器停顿时间瓶颈；
*   **ZGC 原理**：分析 ZGC 停顿时间比 G1 或 CMS 更短的本质原因，以及背后的技术原理；
*   **ZGC 调优实践**：重点分享对 ZGC 调优的理解，并分析若干个实际调优案例；
*   **升级 ZGC 效果**：展示在生产环境应用 ZGC 取得的效果。

## GC 之痛

很多低延迟高可用 Java 服务的系统可用性经常受 GC 停顿的困扰。GC 停顿指垃圾回收期间 STW（Stop The World），当 STW 时，所有应用线程停止活动，等待 GC 停顿结束。以美团风控服务为例，部分上游业务要求风控服务 65ms 内返回结果，并且可用性要达到 99.99%。但因为 GC 停顿，我们未能达到上述可用性目标。当时使用的是 CMS 垃圾回收器，单次 Young GC 40ms，一分钟 10 次，接口平均响应时间 30ms。通过计算可知，有（40ms + 30ms) * 10 次 / 60000ms = 1.12% 的请求的响应时间会增加 0 ~ 40ms 不等，其中 30ms * 10 次 / 60000ms = 0.5% 的请求响应时间会增加 40ms。可见，GC 停顿对响应时间的影响较大。为了降低 GC 停顿对系统可用性的影响，我们从降低单次 GC 时间和降低 GC 频率两个角度出发进行了调优，还测试过 G1 垃圾回收器，但这三项措施均未能降低 GC 对服务可用性的影响。

### CMS 与 G1 停顿时间瓶颈

在介绍 ZGC 之前，首先回顾一下 CMS 和 G1 的 GC 过程以及停顿时间的瓶颈。CMS 新生代的 Young GC、G1 和 ZGC 都基于标记 - 复制算法，但算法具体实现的不同就导致了巨大的性能差异。

标记 - 复制算法应用在 CMS 新生代（ParNew 是 CMS 默认的新生代垃圾回收器）和 G1 垃圾回收器中。标记 - 复制算法可以分为三个阶段：

*   标记阶段，即从 GC Roots 集合开始，标记活跃对象；
*   转移阶段，即把活跃对象复制到新的内存地址上；
*   重定位阶段，因为转移导致对象的地址发生了变化，在重定位阶段，所有指向对象旧地址的指针都要调整到对象新的地址上。

下面以 G1 为例，通过 G1 中标记 - 复制算法过程（G1 的 Young GC 和 Mixed GC 均采用该算法），分析 G1 停顿耗时的主要瓶颈。G1 垃圾回收周期如下图所示：

![](https://p0.meituan.net/travelcube/2f56a9a249bc8d74f4f455782abce6be147997.png@1832w_848h_80q)

G1 的混合回收过程可以分为标记阶段、清理阶段和复制阶段。

**标记阶段停顿分析**

*   初始标记阶段：初始标记阶段是指从 GC Roots 出发标记全部直接子节点的过程，该阶段是 STW 的。由于 GC Roots 数量不多，通常该阶段耗时非常短。
*   并发标记阶段：并发标记阶段是指从 GC Roots 开始对堆中对象进行可达性分析，找出存活对象。该阶段是并发的，即应用线程和 GC 线程可以同时活动。并发标记耗时相对长很多，但因为不是 STW，所以我们不太关心该阶段耗时的长短。
*   再标记阶段：重新标记那些在并发标记阶段发生变化的对象。该阶段是 STW 的。

**清理阶段停顿分析**

*   清理阶段清点出有存活对象的分区和没有存活对象的分区，该阶段不会清理垃圾对象，也不会执行存活对象的复制。该阶段是 STW 的。

**复制阶段停顿分析**

*   复制算法中的转移阶段需要分配新内存和复制对象的成员变量。转移阶段是 STW 的，其中内存分配通常耗时非常短，但对象成员变量的复制耗时有可能较长，这是因为复制耗时与存活对象数量与对象复杂度成正比。对象越复杂，复制耗时越长。

四个 STW 过程中，初始标记因为只标记 GC Roots，耗时较短。再标记因为对象数少，耗时也较短。清理阶段因为内存分区数量少，耗时也较短。转移阶段要处理所有存活的对象，耗时会较长。因此，G1 停顿时间的瓶颈主要是标记 - 复制中的转移阶段 STW。为什么转移阶段不能和标记阶段一样并发执行呢？主要是 G1 未能解决转移过程中准确定位对象地址的问题。

G1 的 Young GC 和 CMS 的 Young GC，其标记 - 复制全过程 STW，这里不再详细阐述。

## ZGC 原理

### 全并发的 ZGC

与 CMS 中的 ParNew 和 G1 类似，ZGC 也采用标记 - 复制算法，不过 ZGC 对该算法做了重大改进：ZGC 在标记、转移和重定位阶段几乎都是并发的，这是 ZGC 实现停顿时间小于 10ms 目标的最关键原因。

ZGC 垃圾回收周期如下图所示：

![](https://p0.meituan.net/travelcube/40838f01e4c29cfe5423171f08771ef8156393.png@1812w_940h_80q)

ZGC 只有三个 STW 阶段：**初始标记**，**再标记**，**初始转移**。其中，初始标记和初始转移分别都只需要扫描所有 GC Roots，其处理时间和 GC Roots 的数量成正比，一般情况耗时非常短；再标记阶段 STW 时间很短，最多 1ms，超过 1ms 则再次进入并发标记阶段。即，ZGC 几乎所有暂停都只依赖于 GC Roots 集合大小，停顿时间不会随着堆的大小或者活跃对象的大小而增加。与 ZGC 对比，G1 的转移阶段完全 STW 的，且停顿时间随存活对象的大小增加而增加。

### ZGC 关键技术

ZGC 通过着色指针和读屏障技术，解决了转移过程中准确访问对象的问题，实现了并发转移。大致原理描述如下：并发转移中 “并发” 意味着 GC 线程在转移对象的过程中，应用线程也在不停地访问对象。假设对象发生转移，但对象地址未及时更新，那么应用线程可能访问到旧地址，从而造成错误。而在 ZGC 中，应用线程访问对象将触发 “读屏障”，如果发现对象被移动了，那么“读屏障” 会把读出来的指针更新到对象的新地址上，这样应用线程始终访问的都是对象的新地址。那么，JVM 是如何判断对象被移动过呢？就是利用对象引用的地址，即着色指针。下面介绍着色指针和读屏障技术细节。

**着色指针**

着色指针是一种将信息存储在指针中的技术。

ZGC 仅支持 64 位系统，它把 64 位虚拟地址空间划分为多个子空间，如下图所示：

![](https://p0.meituan.net/travelcube/f620aa44eb0a756467889e64e13ee86338446.png@1568w_322h_80q)

其中，[0~4TB) 对应 Java 堆，[4TB ~ 8TB) 称为 M0 地址空间，[8TB ~ 12TB) 称为 M1 地址空间，[12TB ~ 16TB) 预留未使用，[16TB ~ 20TB) 称为 Remapped 空间。

当应用程序创建对象时，首先在堆空间申请一个虚拟地址，但该虚拟地址并不会映射到真正的物理地址。ZGC 同时会为该对象在 M0、M1 和 Remapped 地址空间分别申请一个虚拟地址，且这三个虚拟地址对应同一个物理地址，但这三个空间在同一时间有且只有一个空间有效。ZGC 之所以设置三个虚拟地址空间，是因为它使用 “空间换时间” 思想，去降低 GC 停顿时间。“空间换时间”中的空间是虚拟空间，而不是真正的物理空间。后续章节将详细介绍这三个空间的切换过程。

与上述地址空间划分相对应，ZGC 实际仅使用 64 位地址空间的第 0~41 位，而第 42~45 位存储元数据，第 47~63 位固定为 0。

![](https://p0.meituan.net/travelcube/507f599016eafffa0b98de7585a1c80b338346.png@2080w_624h_80q)

ZGC 将对象存活信息存储在 42~45 位中，这与传统的垃圾回收并将对象存活信息放在对象头中完全不同。

**读屏障**

读屏障是 JVM 向应用代码插入一小段代码的技术。当应用线程从堆中读取对象引用时，就会执行这段代码。需要注意的是，仅 “从堆中读取对象引用” 才会触发这段代码。

读屏障示例：

```
Object o = obj.FieldA   // 从堆中读取引用，需要加入屏障
<Load barrier>
Object p = o  // 无需加入屏障，因为不是从堆中读取引用
o.dosomething() // 无需加入屏障，因为不是从堆中读取引用
int i =  obj.FieldB  //无需加入屏障，因为不是对象引用
```

ZGC 中读屏障的代码作用：在对象标记和转移过程中，用于确定对象的引用地址是否满足条件，并作出相应动作。

### ZGC 并发处理演示

接下来详细介绍 ZGC 一次垃圾回收周期中地址视图的切换过程：

*   **初始化**：ZGC 初始化之后，整个内存空间的地址视图被设置为 Remapped。程序正常运行，在内存中分配对象，满足一定条件后垃圾回收启动，此时进入标记阶段。
*   **并发标记阶段**：第一次进入标记阶段时视图为 M0，如果对象被 GC 标记线程或者应用线程访问过，那么就将对象的地址视图从 Remapped 调整为 M0。所以，在标记阶段结束之后，对象的地址要么是 M0 视图，要么是 Remapped。如果对象的地址是 M0 视图，那么说明对象是活跃的；如果对象的地址是 Remapped 视图，说明对象是不活跃的。
*   **并发转移阶段**：标记结束后就进入转移阶段，此时地址视图再次被设置为 Remapped。如果对象被 GC 转移线程或者应用线程访问过，那么就将对象的地址视图从 M0 调整为 Remapped。

其实，在标记阶段存在两个地址视图 M0 和 M1，上面的过程显示只用了一个地址视图。之所以设计成两个，是为了区别前一次标记和当前标记。也即，第二次进入并发标记阶段后，地址视图调整为 M1，而非 M0。

着色指针和读屏障技术不仅应用在并发转移阶段，还应用在并发标记阶段：将对象设置为已标记，传统的垃圾回收器需要进行一次内存访问，并将对象存活信息放在对象头中；而在 ZGC 中，只需要设置指针地址的第 42~45 位即可，并且因为是寄存器访问，所以速度比访问内存更快。

![](https://p0.meituan.net/travelcube/a621733099b8fda2a0f38a8859e6a114213563.png@2070w_806h_80q)

## ZGC 调优实践

ZGC 不是 “银弹”，需要根据服务的具体特点进行调优。网络上能搜索到实战经验较少，调优理论需自行摸索，我们在此阶段也耗费了不少时间，最终才达到理想的性能。本文的一个目的是列举一些使用 ZGC 时常见的问题，帮助大家使用 ZGC 提高服务可用性。

### 调优基础知识

**理解 ZGC 重要配置参数**

以我们服务在生产环境中 ZGC 参数配置为例，说明各个参数的作用：

重要参数配置样例：

```
-Xms10G -Xmx10G 
-XX:ReservedCodeCacheSize=256m -XX:InitialCodeCacheSize=256m 
-XX:+UnlockExperimentalVMOptions -XX:+UseZGC 
-XX:ConcGCThreads=2 -XX:ParallelGCThreads=6 
-XX:ZCollectionInterval=120 -XX:ZAllocationSpikeTolerance=5 
-XX:+UnlockDiagnosticVMOptions -XX:-ZProactive 
-Xlog:safepoint,classhisto*=trace,age*,gc*=info:file=/opt/logs/logs/gc-%t.log:time,tid,tags:filecount=5,filesize=50m
```

**-Xms -Xmx**：堆的最大内存和最小内存，这里都设置为 10G，程序的堆内存将保持 10G 不变。 **-XX:ReservedCodeCacheSize -XX:InitialCodeCacheSize**：设置 CodeCache 的大小， JIT 编译的代码都放在 CodeCache 中，一般服务 64m 或 128m 就已经足够。我们的服务因为有一定特殊性，所以设置的较大，后面会详细介绍。 **-XX:+UnlockExperimentalVMOptions -XX:+UseZGC**：启用 ZGC 的配置。 **-XX:ConcGCThreads**：并发回收垃圾的线程。默认是总核数的 12.5%，8 核 CPU 默认是 1。调大后 GC 变快，但会占用程序运行时的 CPU 资源，吞吐会受到影响。 **-XX:ParallelGCThreads**：STW 阶段使用线程数，默认是总核数的 60%。 **-XX:ZCollectionInterval**：ZGC 发生的最小时间间隔，单位秒。 **-XX:ZAllocationSpikeTolerance**：ZGC 触发自适应算法的修正系数，默认 2，数值越大，越早的触发 ZGC。 **-XX:+UnlockDiagnosticVMOptions -XX:-ZProactive**：是否启用主动回收，默认开启，这里的配置表示关闭。 **-Xlog**：设置 GC 日志中的内容、格式、位置以及每个日志的大小。

**理解 ZGC 触发时机**

相比于 CMS 和 G1 的 GC 触发机制，ZGC 的 GC 触发机制有很大不同。ZGC 的核心特点是并发，GC 过程中一直有新的对象产生。如何保证在 GC 完成之前，新产生的对象不会将堆占满，是 ZGC 参数调优的第一大目标。因为在 ZGC 中，当垃圾来不及回收将堆占满时，会导致正在运行的线程停顿，持续时间可能长达秒级之久。

ZGC 有多种 GC 触发机制，总结如下：

*   阻塞内存分配请求触发：当垃圾来不及回收，垃圾将堆占满时，会导致部分线程阻塞。我们应当避免出现这种触发方式。日志中关键字是 “Allocation Stall”。
*   基于分配速率的自适应算法：最主要的 GC 触发方式，其算法原理可简单描述为”ZGC 根据近期的对象分配速率以及 GC 时间，计算出当内存占用达到什么阈值时触发下一次 GC”。自适应算法的详细理论可参考彭成寒《新一代垃圾回收器 ZGC 设计与实现》一书中的内容。通过 ZAllocationSpikeTolerance 参数控制阈值大小，该参数默认 2，数值越大，越早的触发 GC。我们通过调整此参数解决了一些问题。日志中关键字是 “Allocation Rate”。
*   基于固定时间间隔：通过 ZCollectionInterval 控制，适合应对突增流量场景。流量平稳变化时，自适应算法可能在堆使用率达到 95% 以上才触发 GC。流量突增时，自适应算法触发的时机可能会过晚，导致部分线程阻塞。我们通过调整此参数解决流量突增场景的问题，比如定时活动、秒杀等场景。日志中关键字是 “Timer”。
*   主动触发规则：类似于固定间隔规则，但时间间隔不固定，是 ZGC 自行算出来的时机，我们的服务因为已经加了基于固定时间间隔的触发机制，所以通过 - ZProactive 参数将该功能关闭，以免 GC 频繁，影响服务可用性。 日志中关键字是 “Proactive”。
*   预热规则：服务刚启动时出现，一般不需要关注。日志中关键字是 “Warmup”。
*   外部触发：代码中显式调用 System.gc() 触发。 日志中关键字是 “System.gc()”。
*   元数据分配触发：元数据区不足时导致，一般不需要关注。 日志中关键字是 “Metadata GC Threshold”。

**理解 ZGC 日志**

一次完整的 GC 过程，需要注意的点已在图中标出。

![](https://p0.meituan.net/travelcube/33b1feaa5a9fdd44fa6c53a4704d2fc7362864.png@2146w_783h_80q)

注意：该日志过滤了进入安全点的信息。正常情况，在一次 GC 过程中还穿插着进入安全点的操作。

GC 日志中每一行都注明了 GC 过程中的信息，关键信息如下：

*   **Start**：开始 GC，并标明的 GC 触发的原因。上图中触发原因是自适应算法。
*   **Phase-Pause Mark Start**：初始标记，会 STW。
*   **Phase-Pause Mark End**：再次标记，会 STW。
*   **Phase-Pause Relocate Start**：初始转移，会 STW。
*   **Heap 信息**：记录了 GC 过程中 Mark、Relocate 前后的堆大小变化状况。High 和 Low 记录了其中的最大值和最小值，我们一般关注 High 中 Used 的值，如果达到 100%，在 GC 过程中一定存在内存分配不足的情况，需要调整 GC 的触发时机，更早或者更快地进行 GC。
*   **GC 信息统计**：可以定时的打印垃圾收集信息，观察 10 秒内、10 分钟内、10 个小时内，从启动到现在的所有统计信息。利用这些统计信息，可以排查定位一些异常点。

日志中内容较多，关键点已用红线标出，含义较好理解，更详细的解释大家可以自行在网上查阅资料。

![](https://p1.meituan.net/travelcube/af918165262afd59ee8331c014953f381190232.png@2771w_2500h_80q)

**理解 ZGC 停顿原因**

我们在实战过程中共发现了 6 种使程序停顿的场景，分别如下：

*   **GC 时，初始标记**：日志中 Pause Mark Start。
*   **GC 时，再标记**：日志中 Pause Mark End。
*   **GC 时，初始转移**：日志中 Pause Relocate Start。
*   **内存分配阻塞**：当内存不足时线程会阻塞等待 GC 完成，关键字是”Allocation Stall”。

![](https://p0.meituan.net/travelcube/85c67c1ed8e8ab789760df164d62475181355.png@1188w_260h_80q)

*   **安全点**：所有线程进入到安全点后才能进行 GC，ZGC 定期进入安全点判断是否需要 GC。先进入安全点的线程需要等待后进入安全点的线程直到所有线程挂起。
*   **dump 线程、内存**：比如 jstack、jmap 命令。

![](https://p0.meituan.net/travelcube/b9ad0cb5bc55c3d5796f3bb347b3989166875.png@1926w_114h_80q)

![](https://p0.meituan.net/travelcube/a837e745bb412074b550bfcd5d596fc020071.png@1050w_87h_80q)

### 调优案例

我们维护的服务名叫 Zeus，它是美团的规则平台，常用于风控场景中的规则管理。规则运行是基于开源的表达式执行引擎 [Aviator](https://github.com/killme2008/aviator)。Aviator 内部将每一条表达式转化成 Java 的一个类，通过调用该类的接口实现表达式逻辑。

Zeus 服务内的规则数量超过万条，且每台机器每天的请求量几百万。这些客观条件导致 Aviator 生成的类和方法会产生很多的 ClassLoader 和 CodeCache，这些在使用 ZGC 时都成为过 GC 的性能瓶颈。接下来介绍两类调优案例。

**内存分配阻塞，系统停顿可达到秒级**

**案例一：秒杀活动中流量突增，出现性能毛刺**

**日志信息**：对比出现性能毛刺时间点的 GC 日志和业务日志，发现 JVM 停顿了较长时间，且停顿时 GC 日志中有大量的 “Allocation Stall” 日志。

**分析**：这种案例多出现在 “自适应算法” 为主要 GC 触发机制的场景中。ZGC 是一款并发的垃圾回收器，GC 线程和应用线程同时活动，在 GC 过程中，还会产生新的对象。GC 完成之前，新产生的对象将堆占满，那么应用线程可能因为申请内存失败而导致线程阻塞。当秒杀活动开始，大量请求打入系统，但自适应算法计算的 GC 触发间隔较长，导致 GC 触发不及时，引起了内存分配阻塞，导致停顿。

**解决方法：**

（1）开启” 基于固定时间间隔 “的 GC 触发机制：-XX:ZCollectionInterval。比如调整为 5 秒，甚至更短。  
（2）增大修正系数 - XX:ZAllocationSpikeTolerance，更早触发 GC。ZGC 采用正态分布模型预测内存分配速率，模型修正系数 ZAllocationSpikeTolerance 默认值为 2，值越大，越早的触发 GC，Zeus 中所有集群设置的是 5。

**案例二：压测时，流量逐渐增大到一定程度后，出现性能毛刺**

**日志信息**：平均 1 秒 GC 一次，两次 GC 之间几乎没有间隔。

**分析**：GC 触发及时，但内存标记和回收速度过慢，引起内存分配阻塞，导致停顿。

**解决方法**：增大 - XX:ConcGCThreads， 加快并发标记和回收速度。ConcGCThreads 默认值是核数的 1/8，8 核机器，默认值是 1。该参数影响系统吞吐，如果 GC 间隔时间大于 GC 周期，不建议调整该参数。

**GC Roots 数量大，单次 GC 停顿时间长**

**案例三： 单次 GC 停顿时间 30ms，与预期停顿 10ms 左右有较大差距**

**日志信息**：观察 ZGC 日志信息统计，“Pause Roots ClassLoaderDataGraph” 一项耗时较长。

**分析**：dump 内存文件，发现系统中有上万个 ClassLoader 实例。我们知道 ClassLoader 属于 GC Roots 一部分，且 ZGC 停顿时间与 GC Roots 成正比，GC Roots 数量越大，停顿时间越久。再进一步分析，ClassLoader 的类名表明，这些 ClassLoader 均由 Aviator 组件生成。分析 Aviator 源码，发现 Aviator 对每一个表达式新生成类时，会创建一个 ClassLoader，这导致了 ClassLoader 数量巨大的问题。在更高 Aviator 版本中，该问题已经被修复，即仅创建一个 ClassLoader 为所有表达式生成类。

**解决方法**：升级 Aviator 组件版本，避免生成多余的 ClassLoader。

**案例四：服务启动后，运行时间越长，单次 GC 时间越长，重启后恢复**

**日志信息**：观察 ZGC 日志信息统计，“Pause Roots CodeCache” 的耗时会随着服务运行时间逐渐增长。

**分析**：CodeCache 空间用于存放 Java 热点代码的 JIT 编译结果，而 CodeCache 也属于 GC Roots 一部分。通过添加 - XX:+PrintCodeCacheOnCompilation 参数，打印 CodeCache 中的被优化的方法，发现大量的 Aviator 表达式代码。定位到根本原因，每个表达式都是一个类中一个方法。随着运行时间越长，执行次数增加，这些方法会被 JIT 优化编译进入到 Code Cache 中，导致 CodeCache 越来越大。

**解决方法**：JIT 有一些参数配置可以调整 JIT 编译的条件，但对于我们的问题都不太适用。我们最终通过业务优化解决，删除不需要执行的 Aviator 表达式，从而避免了大量 Aviator 方法进入 CodeCache 中。

值得一提的是，我们并不是在所有这些问题都解决后才全量部署所有集群。即使开始有各种各样的毛刺，但计算后发现，有各种问题的 ZGC 也比之前的 CMS 对服务可用性影响小。所以从开始准备使用 ZGC 到全量部署，大概用了 2 周的时间。在之后的 3 个月时间里，我们边做业务需求，边跟进这些问题，最终逐个解决了上述问题，从而使 ZGC 在各个集群上达到了一个更好表现。

## 升级 ZGC 效果

### 延迟降低

TP(Top Percentile) 是一项衡量系统延迟的指标：TP999 表示 99.9% 请求都能被响应的最小耗时；TP99 表示 99% 请求都能被响应的最小耗时。

在 Zeus 服务不同集群中，ZGC 在低延迟（TP999 < 200ms）场景中收益较大：

*   **TP999**：下降 12~142ms，下降幅度 18%~74%。
*   **TP99**：下降 5~28ms，下降幅度 10%~47%。

超低延迟（TP999 <20ms）和高延迟（TP999> 200ms）服务收益不大，原因是这些服务的响应时间瓶颈不是 GC，而是外部依赖的性能。

### 吞吐下降

对吞吐量优先的场景，ZGC 可能并不适合。例如，Zeus 某离线集群原先使用 CMS，升级 ZGC 后，系统吞吐量明显降低。究其原因有二：第一，ZGC 是单代垃圾回收器，而 CMS 是分代垃圾回收器。单代垃圾回收器每次处理的对象更多，更耗费 CPU 资源；第二，ZGC 使用读屏障，读屏障操作需耗费额外的计算资源。

## 总结

ZGC 作为下一代垃圾回收器，性能非常优秀。ZGC 垃圾回收过程几乎全部是并发，实际 STW 停顿时间极短，不到 10ms。这得益于其采用的着色指针和读屏障技术。

Zeus 在升级 JDK 11+ZGC 中，通过将风险和问题分类，然后各个击破，最终顺利实现了升级目标，GC 停顿也几乎不再影响系统可用性。

最后推荐大家升级 ZGC，Zeus 系统因为业务特点，遇到了较多问题，而风控其他团队在升级时都非常顺利。欢迎大家加入 “ZGC 使用交流” 群。

## 参考文献

*   [ZGC 官网](https://wiki.openjdk.java.net/display/zgc/Main)
*   彭成寒.《新一代垃圾回收器 ZGC 设计与实现》. 机械工业出版社, 2019.
*   [从实际案例聊聊 Java 应用的 GC 优化](https://tech.meituan.com/2017/12/29/jvm-optimize.html)
*   [Java Hotspot G1 GC 的一些关键技术](https://tech.meituan.com/2016/09/23/g1.html)

## 附录

### 如何使用新技术

在生产环境升级 JDK 11，使用 ZGC，大家最关心的可能不是效果怎么样，而是这个新版本用的人少，网上实践也少，靠不靠谱，稳不稳定。其次是升级成本会不会很大，万一不成功岂不是白白浪费时间。所以，在使用新技术前，首先要做的是评估收益、成本和风险。

**评估收益**

对于 JDK 这种世界关注的程序，大版本升级所引入的新技术一般已经在理论上经过验证。我们要做的事情就是确定当前系统的瓶颈是否是新版本 JDK 可解决的问题，切忌问题未诊断清楚就采取措施。评估完收益之后再评估成本和风险，收益过大或者过小，其他两项影响权重就会小很多。

以本文开头提到的案例为例，假设 GC 次数不变（10 次 / 分钟)，且单次 GC 时间从 40ms 降低 10ms。通过计算，一分钟内有 100/60000 = 0.17% 的时间在进行 GC，且期间所有请求仅停顿 10ms，GC 期间影响的请求数和因 GC 增加的延迟都有所减少。

**评估成本**

这里主要指升级所需要的人力成本。此项相对比较成熟，根据新技术的使用手册判断改动点。跟做其他项目区别不大，不再具体细说。

在我们的实践中，两周时间完成线上部署，达到安全稳定运行的状态。后续持续迭代 3 个月，根据业务场景对 ZGC 进行了更契合的优化适配。

**评估风险**

升级 JDK 的风险可以分为三类：

*   **兼容性风险**：Java 程序 JAR 包依赖很多，升级 JDK 版本后程序是否能运行起来。例如我们的服务是从 JDK 7 升级到 JDK 11，需要解决较多 JAR 包不兼容的问题。
*   **功能风险**：运行起来后，是否会有一些组件逻辑变更，影响现有功能的逻辑。
*   **性能风险**：功能如果没有问题，性能是否稳定，能稳定的在线上运行。

经过分类后，每类风险的应对转化成了常见的测试问题，不再属于未知风险。风险是指不确定的事情，如果不确定的事情都能转化成可确定的事情，意味着风险已消除。

### 升级 JDK 11

选择 JDK 11，是因为在 JDK 11 中首次支持 ZGC，而且 JDK 11 属于长期支持（Long Term Support，LTS）版本，至少会被维护三年，普通版本（如 JDK 12、JDK 13 和 JDK 14）只有 6 个月的维护周期，不建议使用。

**本地测试环境安装**

从两个源 [OpenJDK](https://jdk.java.net/archive/)和 [OracleJDK](https://www.oracle.com/java/technologies/javase-jdk11-downloads.html) 下载 JDK 11，二个版本的 JDK 主要区别是长时期的免费和付费，短期内都免费。注意 JDK 11 版本中的 ZGC 不支持 Mac OS 系统，在 Mac OS 系统上使用 JDK 11 只能用其他垃圾回收器，如 G1。

**生产环境安装**

升级 JDK 11 不仅仅是升级自己项目的 JDK 版本，还需要编译、发布部署、运行、监控、性能内存分析工具等项目支持。美团内部的实践：

**编译打包**：美团发布系统支持选择 JDK 11 进行编译打包。 **线上运行 & 全量部署**：要求线上机器已安装 JDK11，有 3 种方式：

1. 新申请默认安装 JDK 11 的虚拟机：试用 JDK 11 时可用这种方式；全量部署时，如果新申请机器数量过多，可能没有足够机器资源。 2. 通过手写脚本给存量虚拟机安装 JDK 11：不推荐，业务同学过多参与到运维当中。 3. 使用容器提供的镜像部署功能，在打包镜像时安装 JDK 11：推荐方式，不需要新申请资源。

**监控指标**：主要是 GC 的时间和频率，我们通过美团的 CAT 监控系统支持 ZGC 数据的收集（[CAT 已开源](https://tech.meituan.com/2018/11/01/cat-in-depth-java-application-monitoring.html)）。 **性能内存分析**：线上遇到性能问题时，还需要借助 Profiling 工具，美团的性能诊断优化平台 Scalpel 已支持 JDK 11 的性能内存分析。如果你的公司没有相关工具，推荐使用 JProfier。

**解决组件兼容性**

我们的项目包含二十多万行代码，需要从 JDK 7 升级到 JDK 11，依赖组件众多。虽然看起来升级会比较复杂，但实际只花了两天时间即解决了兼容性问题。具体过程如下：

1. 编译，需要修改 pom 文件中的 build 配置，根据报错作修改，主要有两类：

a. 一些类被删除：比如 “sun.misc.BASE64Encoder”，找到替换类 java.util.Base64 即可。

b. 组件依赖版本不兼容 JDK 11 问题：找到对应依赖组件，搜索最新版本，一般都支持 JDK 11。

2. 编译成功后，启动运行，此时仍有可能组件依赖版本问题，按照编译时的方式处理即可。

升级所修改的依赖：

```
<dependency>
    <groupId>javax.annotation</groupId>
    <artifactId>javax.annotation-api</artifactId>
    <version>1.3.2</version>
</dependency>
<dependency>
    <groupId>javax.validation</groupId>
    <artifactId>validation-api</artifactId>
    <version>2.0.1.Final</version>
</dependency>
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.4</version>
</dependency>
<dependency>
    <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator-parent</artifactId>
    <version>6.0.16.Final</version>
</dependency>
<dependency>
    <groupId>com.sankuai.inf</groupId>
    <artifactId>patriot-sdk</artifactId>
    <version>1.2.1</version>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.9</version>
</dependency>
<dependency>
    <groupId>commons-lang</groupId>
    <artifactId>commons-lang</artifactId>
    <version>2.6</version>
</dependency>
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.39.Final</version>
</dependency>
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
</dependency>
```

JDK 11 已经出来两年，常见的依赖组件都有兼容性版本。但是，如果是公司内部提供的公司级组件，可能会不兼容 JDK 11，需要推动相关组件进行升级。如果对方升级较为困难，可以考虑拆分功能，将依赖这些组件的功能单独部署，继续使用低版本 JDK。随着 JDK11 的卓越性能被大家悉知，相信会有更多团队会用 JDK 11 解决 GC 问题，使用者越多，各个组件升级的动力也会越大。

**验证功能正确性**

通过完备的单测、集成和回归测试，保证功能正确性。

## 作者简介

*   王东，美团信息安全资深工程师。
*   王伟，美团信息安全技术专家。