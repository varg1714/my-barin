---
source:
  - "[[常见限流算法 与 Guava RateLimiter 实现_guava 滑动窗口 - CSDN 博客]]"
create: 2025-10-19
---

## 1. 背景：常见的限流算法

限流是保护系统免于被高并发流量冲垮的重要手段。在了解 Guava 的实现前，先回顾几种主流的限流算法。

| 算法名称 | 优点 | 缺点 |
| :--- | :--- | :--- |
| **固定窗口计数** | 实现简单，容易理解。 | 在窗口临界点可能导致两倍流量通过，无法平滑处理流量。 |
| **滑动窗口计数** | 通过更细粒度的窗口划分，解决了固定窗口的临界问题，限流更平滑。 | 依然存在微小的临界问题，且实现复杂度更高。 |
| **漏桶算法** | 强制将请求速率平滑化，以固定速率处理请求，能有效保护下游系统。 | 无法应对突发流量。即使系统有处理能力，也只能按固定速率处理，对用户不友好。 |
| **令牌桶算法** | 结合了漏桶和平滑性的优点。允许存储令牌，因此能有效应对突发流量，同时通过控制令牌生成速率来限制长期平均速率。 | 实现相对复杂。 |

**Guava RateLimiter 选择并扩展了令牌桶算法**，因为它在应对突发流量和控制平均速率之间取得了最佳平衡。

![常见限流算法图示](https://img-blog.csdnimg.cn/602b6ab10c3243158845e257e3d3318a.png)

## 2. Guava RateLimiter 的核心设计思想：延迟计算

传统的令牌桶实现通常需要一个后台线程定时向桶里放令牌，这会带来巨大的资源开销。Guava 的设计精髓在于**避免使用任何额外线程，通过“延迟计算”或“按需计算”来实现令牌的生成**。

**核心逻辑如下：**

1. **不存储真实令牌**：系统中没有一个队列或后台线程在“生产”令牌。
2. **记录未来时间点**：`RateLimiter` 维护一个核心变量 `nextFreeTicketMicros`，表示下一次可以免费获取令牌的时间点。
3. **按需计算令牌**：当一个请求到来时，`RateLimiter` 会：
    * 获取当前时间 `nowMicros`。
    * 比较 `nowMicros` 和 `nextFreeTicketMicros`。如果 `nowMicros` > `nextFreeTicketMicros`，说明在过去的一段时间里系统是空闲的。
    * 根据时间差 `(nowMicros - nextFreeTicketMicros)` 和令牌生成间隔，**瞬间计算出**这段空闲期内“本应该”生成多少新令牌。
    * 将这些新令牌补充到 `storedPermits`（当前存储的令牌数）中。
4. **更新未来时间点**：根据本次请求消耗的令牌，计算出下一次可以发放令牌的时间，并更新 `nextFreeTicketMicros`。

这种**“用计算换资源”**的设计，将令牌的生成逻辑分散到每一次请求中，极大地提升了性能和效率。

## 3. 两大核心实现：SmoothBursty 与 SmoothWarmingUp

Guava 提供了两种令牌桶的具体实现，以应对不同场景。

### 3.1. `SmoothBursty` (平滑突发)

这是标准的令牌桶实现，其设计目标是允许并平滑地处理突发流量。

* **工作机制**：系统以恒定速率产生令牌。如果一段时间没有请求，令牌会被存储起来（默认最多存储 1 秒钟产生的量）。当突发流量到来时，它可以立即消费掉所有存储的令牌而**无需等待**。
* **核心特点**：从 `storedPermits` 中获取令牌的成本为 0，即 `storedPermitsToWaitTime()` 方法直接返回 `0L`。只有当存储的令牌不够用，需要“预支”未来的令牌时，请求才需要等待。

### 3.2. `SmoothWarmingUp` (平滑预热)

这是 Guava 最具创造性的设计，旨在解决服务的**冷启动问题**。

* **设计目标**：防止刚启动、处理能力较弱的系统被突发流量压垮。它让请求速率从一个较低的水平，平滑地过渡到正常的最高水平。
* **核心机制**：引入了“成本”概念。**系统越冷（存储的令牌越多），获取令牌的成本（等待时间）就越高。**
* **梯形面积原理**：
    * 它通过一个函数图来定义获取令牌的成本，其中 **Y 轴** 是获取 1 个令牌的时间，**X 轴** 是存储的令牌数。
    * **核心思想：面积 = 时间**。计算获取 N 个令牌的总等待时间，等同于计算函数图下对应 N 个令牌宽度所覆盖的面积。
    * **图形解读**：
        ```
        ^ throttling
                      |
                cold  +                  /
             interval |                 /.
                      |                / .
                      |               /  .   ← "warmup period" is the area of the trapezoid between
                      |              /   .     thresholdPermits and maxPermits
                      |             /    .
                      |            /     .
                      |           /      .
               stable +----------/  WARM .
             interval |          .   UP  .
                      |          . PERIOD.
                      |          .       .
                    0 +----------+-------+--------------→ storedPermits
                      0 thresholdPermits maxPermits
        ```
        * **右侧梯形区域 (`thresholdPermits` 到 `maxPermits`)**：代表“预热区”。系统最冷时（`storedPermits` 接近 `maxPermits`），获取令牌的成本最高（默认为正常速率的 3 倍）。随着令牌被消耗，成本沿斜线平滑降低。
        * **左侧矩形区域 (0 到 `thresholdPermits`)**：代表“稳定区”。当系统充分预热后（`storedPermits` 降低到阈值以下），获取令牌的成本恒定为正常速率。
    * **效果**：通过这个设计，`RateLimiter` 自动在系统启动初期“踩刹车”，并随着系统的“升温”而逐渐“松开刹车”，实现了全自动的平滑预热。

## 4. 关键工程设计与实现细节

除了核心算法，Guava `RateLimiter` 的健壮性还体现在以下工程细节上：

1. **线程安全 (`synchronized`)**
    所有对内部状态（如 `storedPermits`, `nextFreeTicketMicros`）的读写操作都被 `synchronized` 块保护，确保了其在多线程环境下的安全可靠。

2. **时间源抽象 (`SleepingStopwatch`)**
    Guava 没有硬编码 `System.nanoTime()` 和 `Thread.sleep()`，而是将其抽象为 `SleepingStopwatch` 接口。
    * **目的**：实现**关注点分离**和**可测试性**。
    * **优势**：在单元测试中，可以传入一个可被完全控制的“伪造时钟”，从而在不实际等待的情况下，精确地测试各种时间相关的逻辑。

3. **动态速率调整 (`setRate`)**
    `RateLimiter` 允许在运行时动态调整限流速率。调整时，它并非粗暴地重置状态，而是采用**按比例缩放**的策略。
    * **逻辑**：保持令牌桶的“充满度”百分比不变。例如，若速率从 10 调整到 20，一个半满的桶（5/10）会变成一个新的、容量更大但同样半满的桶（10/20）。
    * **公式**：`new_storedPermits = storedPermits * new_maxPermits / old_maxPermits`
    * **效果**：保证了速率调整的平滑过渡，避免了行为突变。

4. **重要限制：单机限流 (Single-JVM Only)**
    **这是使用 Guava RateLimiter 最重要的前提**。它的所有状态都保存在单个 JVM 实例的内存中，**无法用于分布式系统**。在分布式环境中，需要借助 Redis+Lua 等外部中间件来实现全局限流。

## 5. 核心源码片段解读

### 5.1. `resync()` - “延迟计算”的体现

```java
void resync(long nowMicros) {
    // 如果当前时间晚于下次发令牌的时间，说明有空闲
    if (nowMicros > nextFreeTicketMicros) {
      // 计算空闲期内本应产生的新令牌
      double newPermits = (nowMicros - nextFreeTicketMicros) / coolDownIntervalMicros();
      // 补充令牌，但不超过最大值
      storedPermits = min(maxPermits, storedPermits + newPermits);
      // 更新时间基点
      nextFreeTicketMicros = nowMicros;
    }
}
```

### 5.2. `reserveEarliestAvailable()` - 获取令牌的核心流程

```java
final long reserveEarliestAvailable(int requiredPermits, long nowMicros) {
    resync(nowMicros); // 步骤1: 补充空闲期令牌
    long returnValue = nextFreeTicketMicros;
    
    // 步骤2: 计算本次请求分别从存储桶和未来获取多少令牌
    double storedPermitsToSpend = min(requiredPermits, this.storedPermits);
    double freshPermits = requiredPermits - storedPermitsToSpend;

    // 步骤3: 计算总等待时间 = 获取存储令牌的成本 + 获取未来令牌的成本
    long waitMicros =
        storedPermitsToWaitTime(this.storedPermits, storedPermitsToSpend) // 两种策略的核心区别点
            + (long) (freshPermits * stableIntervalMicros);

    // 步骤4: 更新下次发令牌的时间点，并消耗存储的令牌
    this.nextFreeTicketMicros = LongMath.saturatedAdd(nextFreeTicketMicros, waitMicros);
    this.storedPermits -= storedPermitsToSpend;
    return returnValue;
}
```

### 5.3. `storedPermitsToWaitTime()` - 两种策略的本质区别

* **`SmoothBursty` 的实现：**

    ```java
    long storedPermitsToWaitTime(double storedPermits, double permitsToTake) {
      return 0L; // 获取存储令牌的成本为0
    }
    ```

* **`SmoothWarmingUp` 的实现：**

    ```java
    long storedPermitsToWaitTime(double storedPermits, double permitsToTake) {
      // ... 此处是复杂的梯形和矩形面积计算逻辑 ...
      // 核心是计算获取 permitsToTake 个令牌在函数图下覆盖的面积
      // ...
      return micros; // 返回计算出的时间成本
    }
    ```
