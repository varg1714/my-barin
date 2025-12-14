---
source:
  - "[[深入理解 Java 并发编程之 Fork_Join 框架]]"
create: 2025-11-29
---

## 1. Java Fork/Join 框架

**核心思想**：**分治 (Divide and Conquer)**。
**适用场景**：将一个复杂的计算密集型大任务，拆分成多个相似的小任务并行执行，最后合并结果（类似单机版的 MapReduce）。

### 1.1. 核心组件

Fork/Join 框架主要由两部分组成：

#### 1.1.1. ForkJoinPool (线程池)

*  **角色**：执行任务的引擎。
*  **特点**：
    *  每个工作线程都有自己的**双端队列 (Deque)**。
    *  实现了**工作窃取 (Work Stealing)** 算法。

#### 1.1.2. ForkJoinTask (任务)

*  **角色**：具体的任务逻辑载体。
*  **核心方法**：
    *  `fork()`: 异步执行子任务（放入队列）。
    *  `join()`: 阻塞当前线程等待子任务结果。
*  **常用子类**：
    *  **`RecursiveTask<V>`**：**有**返回值（最常用，如计算斐波那契、累加）。
    *  **`RecursiveAction`**：**无**返回值（如并行排序、数据清洗）。

### 1.2. 核心机制：工作窃取 (Work Stealing)

这是 ForkJoinPool 性能高效的关键：

1. **双端队列**：每个工作线程维护一个双端队列。
2. **LIFO 处理**：工作线程从自己队列的**头部**（Head）获取任务执行（LIFO，后进先出），这有利于利用 CPU 缓存（热点数据）。
3. **窃取机制**：当某个线程 T1 的队列为空（无事可做）时，它会从其他忙碌线程 T2 的队列**尾部**（Tail）窃取任务来执行。
4. **优势**：
    *  充分利用 CPU 多核资源，避免线程闲置。
    *  窃取者和被窃取者从队列不同端操作，减少了锁竞争。

### 1.3. 编程范式 (代码模板)

使用 Fork/Join 通常遵循以下伪代码逻辑：

```java
class MyTask extends RecursiveTask<Integer> {
    // 阈值，决定何时不再拆分直接计算
    static final int THRESHOLD = ...; 

    @Override
    protected Integer compute() {
        // 1. 判断任务是否足够小
        if (任务规模 <= THRESHOLD) {
            return 直接计算逻辑();
        } 
        // 2. 任务太大，进行拆分 (Fork)
        else {
            int mid = (start + end) / 2;
            MyTask leftTask = new MyTask(start, mid);
            MyTask rightTask = new MyTask(mid + 1, end);
            
            // 异步执行左任务
            leftTask.fork(); 
            // 同步执行右任务 (优化点：当前线程复用)
            // 或者两个都 fork，然后两个都 join
            
            // 3. 等待结果并合并 (Join)
            return rightTask.compute() + leftTask.join();
        }
    }
}
```

### 1.4. ForkJoinPool vs ThreadPoolExecutor

| 特性 | ForkJoinPool | ThreadPoolExecutor |
| :--- | :--- | :--- |
| **任务模型** | **分治模型** (大任务拆小任务) | **简单并行** (任务间通常独立) |
| **内部结构** | 每个线程有一个队列 (双端) | 所有线程共享一个队列 (阻塞队列) |
| **调度策略** | **工作窃取** (Work Stealing) | **工作复用** (Work Reusing) |
| **队列顺序** | LIFO (本地消费), FIFO (被窃取) | FIFO (通常情况) |
| **适用性** | CPU 密集型、递归任务 | IO 密集型、Web 请求处理 |

### 1.5. 避坑指南与最佳实践

1. **Java 8 Parallel Stream**：
    *  `list.parallelStream()` 默认使用全局共享的 `ForkJoinPool.commonPool()`。
    *  **风险**：如果在这个公共池中执行了耗时的 I/O 操作（如数据库查询、网络请求），会阻塞所有使用并行流的业务，导致系统整体性能雪崩。
    *  **建议**：I/O 密集型任务严禁使用默认并行流，应自定义独立的线程池。

2. **异常处理**：
    *  ForkJoinTask 在执行时抛出的异常无法在主线程直接捕获，需要通过 `task.getException()` 或 `isCompletedAbnormally()` 来检查。

3. **拆分粒度**：
    *  拆分得太细会增加线程调度和上下文切换的开销；拆分得太粗则无法充分利用多核。需要根据具体场景测试合适的阈值。
**一句话总结**：Fork/Join 是 Java 处理**递归分治算法**的神器，利用**工作窃取**榨干 CPU 性能，但要注意避免在共享池中进行 I/O 阻塞操作。