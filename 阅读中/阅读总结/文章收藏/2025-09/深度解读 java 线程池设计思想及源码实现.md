---
source: "[[阅读中/文章列表/文章收藏/2025-09/深度解读 java 线程池设计思想及源码实现]]"
create: 2025-09-29
---

## 1. Java 线程池核心原理与源码深度解析笔记

### 1.1. 核心思想与设计总览

Java 线程池的核心设计思想是**将任务的提交与任务的执行解耦**。它通过维护一组工作线程来响应任务请求，从而避免了为每个任务都创建和销毁线程所带来的巨大开销，实现了线程的复用，并提供了对并发线程数量的有效控制。

#### 1.1.1. 类继承结构

![Java 线程池类继承结构](https://assets.javadoop.com/imgs/20510079/java-thread-pool/1.jpg)

*   **`Executor`**: 顶层接口，仅定义 `void execute(Runnable command)`，是任务执行的入口。
*   **`ExecutorService`**: 继承 `Executor`，提供了更完整的生命周期管理和任务提交功能（如 `submit`, `shutdown`, `invokeAll`）。我们通常面向此接口编程。
*   **`AbstractExecutorService`**: 实现了 `ExecutorService` 的部分通用方法，如 `submit` 方法的逻辑（将 `Runnable` / `Callable` 包装为 `FutureTask`）。
*   **`ThreadPoolExecutor`**: **线程池的核心实现类**，是本文档分析的重点。
*   **`ScheduledThreadPoolExecutor`**: 继承 `ThreadPoolExecutor`，增加了定时执行和周期性执行任务的功能。
*   **`Executors`**: 工具类，提供创建各种预设配置线程池的静态工厂方法。
*   **`FutureTask`**: 实现了 `Runnable` 和 `Future` 接口，是连接 `Callable` 和线程池的桥梁，用于获取异步任务的执行结果。

### 1.2. `ThreadPoolExecutor` 深度剖析

这是 Java 线程池最核心的实现，其行为由构造函数的几个关键参数决定。

#### 1.2.1. 核心构造参数

```java
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler)
```

1.  **`corePoolSize` (核心线程数)**: 线程池中需要维持的最小线程数。即使空闲，默认也不会被回收。
2.  **`maximumPoolSize` (最大线程数)**: 线程池允许创建的最大线程总数。
3.  **`keepAliveTime` (空闲线程存活时间)**: 当线程数超过 `corePoolSize` 时，多余的空闲线程在被销毁前可以存活的时间。
4.  **`workQueue` (任务队列)**: `BlockingQueue`，用于暂存待执行的任务。
    *   `ArrayBlockingQueue`: 有界队列，基于数组。
    *   `LinkedBlockingQueue`: 可选有界/无界队列，基于链表。
    *   `SynchronousQueue`: 不存储元素的队列，任务直接移交给消费者线程。
5.  **`threadFactory` (线程工厂)**: 用于创建新线程，可自定义线程名、优先级等。
6.  **`handler` (拒绝策略)**: 当队列已满且线程数达到 `maximumPoolSize` 时，用于处理新任务的策略。

#### 1.2.2. 内部状态管理 (`ctl`)

`ThreadPoolExecutor` 使用一个 `AtomicInteger ctl` 来同时存储两个信息，这是其高性能、无锁设计的关键。

*   **高 3 位**: 存储线程池的运行状态 (`runState`)。
*   **低 29 位**: 存储当前的工作线程数量 (`workerCount`)。

```java
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
private static final int COUNT_BITS = Integer.SIZE - 3; // 29
private static final int CAPACITY   = (1 << COUNT_BITS) - 1; // 2^29 - 1

// 状态值
private static final int RUNNING    = -1 << COUNT_BITS; // 111... (负数)
private static final int SHUTDOWN   =  0 << COUNT_BITS; // 000...
private static final int STOP       =  1 << COUNT_BITS; // 001...
private static final int TIDYING    =  2 << COUNT_BITS; // 010...
private static final int TERMINATED =  3 << COUNT_BITS; // 011...
```

> [!NOTE] 为什么这么设计？
> **原子性与高效性**: 将状态和数量打包进一个原子变量，允许通过一次 CAS 操作同时检查和修改它们，避免了使用锁，在高并发下性能极高。`RUNNING` 状态为负数的设计，使得 `c < SHUTDOWN` 就能快速判断是否在运行。

#### 1.2.3. 任务提交流程 (`execute` 方法)

这是线程池的核心调度逻辑，严格遵循以下优先级顺序：

> [!IMPORTANT] 任务处理优先级
> **核心线程 (`corePoolSize`) > 任务队列 (`workQueue`) > 最大线程 (`maximumPoolSize`) > 拒绝策略 (`handler`)**

当一个新任务通过 `execute()` 提交时：

1.  **第一步：检查核心线程**
    *   如果当前工作线程数 `workerCount` 小于 `corePoolSize`，则调用 `addWorker()` 创建一个新的**核心线程**来执行该任务。

2.  **第二步：尝试入队**
    *   如果核心线程已满，则尝试将任务添加到 `workQueue` 中。
    *   **双重检查 (Double-Check)**: 任务入队成功后，必须重新检查线程池状态。因为在入队期间，线程池可能被其他线程关闭 (`shutdown`)。如果发现已关闭，则需将任务从队列中移除，并执行拒绝策略。
    *   **预防措施**: 如果入队成功，但此时池中已无任何线程（可能因异常全部终止），则必须启动一个新线程来确保队列中的任务能被处理。

3.  **第三步：尝试创建非核心线程**
    *   如果任务队列已满，无法入队，则再次调用 `addWorker()` 尝试创建**非核心线程**来执行任务。
    *   前提是 `workerCount` 不能超过 `maximumPoolSize`。

4.  **第四步：执行拒绝策略**
    *   如果线程数已达 `maximumPoolSize` 且队列也已满，则线程池彻底饱和，此时执行 `RejectedExecutionHandler` 来处理该任务。

#### 1.2.4. 工作单元：`Worker` 内部类

`Worker` 是执行任务的实际单元，它巧妙地将线程和任务循环绑定在一起。

*   **结构**: `private final class Worker extends AbstractQueuedSynchronizer implements Runnable`
*   **核心属性**:
    *   `final Thread thread`: `Worker` 内部封装的真正执行任务的线程。
    *   `Runnable firstTask`: 创建 `Worker` 时分配给它的第一个任务。
*   **设计精髓**:
    1.  **实现 `Runnable`**: `Worker` 自身是一个 `Runnable`。在构造时，它会通过 `ThreadFactory` 创建一个新线程，并将 `this`（即 `Worker` 自身）作为 `Runnable` 目标传入。因此，`thread.start()` 最终会执行 `Worker` 的 `run()` 方法。
    2.  **继承 `AQS`**: `Worker` 实现了一个简单的独占锁。`runWorker()` 开始时会上锁，结束时解锁。这用于区分**空闲线程**和**工作线程**。`shutdown()` 只会中断空闲线程（能成功 `tryLock()` 的），而不会影响正在执行任务的线程。
    3.  **任务循环**: `Worker` 的 `run()` 方法会调用 `runWorker()`，该方法内有一个 `while` 循环，不断通过 `getTask()` 从任务队列中获取任务并执行，直到 `getTask()` 返回 `null`。

#### 1.2.5. 线程的生命周期：`getTask()` 方法

`getTask()` 是 `Worker` 任务循环的核心，它决定了线程是继续等待任务还是被回收。

*   **线程回收的关键逻辑**:
    1.  **判断是否超时等待 (`timed`)**:
        *   `boolean timed = allowCoreThreadTimeOut || wc > corePoolSize;`
        *   如果允许核心线程超时 (`allowCoreThreadTimeOut=true`) 或者当前线程数超过了核心数，则 `timed` 为 `true`。
    2.  **获取任务**:
        *   如果 `timed` 为 `true`，则调用 `workQueue.poll(keepAliveTime, ...)` 进行**限时等待**。
        *   如果 `timed` 为 `false`（即核心线程且不允许超时），则调用 `workQueue.take()` 进行**无限期阻塞等待**。
    3.  **线程终止**:
        *   当 `poll()` 因超时返回 `null` 时，`getTask()` 在下一次循环中会判断到超时，并尝试通过 CAS 减少 `workerCount`，然后返回 `null`。
        *   `runWorker()` 的 `while` 循环接收到 `null` 后会退出，线程执行完毕，自然死亡。

#### 1.2.6. 拒绝策略 (`RejectedExecutionHandler`)

当线程池饱和时触发，JDK 提供了四种默认实现：

*   `AbortPolicy` (默认): 抛出 `RejectedExecutionException` 异常。
*   `CallerRunsPolicy`: 由提交任务的线程自己来执行该任务。
*   `DiscardPolicy`: 直接丢弃任务，不做任何事。
*   `DiscardOldestPolicy`: 丢弃任务队列中等待时间最长的任务，然后重新尝试提交当前任务。

### 1.3. `Executors` 工具类的风险

`Executors` 提供了一些便捷的工厂方法，但在生产环境中需要谨慎使用。

*   **`Executors.newFixedThreadPool(n)`**:
    *   **配置**: `corePoolSize = n`, `maximumPoolSize = n`, `keepAliveTime = 0L`, `workQueue = new LinkedBlockingQueue<>()`。
    *   > [!WARNING] 风险
    >   使用**无界队列** `LinkedBlockingQueue`。如果任务生产速度远大于消费速度，会导致任务在队列中无限堆积，最终可能耗尽内存，导致 **OOM (OutOfMemoryError)**。

*   **`Executors.newCachedThreadPool()`**:
    *   **配置**: `corePoolSize = 0`, `maximumPoolSize = Integer.MAX_VALUE`, `keepAliveTime = 60L`, `workQueue = new SynchronousQueue<>()`。
    *   > [!WARNING] 风险
    >   `maximumPoolSize` 设置为 `Integer.MAX_VALUE`，理论上可以创建无限多的线程。如果瞬间任务量过大，可能会创建大量线程，耗尽系统资源，导致服务宕机。

### 1.4. 复习要点 (Q&A)

1.  **线程池创建时机是怎样的？**
    *   **1. 创建核心线程**: 提交任务时，若 `workerCount < corePoolSize`，创建新线程执行。
    *   **2. 任务入队**: 若核心线程已满，将任务放入 `workQueue`。
    *   **3. 创建非核心线程**: 若队列已满，创建新线程执行，但 `workerCount` 不能超过 `maximumPoolSize`。
    *   **4. 拒绝**: 若以上都失败，执行拒绝策略。

2.  **任务执行过程中发生异常会怎样？**
    *   执行该任务的线程会终止。`ThreadPoolExecutor` 的 `runWorker` 方法的 `finally` 块会捕获这个情况，并调用 `processWorkerExit()`。在这个过程中，会移除死掉的 `Worker` 并可能会启动一个新的线程来代替它，以维持线程池的稳定。

3.  **`newFixedThreadPool` 和 `newCachedThreadPool` 的核心区别和风险是什么？**
    *   **`Fixed`**: 固定大小，使用无界队列。风险是任务堆积导致 OOM。
    *   **`Cached`**: 可变大小（理论上无限），使用同步队列。风险是线程无限创建导致系统资源耗尽。

4.  **`keepAliveTime` 对核心线程有效吗？**
    *   默认情况下无效。核心线程即使空闲也不会被回收。
    *   但是，可以通过调用 `allowCoreThreadTimeOut(true)` 方法来使 `keepAliveTime` 对核心线程也生效。

5.  **`shutdown()` 和 `shutdownNow()` 有什么区别？**
    *   **`shutdown()`**: 将线程池状态设为 `SHUTDOWN`。不再接受新任务，但会继续处理队列中的存量任务。它会中断**空闲**的线程。
    *   **`shutdownNow()`**: 将线程池状态设为 `STOP`。不再接受新任务，清空任务队列，并尝试中断**所有**正在执行任务的线程。