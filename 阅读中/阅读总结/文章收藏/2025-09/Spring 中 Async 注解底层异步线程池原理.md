---
source: "[[浅析 Spring 中 Async 注解底层异步线程池原理｜得物技术]]"
create: 2025-09-19
---

## 1. Spring @Async 与 CompletableFuture 默认线程池深度解析

### 1.1. 核心差异总览

`@Async` 注解和 `CompletableFuture` 都用于实现异步编程，但它们默认使用的线程池在类型、来源、配置和核心机制上存在本质区别。

| 特性        | @Async (默认)                                               | CompletableFuture (默认)                     |
| --------- | --------------------------------------------------------- | ------------------------------------------ |
| **线程池类型** | `ThreadPoolTaskExecutor` (常见) 或 `SimpleAsyncTaskExecutor` | `ForkJoinPool`                             |
| **来源**    | 由 Spring 框架根据上下文自动配置或创建                                   | JDK 内置的公共线程池 (`ForkJoinPool.commonPool()`) |
| **默认配置**  | 核心线程数=8，最大线程数和队列容量为 `Integer.MAX_VALUE` (存在 OOM 风险)       | 线程数通常与 CPU 核心数相关，为计算密集型任务优化                |
| **核心机制**  | 传统的生产者-消费者模型，共享任务队列                                       | 工作窃取 (Work-Stealing) 算法，每个线程有自己的双端队列       |

### 1.2. 一、Spring `@Async` 详解

此部分内容主要基于文章 [[浅析 Spring 中 Async 注解底层异步线程池原理｜得物技术]]。

#### 1.2.1. 核心原理：AOP

Spring 通过 **AOP (面向切面编程)** 来实现 `@Async` 的功能。

*   **切面类**: `AsyncAnnotationAdvisor` 会扫描被 `@Async` 注解标识的方法。
*   **拦截器**: 当这些方法被调用时，`AsyncExecutionInterceptor` 会拦截调用请求。
*   **异步执行**: 拦截器不会立即执行方法，而是将方法调用封装成一个任务，并提交到一个线程池中去执行，从而实现了主调线程的非阻塞。

#### 1.2.2. 默认线程池获取流程

当使用 `@Async` 且未指定线程池名称时（如 `@Async("myThreadPool")`），Spring 会按照以下顺序寻找一个默认线程池：

1.  **查找唯一的 `TaskExecutor` Bean**：Spring 会尝试从容器中寻找一个 `TaskExecutor` 类型的 Bean。如果容器中**有且仅有一个**这样的 Bean（可能是你自己配置的，也可能是第三方库引入的），`@Async` 就会使用它。
2.  **Spring Boot 自动装配**：如果找不到唯一的 `TaskExecutor`，Spring Boot 的 `TaskExecutionAutoConfiguration` 会自动创建一个 `ThreadPoolTaskExecutor` 实例。
    *   **默认参数**：
        *   核心线程数 (`corePoolSize`): **8**
        *   最大线程数 (`maxPoolSize`): **`Integer.MAX_VALUE`**
        *   队列容量 (`queueCapacity`): **`Integer.MAX_VALUE`**
3.  **最终回退机制**：如果以上两种方式都失败，Spring 会创建一个 `SimpleAsyncTaskExecutor`。

#### 1.2.3. 潜在风险与最佳实践

`@Async` 的默认行为可能导致以下问题：

*   **风险 1：误用未知线程池**：如果项目中存在一个未知的 `TaskExecutor` Bean，可能会导致 `@Async` 任务在不合适的线程池上运行，引发难以排查的问题。
*   **风险 2：内存溢出 (OOM)**：Spring Boot 自动装配的 `ThreadPoolTaskExecutor` 使用了无界队列 (`Integer.MAX_VALUE`)。如果短时间内有大量异步任务被提交，会导致任务在队列中大量堆积，最终耗尽内存。
*   **风险 3：性能下降**：`SimpleAsyncTaskExecutor` **不是一个真正的线程池**。它每次执行任务都会创建一个新线程，且不会复用。在高并发场景下，频繁创建和销毁线程会带来巨大的性能开销。

**最佳实践**：**永远不要依赖 `@Async` 的默认线程池**。应当始终在项目中显式地配置一个或多个自定义的 `ThreadPoolTaskExecutor` Bean，并根据业务场景合理设置其核心线程数、最大线程数、队列容量和拒绝策略。然后通过 `@Async("myThreadPoolName")` 的方式明确指定使用哪个线程池。

### 1.3. 二、`CompletableFuture` 与 `ForkJoinPool` 详解

#### 1.3.1. 默认线程池：`ForkJoinPool.commonPool()`

`CompletableFuture` 的异步方法（如 `supplyAsync`, `runAsync`）在不指定 `Executor` 的情况下，默认使用 JDK 全局共享的 `ForkJoinPool.commonPool()`。

#### 1.3.2. 核心机制：工作窃取 (Work-Stealing) 算法

`ForkJoinPool` 的高效源于其**工作窃取**机制，这与传统线程池（如 `ThreadPoolExecutor`）有根本不同。

*   **传统模型**：所有线程从一个**共享的公共任务队列**中获取任务，在高并发下会产生锁竞争，且容易因长任务阻塞导致负载不均。
*   **工作窃取模型**：
    1.  **独立双端队列 (Deque)**：每个工作线程都有一个自己的私有任务队列。
    2.  **LIFO/FIFO 策略**：
        *   **工作时 (LIFO)**：线程从**自己队列的头部**获取任务执行。这能提高 CPU 缓存命中率，因为刚被拆分的子任务最可能在缓存中。
        *   **空闲时 (FIFO)**：当线程完成自己队列的所有任务后，它会变成“窃取者”，随机扫描其他线程的队列，并从**队列的尾部**“偷”一个任务来执行。尾部的任务通常是更早被放入的、粒度更大的父任务。
    3.  **优势**：
        *   **减少竞争**：绝大部分时间线程都在操作自己的队列，锁竞争极小。
        *   **充分利用 CPU**：确保了只要还有任务存在，就不会有线程闲置，实现了高效的负载均衡。

#### 1.3.3. 为何适合分治 (Divide-and-Conquer) 任务

工作窃取算法与分治思想是天作之合。当一个大任务可以被递归地分解成多个独立的子任务时（如并行排序、大规模数据处理），`ForkJoinPool` 能发挥最大威力。一个线程可以将任务分解（Fork），并将子任务放入自己的队列，其他空闲线程可以立即窃取这些子任务并行处理，最后再将结果合并（Join）。

### 1.4. 三、理论与实践：如何实现任务拆分

#### 1.4.1. `supplyAsync` 中的“黑箱”任务

直接传递给 `CompletableFuture.supplyAsync` 的 Lambda 表达式，对于 `ForkJoinPool` 来说是一个**不可拆分的原子任务**。线程池只会完整地执行它，而不会尝试进入其内部进行拆分。在这种场景下，工作窃取的作用是在**多个独立的 `supplyAsync` 任务之间**实现负载均衡。

#### 1.4.2. 显式实现可拆分任务：`RecursiveTask`

要真正利用 `ForkJoinPool` 的分治能力，必须显式地编写拆分逻辑，通常通过继承 `RecursiveTask<V>` (有返回值) 或 `RecursiveAction` (无返回值) 来实现。

**核心步骤**：
1.  **继承 `RecursiveTask<V>`** 并重写 `compute()` 方法。
2.  在 `compute()` 方法中定义**任务拆分的阈值 (Threshold)**。当任务规模小于该阈值时，直接计算，这是递归的出口。
3.  当任务过大时，将其**拆分 (Fork)** 成更小的子任务。通常调用 `subTask.fork()` 将子任务异步提交到队列。
4.  等待子任务完成并**合并 (Join)** 结果。调用 `subTask.join()` 会阻塞等待其结果。

### 1.5. 四、强强联合：结合 `CompletableFuture` 与 `ForkJoinTask`

我们可以将 `ForkJoinTask` 的强大计算能力与 `CompletableFuture` 的灵活编排能力结合起来。

#### 1.5.1. 角色分工

*   **`RecursiveTask`**：定义**“怎么算”**，即任务分治的具体逻辑。
*   **`ForkJoinPool`**：提供**“在哪里算”**的执行环境。
*   **`CompletableFuture`**：负责**“算完之后干什么”**，编排异步工作流。

#### 1.5.2. 实现模式

将耗时的、可分治的 `ForkJoinTask` 计算过程，封装在一个 `CompletableFuture.supplyAsync` 调用中。

```java
// 创建一个可拆分的任务
SumTask myRecursiveTask = new SumTask(...);
ForkJoinPool pool = ForkJoinPool.commonPool();

// 将阻塞的invoke调用封装成一个非阻塞的Future
CompletableFuture<Long> future = CompletableFuture.supplyAsync(() -> {
    return pool.invoke(myRecursiveTask); 
});

// 之后便可以利用CompletableFuture进行链式调用
future.thenApply(...)
      .thenAccept(...);
```

#### 1.5.3. 核心优势

1.  **非阻塞封装**：将 `forkJoinPool.invoke()` 这个阻塞调用隔离在异步线程中，让主调用线程保持非阻塞。
2.  **能力组合**：可以轻松地将 CPU 密集型的分治计算结果与 IO 密集型任务（如数据库查询、RPC 调用）的结果进行组合和编排。
3.  **统一模型**：将底层复杂的计算任务统一到 `CompletableFuture` 的异步编程模型下，使代码更具可读性和可维护性。