---
source:
  - "[[Stream 原理与执行流程探析]]"
  - "[[总结｜Stream 流技术在真实案例中的应用]]"
create: 2025-11-06
---

## 1. Stream 概述

Java Stream 提供了一种声明式、函数式的数据处理方式，能够让代码更简洁、逻辑更清晰。与传统的 `for` 循环相比，Stream API 可以将一系列复杂操作（如筛选、转换、排序、聚合）串联起来，一气呵成。

### 1.1. 优势

1. **代码更简洁**：偏声明式的编码风格，更容易体现出代码的逻辑意图。
2. **逻辑间解耦**：一个 stream 中间处理逻辑，无需关注上游与下游的内容，只需要按约定实现自身逻辑即可。
3. **并行处理**：通过并行流（`parallelStream`）可以轻松利用多核 CPU，提升大数据量下的处理效率。
4. **惰性执行**：中间操作会延迟执行，只有遇到终止操作时才会真正开始处理数据，这可以避免不必要计算，提升性能。

### 1.2. 劣势

1. **Debug 不方便**：由于其链式调用和内部迭代的特性，在 Debug 时追踪某个特定元素的处理过程相对复杂。

## 2. Stream 的核心三大特点

1. **不存储元素 (Stateless Storage):** Stream 本身不是一个数据结构，它不存储任何数据。数据元素实际存储在底层的集合（如 `List`）中，或者在需要时按需生成。Stream 更像是一条流经数据的“管道”。
2. **不修改数据源 (Immutability):** Stream 的所有操作（无论是中间操作还是终结操作）都不会修改原始的数据源。它们会返回一个新的 Stream 或者一个最终结果。这保证了数据源的安全性。
3. **惰性执行 (Lazy Execution):** 这是 Stream 最高效、最关键的特性。中间操作（如 `map`, `filter`）在被调用时，并不会立即执行。它们只是在“搭建流水线”。只有当一个**终结操作**（如 `collect`, `forEach`）被调用时，整个数据处理流程才会真正开始。

## 3. Stream 操作的分类与 API 概览

可以将 Stream 流操作分为 3 种类型：创建 Stream、Stream 中间处理、终止 Steam。

![https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLdtYvRxoZK9s2CVS3IK7CuNdQbh70SMulZ27Zk7DcvcAIXJCDIDpPiavrtXJ60adJydia3jp1mbGsQ/640?wx_fmt=other&from=appmsg](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLdtYvRxoZK9s2CVS3IK7CuNdQbh70SMulZ27Zk7DcvcAIXJCDIDpPiavrtXJ60adJydia3jp1mbGsQ/640?wx_fmt=other&from=appmsg)

### 3.1. 创建操作 (开始管道)

负责新建一个 Stream 流，或者基于现有的集合、数组等创建出新的 Stream 流。

![https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naLdtYvRxoZK9s2CVS3IK7CuhKiaETQpuKJVicHCMy0wXhE3gVleiaDibiayib0j8ItycmKczdQ7Pe5VU2Ow/640?wx_fmt=png&from=appmsg](https://mmbiz.qpic.cn/mmbiz_png/Z6bicxIx5naLdtYvRxoZK9s2CVS3IK7CuhKiaETQpuKJVicHCMy0wXhE3gVleiaDibiayib0j8ItycmKczdQ7Pe5VU2Ow/640?wx_fmt=png&from=appmsg)

### 3.2. 中间操作 (中间管道)

负责对 Stream 进行处理，并返回一个新的 Stream。中间操作可以进行多次链式调用。

| API | 功能说明 |
| :--- | :--- |
| `filter()` | 按照条件过滤符合要求的元素，返回新的 stream 流。 |
| `map()` | 将已有元素转换为另一个对象类型，一对一逻辑，返回新的 stream 流。 |
| `flatMap()` | 将已有元素转换为另一个对象类型，一对多逻辑，即原来一个元素对象可能会转换为 1 个或者多个新类型的元素，返回新的 stream 流。 |
| `limit()` | 仅保留集合前面指定个数的元素，返回新的 stream 流。 |
| `skip()` | 跳过集合前面指定个数的元素，返回新的 stream 流。 |
| `concat()` | 将两个流的数据合并起来为 1 个新的流，返回新的 stream 流。 |
| `distinct()` | 对 Stream 中所有元素进行去重，返回新的 stream 流。 |
| `sorted()` | 对 stream 中所有的元素按照指定规则进行排序，返回新的 stream 流。 |
| `peek()` | 对 stream 流中的每个元素进行逐个遍历处理，返回处理后的 stream 流。主要用于 Debug。 |

### 3.3. 终止操作 (终止管道)

执行终止操作后，Stream 流将会结束，并产生一个最终结果或副作用。一个流只能被消费一次。

| API | 功能说明 |
| :--- | :--- |
| `count()` | 返回 stream 处理后最终的元素个数。 |
| `max()` / `min()` | 返回 stream 处理后的元素最大/最小值。 |
| `findFirst()` | 找到第一个符合条件的元素时则终止流处理。 |
| `findAny()` | 找到任何一个符合条件的元素时则退出流处理，在并行流中效率更高。 |
| `anyMatch()` | 判断是否有任一元素符合条件，返回 boolean。 |
| `allMatch()` | 判断是否所有元素都符合条件，返回 boolean。 |
| `noneMatch()` | 判断是否所有元素都不符合条件，返回 boolean。 |
| `collect()` | 将流转换为指定的集合类型（List, Set, Map 等）或进行其他复杂聚合。 |
| `toArray()` | 将流转换为数组。 |
| `forEach()` | 无返回值，对元素进行逐个遍历，然后执行给定的处理逻辑。 |

## 4. 常用 Stream 方法详解与真实案例

### 4.1. `map` 与 `flatMap`

两者都用于转换元素，但核心区别在于映射关系：

* **`map`**: **一对一**转换。每个输入元素都只转换为一个输出元素。
* **`flatMap`**: **一对多**转换。每个输入元素可以转换为零个、一个或多个输出元素，最终会将所有生成的子流“扁平化”成一个单一的流。

**`map` 用例：** 将字符串 ID 列表转换为对象列表。

```java
List<String> ids = Arrays.asList("205", "105", "308");
List<NormalOfferModel> results = ids.stream()
        .map(id -> {
            NormalOfferModel model = new NormalOfferModel();
            model.setCate1LevelId(id);
            return model;
        })
        .collect(Collectors.toList());
```

**`flatMap` 用例：** 将多个句子中的所有单词提取到一个列表中。

```java
List<String> sentences = Arrays.asList("hello world", "java stream");
List<String> words = sentences.stream()
        .flatMap(sentence -> Arrays.stream(sentence.split(" ")))
        .collect(Collectors.toList());
// 结果: [hello, world, java, stream]
```

![https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLdtYvRxoZK9s2CVS3IK7CuneZ6nvwmDMhtD29lbP0o3AQOvQInia8UgNyzf4uuHbiaPicdlRJyUShgg/640?wx_fmt=other&from=appmsg](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLdtYvRxoZK9s2CVS3IK7CuneZ6nvwmDMhtD29lbP0o3AQOvQInia8UgNyzf4uuHbiaPicdlRJyUShgg/640?wx_fmt=other&from=appmsg)

### 4.2. `peek` 与 `forEach`

两者都可以遍历元素，但用途和性质完全不同：

* **`peek`**: 是一个**中间操作**。它返回一个新的流，主要用于在不改变流内元素的情况下，对元素执行某些操作，最常见的用途是 **Debug**。由于惰性执行，如果 `peek` 后面没有终止操作，它将不会被执行。
* **`forEach`**: 是一个**终止操作**。它没有返回值，用于消费流中的所有元素，执行最终操作。

```java
List<String> sentences = Arrays.asList("hello world", "Jia Gou Wu Dao");

// peek 独自调用时不会执行
System.out.println("----before peek----");
sentences.stream().peek(System.out::println);
System.out.println("----after peek----");

// forEach 会立即执行
System.out.println("----before foreach----");
sentences.stream().forEach(System.out::println);
System.out.println("----after foreach----");

// peek 后面跟终止操作，peek 就会执行
System.out.println("----before peek and count----");
sentences.stream().peek(System.out::println).count();
System.out.println("----after peek and count----");
```

### 4.3. `collect` 结果收集

`collect` 是一个功能极其强大的终止操作，通过 `Collectors` 工具类可以实现多种聚合。

#### 4.3.1. 生成集合对象

```java
// collect成List
List<NormalOfferModel> list = offers.stream().collect(Collectors.toList());

// collect成Set
Set<NormalOfferModel> set = offers.stream().collect(Collectors.toSet());

// collect成Map，key为id，value为对象本身
// 注意：(k1, k2) -> k2 是为了处理 key 重复的情况，保留后者
Map<String, NormalOfferModel> map = offers.stream()
    .collect(Collectors.toMap(NormalOfferModel::getCate1LevelId, Function.identity(), (k1, k2) -> k2));
```

#### 4.3.2. 生成拼接字符串

使用 `Collectors.joining()` 可以优雅地拼接字符串，无需手动处理分隔符。

```java
List<String> ids = Arrays.asList("205", "10", "308");
String joinResult = ids.stream().collect(Collectors.joining(","));
// 结果: "205,10,308"
```

#### 4.3.3. 批量数学运算

```java
List<Integer> numbers = Arrays.asList(10, 20, 30, 40, 50);

// 计算平均值
Double average = numbers.stream().collect(Collectors.averagingInt(value -> value)); // 30.0

// 获取完整的统计信息（数量、总和、最大/最小/平均值）
IntSummaryStatistics summary = numbers.stream().collect(Collectors.summarizingInt(value -> value));
// 结果: IntSummaryStatistics{count=5, sum=150, min=10, average=30.000000, max=50}
```

## 5. Stream 的完整执行流程（三阶段模型）

Stream 的一次完整运行可以清晰地划分为三个阶段：

### 5.1. 阶段一：搭建流水线 (Pipeline Construction)

这个阶段就像是为工厂设计并安装一条装配流水线，但还没有开启电源。

1. **创建源头 (`stream()`):** 创建流水线的第一个节点 `ReferencePipeline$Head`，它持有指向原始数据源的 `Spliterator`。
2. **添加中间操作 (`map`, `filter` 等):** 每调用一个中间操作，就会在流水线末端添加一个新节点，形成一个 `Head -> Map -> Filter` 这样的双向链表结构。此阶段，Lambda 表达式仅被存储，不执行。

### 5.2. 阶段二：反向回溯生成操作实例 (Sink Instantiation)

这是由**终结操作**触发的关键一步，可以理解为给流水线上的每个工位分派一个“工人”（即 `Sink`）。这个过程是**从后往前**的。

1. **触发点：** 调用 `collect()` 等终结操作。
2. **“套娃”过程：**
    * 终点站 (`collect`) 首先创建自己的 `Sink_Collect`。
    * 上一站 (`filter`) 接着创建 `Sink_Filter`，并将 `Sink_Collect` 作为其下游包裹起来。
    * 再上一站 (`map`) 最后创建 `Sink_Map`，并将 `Sink_Filter` 包裹起来。
3. **结果：** 最终形成一个 `Sink_Map(Sink_Filter(Sink_Collect))` 这样的“套娃”结构。

### 5.3. 阶段三：启动流水线 (Pipeline Execution)

一切准备就绪，现在开启电源，让数据真正地流动起来。

1. **遍历数据源：** 开始遍历 `Spliterator`。
2. **垂直处理：** 数据元素是**一个一个地**流过整个流水线的，而不是一批一批地处理。一个元素会依次经过 `map`, `filter`, `collect` 的完整处理，然后才轮到下一个元素。这个过程没有产生任何中间集合。
3. **获取结果：** 所有元素处理完毕后，终结操作从最内层的 `Sink` 中获取最终结果并返回。

## 6. 核心概念深度解析

### 6.1. `Sink` 到底是什么？

可以把它理解为**“处理站”**或**“水槽”**。它是一个封装了单步操作逻辑的对象，通过 `accept(T t)` 方法接收上游数据，处理后调用下游 `downstream.accept()` 传递结果，构成了数据在流水线中的实际流动路径。

### 6.2. Stream 操作分类（按行为和状态）

| 分类维度 | 类型 | 解释 | 示例 |
| :--- | :--- | :--- | :--- |
| **按状态** | **无状态 (Stateless)** | 处理一个元素时，不需要知道其他任何元素的信息。 | `map`, `filter`, `flatMap` |
| | **有状态 (Stateful)** | 需要知道流中其他元素的信息才能进行处理，通常需要一个缓冲区来存储状态。 | `distinct`, `sorted`, `limit` |
| **按行为** | **非短路 (Non-short-circuiting)** | 必须处理流中的**所有**元素才能得到结果。 | `collect`, `forEach`, `count`, `reduce` |
| | **短路 (Short-circuiting)** | **不一定**需要处理所有元素，一旦找到满足条件的结果，就可以**立即停止**。 | `anyMatch`, `allMatch`, `findFirst`, `limit` |

### 6.3. 多个有状态操作的交互与性能考量

当流水线中包含多个有状态操作时，它们的交互方式对性能有巨大影响。

**核心机制：** 有状态操作的 `Sink` 通常会“截断”数据流。它会接收所有上游数据并存入内部的缓冲区，直到数据源耗尽，才在 `end()` 方法被调用时进行最终处理，并将结果一次性向下游“冲刷” (flush)。

**示例：** `stream().distinct().sorted().collect()`

1. **Sink 结构：** `Sink_Distinct( Sink_Sorted( Sink_Collect ) )`
2. **执行过程：**
    * `Sink_Distinct` 内部维护一个 `HashSet` 作为缓冲区。
    * `Sink_Sorted` 内部维护一个 `ArrayList` 作为缓冲区。
    * 当数据元素逐个流经时，`Sink_Distinct` 会去重，并将不重复的元素传递给 `Sink_Sorted`。
    * `Sink_Sorted` 接收到元素后，**并不会立即传递下去**，而是将它们全部存入自己的 `ArrayList` 缓冲区。
    * **关键点：** 在整个数据流处理阶段，`distinct` 的 `HashSet` 和 `sorted` 的 `ArrayList` **同时存在于内存中并不断增长**。
3. **收尾阶段：**
    * 当所有数据源元素处理完毕，`Sink_Sorted` 的 `end()` 方法被触发。
    * 它对自己缓冲区内的所有元素进行排序。
    * 排序完成后，它才将排好序的元素逐个传递给下游的 `Sink_Collect`。

**结论与性能启示：**

* **高内存消耗：** 多个有状态操作串联会非常消耗内存，因为每个操作都需要自己的缓冲区。对于大数据量的流，这可能导致 `OutOfMemoryError`。
* **操作顺序至关重要：**
    * **推荐:** `stream().distinct().sorted()`。先去重，可以有效减少 `sorted` 需要缓存和排序的数据量，性能更好。
    * **不推荐:** `stream().sorted().distinct()`。先排序所有元素（包括重复的），内存消耗更大，计算量也更大。
* **最佳实践：** 谨慎使用有状态操作。如果必须使用多个，请将能有效减少流元素数量的操作（如 `filter`, `distinct`）放在前面。

## 7. 并行 Stream (Parallel Stream)

### 7.1. 机制说明

并行流通过“分而治之”的思想，将一个大任务切分成多个子任务，并利用 JDK 内置的全局 `ForkJoinPool` 线程池来并行执行这些子任务，最后将结果汇总。这可以有效利用计算机的多核 CPU，提升计算密集型任务的执行速度。

![https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLdtYvRxoZK9s2CVS3IK7CuribDy0at5JnFCic7aSrIMfWdnGQReUJObBNv78pO0jz0OfibTl0vxwztg/640?wx_fmt=other&from=appmsg](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLdtYvRxoZK9s2CVS3IK7CuribDy0at5JnFCic7aSrIMfWdnGQReUJObBNv78pO0jz0OfibTl0vxwztg/640?wx_fmt=other&from=appmsg)

### 7.2. 约束与限制

1. **线程安全**：在 `parallelStream()` 中执行的操作（如 `forEach`）必须是线程安全的。
2. **避免使用默认线程池**：对于关键或耗时任务，不建议直接使用 `parallelStream()`。因为它会与 GC 等其他系统任务共用全局 `ForkJoinPool`，可能导致整个应用性能下降。推荐使用自定义线程池。
3. **避免耗时操作**：并行流不适用于包含大量 IO 操作或 `Thread.sleep()` 的场景。这些阻塞操作会让线程池的线程被长时间占用，无法发挥并行优势，反而可能降低性能。

## 8. 常见陷阱与最佳实践

1. **流只能消费一次**：一个 Stream 在执行了终止操作后就会关闭，不能被再次使用。否则会抛出 `IllegalStateException: stream has already been operated upon or closed`。
2. **`parallelStream` 与 `ThreadLocal`**：`parallelStream` 使用的 `ForkJoinPool` 中的线程无法访问到主线程设置的 `ThreadLocal` 数据，因为 `ThreadLocal` 不具备跨线程继承性。
3. **`toMap` 的 Key 重复问题**：在使用 `Collectors.toMap()` 时，如果流中存在重复的 key，会抛出 `IllegalStateException`。必须提供第三个参数（一个合并函数）来解决冲突，例如 `(oldValue, newValue) -> newValue`。
4. **谨慎使用有状态操作**：如 `sorted` 和 `distinct`，它们需要缓冲所有元素，对内存消耗较大。应尽量将 `filter` 等能减少元素数量的操作放在它们前面。

## 9. 总结

* **本质：** Stream 的本质是一个由 `AbstractPipeline` 节点构成的**双向链表**，用于构建处理流程的蓝图。
* **核心机制：** 通过**终结操作**触发，**反向**实例化一个链式包裹的 `Sink` 对象链。
* **执行模式：** 数据**逐个垂直**地流过整个 `Sink` 链，实现了高效的**惰性执行**，避免了中间集合的开销。对于有状态操作，会使用内部缓冲区暂存数据，可能带来较高的内存开销。