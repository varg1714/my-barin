---
source:
  - "[[Caffeine 源码、架构、原理（史上最全，10W 字超级长文）]]"
  - "[[缓存之美：万字长文详解 Caffeine 实现原理]]"
create: 2025-11-05
---

## 1. 引言：为什么需要高性能本地缓存

本地缓存是提升应用性能、降低后端压力的关键手段。它主要应对两类场景：

1. **突发性 Hotkey 场景**：如秒杀、热点新闻，通过在应用本地缓存热点数据，可以有效保护下游的分布式缓存（如 Redis）和数据库，防止因流量突增导致的缓存击穿。
2. **常规性 Hotkey 场景**：如系统配置、组织架构等变化频率低但访问频繁的数据，缓存后可极大提升访问速度。

### 1.1. 本地缓存的优缺点

* **优点**：访问速度极快，因为数据在应用进程内存中，没有网络开销。
* **缺点**：
    * **容量有限**：受限于应用进程的内存大小，无法存储海量数据。
    * **数据一致性**：在集群部署中，需确保各节点本地缓存的同步更新，通常借助消息队列的发布/订阅模式实现，复杂度较高。
    * **生命周期**：数据随应用进程的重启而丢失，不具备持久性。

### 1.2. 核心挑战

* **缓存污染**：无效数据（未来不再被访问）占据了宝贵的缓存空间，挤出了本应保留的热点数据，导致命中率下降。
* **缓存命中率**：衡量缓存性能的核心指标，直接与缓存的**淘汰算法**挂钩。

## 2. Java 本地缓存技术概览

| 技术 | 优点 | 缺点 |
| :--- | :--- | :--- |
| **HashMap** | 简单粗暴，无需引入第三方包。 | 没有缓存淘汰策略，功能简陋。 |
| **Guava Cache** | 支持容量限制、多种过期策略（基于时间）、统计功能。 | 性能被 Caffeine 全面超越，Spring 5.x 后不再支持。 |
| **Caffeine** | **性能接近理论最优**，采用 W-TinyLFU 算法，功能全面。 | 暂无明显缺点。 |
| **Ehcache** | 支持多种淘汰算法（LRU, LFU, FIFO），支持堆外和磁盘缓存，支持集群。 | 性能不如 Caffeine。 |

## 3. 缓存淘汰算法的演进之路：从 LRU/LFU 到 W-TinyLFU

![](https://mmbiz.qpic.cn/mmbiz_png/RQv8vncPm1XhHcDPyPGMl0M4Ah1uf0yyUTVRgia0qtjyRfRibPXSsMv5ViaT6mtoJtyZ2dBRlrJ2IefhnsI1taVUA/640?wx_fmt=png&from=appmsg)


Caffeine 的核心优势在于其先进的淘汰算法，理解其演进过程至关重要。

### 3.1. FIFO (First In First Out - 先进先出)

* **思想**：最先进入缓存的数据最先被淘汰。
* **缺点**：不考虑访问频率和新近度，命中率低，容易将热点数据淘汰。

### 3.2. LRU (Least Recently Used - 最近最少使用)

* **思想**：如果一个数据最近被访问过，那么它将来被访问的概率也更高。优先淘汰最久未被访问的数据。
* **优点**：实现简单，对**突发性、稀疏的流量**（如新热点）表现良好。
* **缺点**：容易被**周期性访问**和**偶然的全量扫描**污染。例如，一次全量数据遍历会把所有真正的周期性热点数据全部淘汰，造成严重的缓存污染。
* **变种**：LRU-K、Two Queues 等算法尝试通过记录历史访问次数来缓解此问题。

### 3.3. LFU (Least Frequently Used - 最近最不经常使用)

* **思想**：如果一个数据在一段时间内使用次数很少，那么它将来被使用的概率也很小。优先淘汰访问频率最低的数据。
* **优点**：能很好地保留**周期性热点数据**，避免被偶然访问所干扰。
* **缺点**：
    * **开销大**：需要为每个缓存项维护一个频率计数器，占用额外空间和更新时间。
    * **历史问题**：一个曾经的热点数据，即使现在不再被访问，其很高的频率计数也会让它“论资排辈”，长期占据缓存，无法被淘汰。
    * **对新热点不友好**：一个新的突发热点，因为初始频率为 0，很容易在频率竞赛中输给历史数据而被立即淘汰。

### 3.4. TinyLFU：对 LFU 的革命性优化

TinyLFU 旨在解决 LFU 的上述三大痛点。

* **解决开销问题 -> Count-Min Sketch 算法 (`FrequencySketch`)**：
    * 它是一种概率性数据结构，可以看作是“数值版”的布隆过滤器。它使用 4 个独立的哈希函数将一个 Key 映射到一个 `long[] table` 数组的多个位置，并对这些位置的计数器进行累加。查询时，返回这 4 个计数器中的最小值作为预估频率。
    * **Caffeine 的极致优化**：Caffeine 认为频率达到 15 已经足够热，因此将每个计数器大小压缩到 **4-bit**。这样，一个 `long` (64-bit) 就可以存储 16 个计数器，极大地节省了空间。
* **解决历史问题 -> 降鲜机制 (Freshness Mechanism)**：
    * `FrequencySketch` 内部有一个 `sampleSize` 阈值。当总访问次数达到该阈值时，会触发 `reset()` 方法，将**所有计数器的值减半**。这使得历史热点数据的“权重”会随着时间的推移而衰减，为新的热点数据腾出机会，解决了 LFU 的历史问题。

### 3.5. W-TinyLFU：Caffeine 的最终形态，LRU 与 LFU 的完美结合

TinyLFU 解决了 LFU 的问题，但它本身仍然对突发流量（LRU 的强项）不够友好。W-TinyLFU (Window-TinyLFU) 通过引入一个 LRU 窗口来弥补这一缺陷。

* **核心思想**：`W-TinyLFU = LRU (应对突发流量) + TinyLFU (应对周期性流量)`
* **数据结构与 JVM 分代模型的类比**：Caffeine 将主缓存空间（Main Space）划分为考察区（Probation）和保护区（Protected），再加上一个独立的窗口区（Window），形成了类似 JVM 分代模型的结构。

| Caffeine 区域 | 默认占比 | 策略 | 类似 JVM 区域 | 作用 |
| :--- | :--- | :--- | :--- | :--- |
| **Window (窗口区)** | 1% | LRU | **Eden 区** | 所有新数据首先进入，保护突发热点不被立即淘汰。 |
| **Probation (考察区)** | ~20% of Main (19.8%) | LRU | **Survivor 区** | 从 Window 区淘汰下来的数据进入此区域，接受频率考验。 |
| **Protected (保护区)** | ~80% of Main (79.2%) | LRU | **Old Gen (老年代)** | 只有在 Probation 区被再次访问的数据才能晋升至此，是真正的高频热点区。 |

* **数据流转与淘汰 (PK 机制) - 源码视角**：
    1. **进入 (Entry)**：`put` 操作将数据存入 `ConcurrentHashMap`，并向 `WriteBuffer` 提交一个 `AddTask`。后台维护线程执行此任务，将新节点放入 **Window** 区 (`accessOrderWindowDeque`) 的队尾。
    2. **降级 (Window -> Probation)**：当 `maintenance` 循环中的 `evictEntries` 检测到 Window 区大小超过其最大值时，会调用 `evictFromWindow`，根据 LRU 规则将队首（最老）的节点（`candidate`）移出，放入 **Probation** 区 (`accessOrderProbationDeque`) 的队尾。
    3. **PK 对决**：当缓存总大小超限，`evictFromMain` 方法被触发。它会从 Probation 区的队首取出最老的数据作为“牺牲者” (`victim`)，与来自 Window 区的 `candidate` 进行频率 PK。
    4. **PK 规则**：调用 `admit(candidateKey, victimKey)` 方法，内部使用 `frequencySketch` 比较二者频率。**频率低者被彻底淘汰** (`evictEntry`)，频率高者留下。如果 `candidate` 频率不高但大于等于一个阈值，会有极小概率随机接受它，以防止 DoS 攻击。
    5. **晋升 (Probation -> Protected)**：当 Probation 区中的一个数据被再次访问（读或写），会触发 `onAccess` 逻辑，调用 `reorderProbation` 方法将其**晋升**到 **Protected** 区 (`accessOrderProtectedDeque`) 的队尾。
    6. **降级 (Protected -> Probation)**：当 Protected 区满时（通常在 `climb` 调整分区大小后），`demoteFromMainProtected` 方法会将 Protected 区队首（最老）的数据**降级**回 **Probation** 区的队尾，重新接受考验。
* **自适应调整 (Hill Climbing)**：在 `maintenance` 循环的最后，`climb()` 方法会执行爬山算法。它通过 `determineAdjustment` 计算近期缓存命中率的变化，动态调整 Window 区和 Protected 区的比例（通过 `increaseWindow` / `decreaseWindow`），以自动适应当前应用的负载特性，实现最优的缓存配置。

## 4. Caffeine 核心特性与使用实操

### 4.1. 填充 (Population)

指当缓存中不存在某个 Key 时，如何加载数据。

#### 4.1.1. 手动 (Manual)

```java
Cache<String, Integer> cache = Caffeine.newBuilder().build();

// 如果不存在，返回 null
Integer age1 = cache.getIfPresent("张三"); 

// 如果 key 不存在，则同步调用 lambda 表达式加载数据，并放入缓存后返回
Integer age2 = cache.get("张三", k -> {
    System.out.println("加载数据 for key: " + k);
    return 18; // 从数据库或其他数据源加载
});
```

#### 4.1.2. 自动 (Loading)

通过在 `build` 方法中提供一个 `CacheLoader`，实现自动加载。

```java
LoadingCache<String, Integer> cache = Caffeine.newBuilder()
    .build(key -> {
        System.out.println("自动加载数据 for key: " + key);
        return 18;
    });

// key 不存在时，会自动调用 CacheLoader 的 load 方法
Integer age = cache.get("张三");
```

#### 4.1.3. 异步 (Asynchronous)

对于加载耗时较长的场景，可以使用异步加载，避免阻塞调用线程。

```java
AsyncLoadingCache<String, Integer> cache = Caffeine.newBuilder()
    .buildAsync(key -> {
        // 模拟耗时操作
        return 18;
    });

// get 方法立即返回一个 CompletableFuture
CompletableFuture<Integer> ageFuture = cache.get("张三");

// 在需要结果时再阻塞获取
Integer age = ageFuture.get();
```

### 4.2. 驱逐策略 (Eviction)

#### 4.2.1. 基于大小 (Size-based)

可以基于条目数量或权重进行驱逐。

```java
// 基于数量，最多10000条
LoadingCache<Key, Graph> cacheByCount = Caffeine.newBuilder()
    .maximumSize(10_000)
    .build(key -> createExpensiveGraph(key));

// 基于权重，总权重不超过10000
// weigher 用来计算每个 value 的权重
LoadingCache<Key, Graph> cacheByWeight = Caffeine.newBuilder()
    .maximumWeight(10_000)
    .weigher((Key key, Graph graph) -> graph.vertices().size())
    .build(key -> createExpensiveGraph(key));
```

**注意**：`maximumSize` 和 `maximumWeight` 不能同时使用。

#### 4.2.2. 基于时间 (Time-based)

```java
// 访问后5分钟过期
Cache<Key, Graph> expireAfterAccessCache = Caffeine.newBuilder()
    .expireAfterAccess(5, TimeUnit.MINUTES)
    .build();

// 写入后10分钟过期
Cache<Key, Graph> expireAfterWriteCache = Caffeine.newBuilder()
    .expireAfterWrite(10, TimeUnit.MINUTES)
    .build();

// 自定义可变过期时间
Cache<Key, Graph> expireAfterCache = Caffeine.newBuilder()
    .expireAfter(new Expiry<Key, Graph>() {
      @Override
      public long expireAfterCreate(Key key, Graph graph, long currentTime) {
        // 创建后5小时过期
        return TimeUnit.HOURS.toNanos(5);
      }
      @Override
      public long expireAfterUpdate(Key key, Graph graph, long currentTime, long currentDuration) {
        // 更新后保持原有过期时间不变
        return currentDuration;
      }
      @Override
      public long expireAfterRead(Key key, Graph graph, long currentTime, long currentDuration) {
        // 读取后保持原有过期时间不变
        return currentDuration;
      }
    })
    .build();
```

#### 4.2.3. 基于引用 (Reference-based)

利用 Java 的垃圾回收机制来驱逐缓存。

| 引用类型 | 被垃圾回收时间 | 用途 |
| :--- | :--- | :--- |
| **强引用** | 从不回收（除非不可达） | 对象的一般状态 |
| **软引用** | 内存不足时回收 | 适合做内存敏感的缓存 |
| **弱引用** | 下一次 GC 时回收 | 适合做生命周期不确定的缓存 |
| **虚引用** | GC 时回收，必须与引用队列联用 | 主要用于跟踪对象回收，如管理堆外内存 |

```java
// 当 Key 没有其他强引用时，条目可能被回收
LoadingCache<Key, Graph> weakKeysCache = Caffeine.newBuilder()
    .weakKeys()
    .build(key -> createExpensiveGraph(key));

// 当 Value 没有其他强引用时，条目可能被回收
LoadingCache<Key, Graph> weakValuesCache = Caffeine.newBuilder()
    .weakValues()
    .build(key -> createExpensiveGraph(key));

// 当内存不足时，GC 会回收软引用的 Value
LoadingCache<Key, Graph> softValuesCache = Caffeine.newBuilder()
    .softValues()
    .build(key -> createExpensiveGraph(key));
```

**注意**：`weakValues` 和 `softValues` 不能同时使用。`AsyncLoadingCache` 不支持引用驱逐。

### 4.3. 刷新策略 (Refresh)

`refreshAfterWrite` 会在写入一段时间后，当条目被再次访问时，**异步**地重新加载它。在加载完成前，访问会返回旧值。

```java
LoadingCache<String, String> cache = Caffeine.newBuilder()
   .refreshAfterWrite(1, TimeUnit.MINUTES)
   .build(key -> loadFromDatabase(key));
```

这与 `expireAfterWrite` 不同，后者是到期后直接驱逐，下次访问会阻塞并同步加载。

### 4.4. 移除监听器 (Removal Listener)

可以在缓存条目被移除（因驱逐或手动删除）时执行一个异步回调。

```java
Cache<Key, Graph> cache = Caffeine.newBuilder()
    .removalListener((Key key, Graph graph, RemovalCause cause) ->
        System.out.printf("Key %s was removed (%s)%n", key, cause))
    .build();
```

### 4.5. 写入器 (CacheWriter)

实现 `write-through` (直写) 或 `write-back` (回写) 模式，将缓存操作同步到底层数据源。

```java
LoadingCache<Key, Graph> cache = Caffeine.newBuilder()
  .writer(new CacheWriter<Key, Graph>() {
    @Override public void write(Key key, Graph graph) {
      // 同步写入到数据库或二级缓存
      storage.save(key, graph);
    }
    @Override public void delete(Key key, Graph graph, RemovalCause cause) {
      // 从数据库或二级缓存中删除
      storage.delete(key);
    }
  })
  .build(key -> createExpensiveGraph(key));
```

## 5. 高性能并发架构：异步化设计的精髓

Caffeine 的性能远超同类，其秘诀在于**彻底的异步化设计**，将用户操作与内部维护解耦。

### 5.1. 设计哲学：先办事，后记账 (类 Write-Ahead Logging)

* **传统方式 (同步)**：`get()` 操作需要加锁，完成数据查找和 LRU 链表更新后才能返回。高并发下锁竞争激烈。
* **Caffeine 方式 (异步)**：`get()` 或 `put()` 操作只负责在 `ConcurrentHashMap` 中进行快速的读写并立即返回，同时将一个“访问/写入事件”提交到一个**缓冲区**。所有耗时的维护操作（更新队列、计算频率、执行淘汰）都由后台线程**异步、批量**地处理。

### 5.2. 核心组件：读写缓冲区 (`ReadBuffer` & `WriteBuffer`)

这是实现异步化的关键。

* **`ReadBuffer` (读缓冲) - 为极致吞吐而生**
    * **用途**：记录高频的**读事件**。
    * **实现**：**Striped Ring Buffer (条带化环形缓冲区)**。
        * **Striped (条带化)**：内部是一个 `Buffer[] table` 数组。每个线程根据自己独有的 `threadLocalRandomProbe` 值哈希到一个专属的 `Buffer` 进行写入，极大减少了线程间的写入冲突。如果发生冲突或 `Buffer` 不存在，`expandOrRetry` 逻辑会尝试初始化、扩容或线性探测下一个槽位。
        * **Ring Buffer (环形缓冲区)**：每个 `Buffer` 都是一个固定大小的无锁数组队列，通过 CAS 操作 `readCounter` 和 `writeCounter` 头尾指针，并用 `& MASK` 实现环形访问，避免了动态创建节点，减少了 GC 压力。
    * **并发控制**：**乐观 CAS 竞赛**。多个线程竞争同一个 `Buffer` 时，通过一次性的 CAS 操作来“预定”写入位置。成功者写入，**失败者立即放弃**。
    * **关键特性**：**有损 (Lossy)**。允许丢失读事件。因为丢失一次访问记录对淘汰策略的精确度影响微乎其微，但避免了线程阻塞或自旋，换来了极致的低延迟和高吞吐。
* **`WriteBuffer` (写缓冲) - 为数据一致性负责**
    * **用途**：记录**写/更新/删除事件**，这些事件绝不能丢失。
    * **实现**：**MpscGrowableArrayQueue (多生产者单消费者可增长队列)**。
    * **并发控制**：生产者（用户线程）通过 CAS 写入。因为只有一个消费者（后台维护线程），所以消费端**无需任何并发控制**。
    * **关键特性**：**无损 (Lossless)**。如果队列已满，它会**自动扩容**。扩容时，它会创建一个更大的新数组，并通过一个特殊的 `JUMP` 对象将新旧数组链接起来，确保消费者可以平滑地从旧数组过渡到新数组，不会丢失任何数据。

### 5.3. 调度核心：`maintenance` 维护循环

* **角色**：缓冲区的**唯一消费者**。
* **触发**：当缓冲区累积到一定量或特定操作（如 `writeBuffer.offer` 失败）触发时，会通过一个状态机（`IDLE`, `REQUIRED`, `PROCESSING`）来决策是否向 `Executor` (默认为 `ForkJoinPool.commonPool()`) 提交一个可复用的 `PerformCleanupTask` 任务。
* **流程**：这是一个低频、批量的加锁过程，所有维护操作都由 `evictionLock` 保护，确保单线程执行，开销被摊销。
    1. `drainReadBuffer()`: 排空读缓冲，对每个节点执行 `onAccess`，更新频率和 LRU 顺序。
    2. `drainWriteBuffer()`: 排空写缓冲，执行 `AddTask` / `UpdateTask`，将节点放入相应队列。
    3. `expireEntries()`: 处理过期条目。
    4. `evictEntries()`: 执行 W-TinyLFU 淘汰逻辑（PK 对决）。
    5. `climb()`: 执行自适应调整（爬山算法）。

### 5.4. 动态类生成与底层优化

![](https://mmbiz.qpic.cn/mmbiz_png/RQv8vncPm1XhHcDPyPGMl0M4Ah1uf0yyQ0TzaQ1GVLLVy0mfaUjaLyFTlLJhndyGqpFbrnibn1eCIs0vNSKxSYw/640?wx_fmt=png&from=appmsg)

* **动态类生成**：Caffeine 的一个独特之处在于它会根据你的配置在运行时动态生成并加载最优的缓存实现类。例如，一个配置了强引用 Key/Value 和基于容量驱逐的缓存，会生成一个名为 `SSMS` 的类。这通过 `LocalCacheFactory` 实现，它拼接字符串（如 `S` for Strong, `M` for Maximum size, `A` for expireAfterAccess）来确定类名，然后反射创建实例。节点类（如 `PSMS`）也采用同样机制。这最大限度地重用了代码，并避免了在实例中包含不必要的字段和逻辑。
* **`TimerWheel` (分层时间轮)**：用于高效实现 `expireAfter` (自定义可变过期时间)。它是一种 O(1) 复杂度的定时器，远优于 `DelayQueue` 的 O(log n)。
* **底层优化**：
    * **`VarHandle`**：使用 JDK 9+ 的安全、高效的原子操作 API，替代 `Unsafe`。
    * **缓存行填充**：在 `MpscGrowableArrayQueue` 和 `RingBuffer` 等并发数据结构中，在关键并发变量（如 `producerIndex` 和 `consumerIndex`）周围填充大量无用字节，确保它们位于不同的 CPU 缓存行（Cache Line）上，避免了多核 CPU 下因一个线程修改数据导致另一线程的缓存行失效的“伪共享”问题，榨干硬件性能。

## 6. 性能对比

以下是文章中提供的基准测试数据，展示了 Caffeine 在不同场景下的吞吐量优势（单位：ops/s，越高越好）。

### 6.1. 读 (100%) - 8 线程

| Bounded Cache | ops/s |
| :--- | :--- |
| **Caffeine** | **181,703,298** |
| ConcurrentLinkedHashMap | 154,771,582 |
| Guava (64) | 24,533,922 |
| Ehcache2_Lru | 11,252,172 |

### 6.2. 读 (75%) / 写 (25%) - 8 线程

| Bounded Cache | ops/s |
| :--- | :--- |
| **Caffeine** | **144,193,725** |
| ConcurrentLinkedHashMap | 63,968,369 |
| Guava (64) | 22,782,431 |
| Ehcache2_Lru | 9,472,810 |

### 6.3. 写 (100%) - 8 线程

| Bounded Cache | ops/s |
| :--- | :--- |
| **Caffeine** | **55,281,751** |
| ConcurrentLinkedHashMap | 23,819,597 |
| Guava (64) | 8,128,024 |
| Ehcache2_Lru | 4,205,936 |
